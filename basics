Install CentOS and ssh into it

```
sudo yum update
sudo yum install nano wget
```

edit environment file to get rid of errors
```
sudo nano etc/environment
```

an empty file opens. You can paste the following:

```
LANG=en_US.utf-8
LC_ALL=en_US.utf-8
```

close the file and reboot server

I have 4 harddisks in my system:

* HDD1 = OS
* HDD2 = Movies
* HDD3 = Series
* HDD4 = Time Machine & Other Stuff

For the Movies, Series and Time Machine drives I prepare separate mount points.

```
sudo mkdir /media/{series,movies,timemachine}
```

To make sure all disks are mounted you can use blkid and take notes of the UUID of the corresponding HDD.

```
sudo blkid
```

This will give you a list of the drives with their UUID. Now edit your fstab file:

```
sudo nano /etc/fstab
```
and add the following (of course changing the UUID)
```
#Movies 
UUID=HDD2 /media/movies ext4 defaults 0 0
#Series 
UUID=HDD3 /media/series ext4 defaults 0 0
#Timemachine 
UUID=HDD4 /media/timemachine ext4 defaults 0 0
```

Close the file and issue the following command to mount the newly added disks

```
sudo mount -a
```

All drives are now mounted.

To make sure all newly created files inherit the correct Group ID: Setting the setgid permission on a directory ("chmod g\+s") causes new files and subdirectories created within it to inherit its group ID, rather than the primary group ID of the user who created the file (the owner ID is never affected, only the group ID).

So:

```
sudo chown -R username:username /media/{series,movies}
find /media/{series,movies} -type f -exec chmod 664 {} \;
find /media/{series,movies} -type d -exec chmod 770 {} \;
sudo chmod g+s /media/{series,movies}
```

That's it for new drives. For existing drives:

```
find /media/{series,movies} -type d -exec chmod g+s {} \;
```
