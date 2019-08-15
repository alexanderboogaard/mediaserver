---
layout: default
title: Apple Install
---
# Samba for timemachine

Install samba and edit the samba config file:

```
sudo yum install samba
sudo nano /etc/samba/smb.conf
```

Make sure your file holds the following:
```
[global]
        workgroup = SAMBA
        security = user

        passdb backend = tdbsam
        min protocol = SMB2

        vfs objects = catia fruit streams_xattr
        fruit:aapl = yes
        fruit:model = MacSamba
        fruit:resource = file
        fruit:metadata = netatalk
        fruit:locking = netatalk
        fruit:encoding = native

[Series]
        path = /media/series
        writable = yes
        browsable = yes
        guest ok = no
        valid users = username

[Movies]
        path = /media/movies
        writable = yes
        browsable = yes
        guest ok = no
        valid users = username

[Timemachine]
      	path = /media/timemachine/
        guest ok = no
        browseable = yes
        read only = no
        create mask = 0660
        directory mask = 0770
        fruit:time machine = yes
```

After this prepare an avahi file so these folders show up on your Mac.

```
sudo nano /etc/avahi/services/
```
```
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
 <name replace-wildcards="yes">%h</name>
 <service>
   <type>_smb._tcp</type>
   <port>445</port>
 </service>
 <service>
   <type>_device-info._tcp</type>
   <port>0</port>
   <txt-record>model=RackMac</txt-record>
 </service>
</service-group>
```

Add the predefined samba service opening to your firewalld configuration:
```
sudo firewall-cmd --permanent --add-service=samba
```
This is a predefined service and can be found as an XML file in the /usr/lib/firewalld/services/ directory.

As seen on the media install this could also have been a user-defined xml in the /etc/firewalld/services/ directory. Please be aware that when you have two files with the same name, the one in the /etc/ folder takes precedence.

If you have SE-linux enabled on your system you have to tell it that the /media/* drives are allowed to be accessed by defining the shares:
```
sudo chcon -R -t samba_share_t /media/series
sudo chcon -R -t samba_share_t /media/movies
sudo chcon -R -t samba_share_t /media/timemachine
```

If you also let your home directory open up on samba, you should issue the following command to let samba have access:
```
setsebool -P samba_enable_home_dirs on
```

After this you have to restart samba, avahi and reload firewalld:
```
sudo systemctl restart smb.service
sudo systemctl restart avahi-daemon.service
sudo firewall-cmd --reload
```
Your drives should now pop-up in the sidebar of your mac.

# CUPS (AirPrint)

First install CUPS.
```
sudo yum install cups
```

Edit your cupsd.conf file
```
sudo nano /etc/cups/cupsd.conf
```
```
01 ServerAlias *
02 Listen *:631
03 Listen /var/run/cups/cups.sock
04 # Restrict access to the server...
05 <Location />
06    Allow @LOCAL
07    Order allow,deny
08 </Location>
09 # Restrict access to the admin pages...
10 <Location /admin>
11   Order allow,deny
12   Allow @LOCAL
13 </Location>
14 # Restrict access to configuration files...
15 <Location /admin/conf>
16   AuthType Default
17   Require user @SYSTEM
18   Order allow,deny
19   Allow @LOCAL
20 </Location>
```

If you're on CentOS you also have to edit another file to add your group to the @SYSTEM group
```
sudo nano /etc/cups/cups-files.conf
```

Look for the line with 'SystemGroup sys root' and add 'wheel' to it.

Prepare a firewall service:
```
sudo nano /etc/firewalld/services/airprint.xml
```
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Airprint Services</short>
  <description>This service is for Airprint</description>
  <port protocol="tcp" port="631"/>
  <port protocol="udp" port="631"/>
</service>
```
```
sudo firewall-cmd –add-service=airprint –permanent
sudo firewall-cmd –reload
sudo systemctl restart cups.service
```

This tutorial is based on my home printer (Canon Pixma MG2950) which is connected with USB.

For this printer drivers have to be installed as CUPS does not recognize the printer by default, so go to https://canon.com and download the linux 64bit rpm package. This will be a tar package.

Move it to you server and issue the following commands:

```
tar -xvf cnijfilter2-5.00-1-rpm.tar
cd cnijfilter2-5.00-1-rpm
sudo ./install.sh
```

Follow the instructions. If you can't select the printer just exit. This doesn't matter.

Now open your browser and direct it to your http://servername:631

Add your printer to CUPS and print a testpage to see if it works. Make sure the printer is set to 'Shared'

If the testpage works, create an avahi setup using a script made by tjfontaine:

```
git pull https://github.com/tjfontaine/airprint-generate.git /opt/airprint
cd /opt/airprint
sudo python /opt/airprint/airprint-generate.py -d /etc/avahi/services
```
This will create a service file like the following:
```
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">AirPrint iPrinter @ %h</name>
  <service>
    <type>_ipp._tcp</type>
    <subtype>_universal._sub._ipp._tcp</subtype>
    <port>631</port>
    <txt-record>txtvers=1</txt-record>
    <txt-record>qtotal=1</txt-record>
    <txt-record>Transparent=T</txt-record>
    <txt-record>URF=none</txt-record>
    <txt-record>rp=printers/iPrinter</txt-record>
    <txt-record>note=AirPrint</txt-record>
    <txt-record>product=(GPL Ghostscript)</txt-record>
    <txt-record>printer-state=3</txt-record>
    <txt-record>printer-type=0x82900c</txt-record>
    <txt-record>Color=T</txt-record>
    <txt-record>pdl=application/octet-stream,application/pdf,application/postscript,application/vnd.cups-raster,image/gif,image/jpeg,image/png,image/tiff,image/urf,text/html,text/plain,application/vnd.adobe-reader-postscript,application/vnd.cups-command</txt-record>
  </service>
</service-group>
```

You also need to create some MIME-types in order to use AirPrint:
```
sudo echo "image/urf urf string(0,UNIRAST<00>)" > /usr/share/cups/mime/airprint.types
sudo echo "image/urf application/pdf 100 pdftoraster" > /usr/share/cups/mime/airprint.convs
sudo echo "image/urf application/vnd.cups-postscript 66 pdftops" > /usr/share/cups/mime/local.convs
sudo systemctl restart avah-daemon.service
```

You should now be able to print using airprint.
