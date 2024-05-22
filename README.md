# Info
Navidrome on Rocky linux, running on podman rootless container.
Tested on OVH basic VPS, on Rocky linux 9.4

# Instructions

Instructions are based on what i did, feel free to manage your stuff as you want!
Common stuff

```bash
sudo dnf update
sudo dnf install firewalld
sudo dnf group install "Container Management"
```

### You can skip this part if you don't know what you're doing



```bash
sudo vi /etc/ssh/sshd_config.d/00-myconf.conf
---
PasswordAuthentication no  
PubKeyAuthentication yes  
PermitRootLogin no  
Port 2222
SyslogFacility LOCAL5  
LogLevel VERBOSE
#DenyUsers navidrome -> i'll leave it commented for now
```
```bash
sudo vi /etc/ssh/rsyslog.d/00-myconf.conf
---
local5.*      /var/log/ssh.log
```

Managing network stuff
```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --add-port=4533/tcp --permanent
sudo firewall-cmd --remove-port=22/tcp --permanent
sudo firewall-cmd --reload #this will NOT lock you off
```
User Creation
```bash
sudo useradd navidrome 
sudo passwd navidrome
```
Restarting server
```bash
sudo systemctl reastart sshd rsyslog
```

#### Copy your ssh key into your user's .ssh/authorized_keys before leaving the vm or you will be locked out!

Now exit the server and login as navidrome via ssh

Creating the folders
```bash
mkdir /home/navidrome/data
mkdir /home/navidrome/music
mkdir /home/navidrome/.config/systemd/user
```
Now change directory
```bash
cd /home/navidrome/.config/systemd/user
```

Run the podman container
```bash
podman run -d \
--name navidrome \
-v /home/navidrome/data:/data:Z \
-v /home/navidrome/music:/music:Z \
-e ND_SCANSCHEDULE=1h \
-e ND_LOGLEVEL=info \
-e ND_SESSIONTIMEOUT=24h \
-e ND_BASEURL="" \
-p 4533:4533 \
deluan/navidrome:latest
```

Wait for it to be up and running checking the logs, and then you can generate the systemd unit
```bash
podman generate systemd --name navidrome --files
systemctl --user daemon-reload
systemctl --user enable container-navidrome.service
```

Exit and relogin as rocky via ssh (or your admin/sudo user). Enable lingering for user navidrome
```bash
sudo loginctl enable-linger navidrome
```
Let's avoid navidrome user to ssh login 
```bash
sudo vi /etc/ssh/sshd_config.d/00-myconf.conf
---
PasswordAuthentication no  
PubKeyAuthentication yes  
PermitRootLogin no  
Port 2222
SyslogFacility LOCAL5  
LogLevel VERBOSE
DenyUsers navidrome        #Now you can uncomment it
```

If everything is alright, after a machine reboot you should have a lingering session spawned for user navidrome, running navidrome in a rootless container, confined and secured. YAY!

# Thanks
Many thanks to morrolinux to letting me know something i never heard about and to his commitment and dedication to spread the open source word.
