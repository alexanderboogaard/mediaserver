---
title: Media Install
---
This tutorial will help you installing:
* Plex
* Sickgear
* Couchpotato
* AutoSub
* Transmission

# Plex
To enable the Plex repository open your text editor and create a new YUM repository configuration file named plex.repo in /etc/yum.repos.d/ directory:

```
sudo nano /etc/yum.repos.d/plex.repo
```

```
[PlexRepo]
name=PlexRepo
baseurl=https://downloads.plex.tv/repo/rpm/$basearch/
enabled=1
gpgkey=https://downloads.plex.tv/plex-keys/PlexSign.key
gpgcheck=1
```

Install the latest version of the Plex Media Server with:

```
sudo yum install plexmediaserver
```

Once the installation is completed start the plexmediaserver service and enable it to start on system boot with the following commands:

```
sudo systemctl start plexmediaserver.service
sudo systemctl enable plexmediaserver.service
```

Open your text editor of choice and create the following Firewalld service:

```
/etc/firewalld/services/plexmediaserver.xml
```

```
<?xml version="1.0" encoding="utf-8"?>
<service version="1.0">
<short>plexmediaserver</short>
<description>Plex TV Media Server</description>
<port port="1900" protocol="udp"/>
<port port="5353" protocol="udp"/>
<port port="32400" protocol="tcp"/>
<port port="32410" protocol="udp"/>
<port port="32412" protocol="udp"/>
<port port="32413" protocol="udp"/>
<port port="32414" protocol="udp"/>
<port port="32469" protocol="tcp"/>
</service>
```
Save the file and apply the new firewall rules by typing:

```
sudo firewall-cmd --add-service=plexmediaserver --permanent
sudo firewall-cmd --reload
```

Finally check if the new firewall rules are applied successfully with:

```
sudo firewall-cmd --list-all
```

To make sure you can delete the mediafiles from within Plex you have to add the Plex user to your own *groupname*:

```
sudo usermod -a -G groupname plex
```

restart the plexmediaserver.service and you can delete files from within plex.

For a more detailled explanation regarding Plex installation follow this link:
[How to install Plex Media Server on CentOS 7](https://linuxize.com/post/how-to-install-plex-media-server-on-centos-7/)

#Prerequisites sichkgear, couch potato and autosub

Because python 2.7 will not be supported on major distros install PYENV for sickrage and couchpotato. Also install git already:

```
sudo yum install git gcc
sudo yum install patch sqlite-devel bzip2-devel readline-devel zlib-devel openssl-devel
mkdir /opt/pyenv
sudo chown username -R /opt/pyenv
git clone https://github.com/yyuu/pyenv /opt/pyenv
```

Time to install your desired python version (at time of writing 2.7.16, but always good to check):
```
PYENV_ROOT=/opt/pyenv /opt/pyenv/bin/pyenv install 2.7.16
```

Prepare a firewall service:
```
sudo nano /etc/firewalld/services/mediastation.xml
```

```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Media Services</short>
  <description>This service is for Sickgear, Couchpotato, Transmission and Autosub</description>
  <port protocol="tcp" port="5050"/>
  <port protocol="tcp" port="8081"/>
  <port protocol="tcp" port="9091"/>
  <port protocol="tcp" port="9960"/>
  <port protocol="tcp" port="51413"/>
</service>
```

```
sudo firewall-cmd –add-service=mediastation –permanent
sudo firewall-cmd --reload
```

Time to install sickgear, couchpotato and autosub

#Sickgear

```
sudo git clone https://github.com/SickGear/SickGear.git /opt/sickgear
sudo chown username -R /opt/sickgear
```

Install sickgear dependencies
```
cd /opt/sickgear && /opt/pyenv/versions/2.7.16/bin/pip install -r requirements.txt
```

Start sickgear to test the install.
```
/opt/pyenv/versions/2.7.16/bin/python /opt/sickgear/sickgear.py
```

Go to http://localhost:8081 to check the install and stop the service
Alternatively hit Ctrl+C from terminal

To enable services after reboot:
```
sudo cp /opt/sickgear/init-scripts/init.systemd /etc/systemd/system/sickgear.service
```

Edit the user, group and the ExecStart-line:
```
sudo nano /etc/systemd/system/sickgear.service
```

```
ExecStart=/opt/pyenv/versions/2.7.16/bin/python /opt/sickgear/SickBeard.py --systemd --datadir=/opt/sickgear/data
```

Save and close file
```
sudo systemctl enable sickgear.service
sudo systemctl start sickgear.service
```

#CouchPotato

For couchpotato to sequence is almost the same.

```
sudo git clone https://github.com/CouchPotato/CouchPotatoServer.git /opt/couchpotato
sudo chown username -R /opt/couchpotato
```

Start couchpotato to test the install.
```
/opt/pyenv/versions/2.7.16/bin/python /opt/couchpotato/CouchPotato.py
```

Go to http://localhost:5050 to check the install and stop the service
Alternatively hit Ctrl+C from terminal

To enable services after reboot:
```
sudo cp /opt/couchpotato/init/couchpotato.service /etc/systemd/system/couchpotato.service
```

Edit the user, group and the ExecStart-line:
```
sudo nano /etc/systemd/system/couchpotato.service
```

```
ExecStart=/opt/pyenv/versions/2.7.16/bin/python /opt/couchpotato/CouchPotato.py
```

Save and close file

```
sudo systemctl enable couchpotato.service
sudo systemctl start couchpotato.service
```

#AutoSub

And again for AutoSub:
```
sudo git clone https://github.com/BenjV/autosub.git /opt/autosub
sudo chown username -R /opt/autosub
```

Start autosub to test the install.
```
/opt/pyenv/versions/2.7.16/bin/python /opt/autosub/AutoSub.py
```

Go to http://localhost:9960 to check the install and stop the service
Alternatively hit Ctrl+C from terminal

To enable services after reboot:
```
sudo nano /etc/systemd/system/autosub.service
```
This opens a new file. Paste the following:
```
[Unit]
Description=Autosub Daemon
Documentation=https://github.com/BenjV/autosub
After=network.target

[Service]
User=username
Group=groupname
WorkingDirectory=/opt/autosub
ExecStart=/opt/pyenv/versions/2.7.16/bin/python /opt/autosub/AutoSub.py -l --config=/opt/autosub/config.properties -d
ExecStop=/usr/bin/curl -f -s "http://xxx.xxx.xxx.xxx:9960/home/shutdown"
GuessMainPID=no
Type=forking
TimeoutStopSec=60
#KillMode=process
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```
Save and close file (edit username and groupname)

```
sudo systemctl enable autosub.service
sudo systemctl start autosub.service
```
