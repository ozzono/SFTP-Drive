# How to change mounted drive partition owner

## This is a swift tutorial on how to change the mounted partition point owner user, access and permissions.

_Leia esse artigo em português [aqui](https://github.com/synini/SFTP-Drive/blob/master/Fix%20partition%20ownership_pt.md)._

This tutorial is meant for those who use a shared storage in a remote SSH-enabled Linux machine. It was tested for Ubuntu, but it will be useful on most Linux distros.

By default, when in a desktop with a GUI (graphical user interface), the mounted partition point is locally accessible by all users. However, when using a server-like machine and mounting the storage manually at least once, it will be needed to configure the permissions to other users besides _root_.

Before executing the commands I suggest in this tutorial, I tried to change the folder's owner with the command `sudo chown -R <user> <folder>`, mode with `sudo chmod a+rwx <folder>`. These had no result, so I kept searching and tried to change the folder's attribute with `sudo chattr -i <folder>`, which returned the following error:
>chattr: Inappropriate ioctl for device while reading flags on \<folder>.

That said, lets start the tutorial.

### Step 1
If mounted, unmount the storage. Type the following command to see a simplified list of disks and partitions:
>`lsblk -o name,mountpoint,label,size,fstype,uuid | egrep -v "^loop"`

It's return looks like this:
```
NAME   MOUNTPOINT                LABEL   SIZE FSTYPE   UUID
sda                                      2.7T          
├─sda1 /data/part1               part1   1.2T vfat     5633-529C
└─sda2 /data/part2               part2   1.2T vfat     9792-26A7
```
It is also possible to use the simpler command `lsblk -f` which has a similar output. 

To unmount the storage using the example, run the command `sudo unmount /data/part1` or `sudo unmount /dev/sda1`

### Step 2
Create the mountpoint folder:
>example `mkdir -p /data/part1`

_The folder /mnt/sda1 is also very common._

### Step 3
Check your userID's `uid` number _(usually it's 1000, sometimes 1001 or 1002)_:
> `grep ^"$USER" /etc/group`

And use that number if you want to grab ownership.

> - `sudo mount -o rw,user,uid=1000,umask=007,exec /dev/sda1 /data/part1` # with execute permission
> - `sudo mount -o rw,user,uid=1000,dmask=007,fmask=117 /dev/sda1 /data/part1` # without execute permission
> - `sudo mount -o rw,users,umask=000,exec /dev/sda1 /data.part1` # with permission to everyone - convenient but unsafe

_Note: it is also possible to run a similar command using the user group's `gid`._

### **Et voilà**
```
$ls -l data/
total 512
drwxrwx--- 4 home root 262144 Dec 31  1969 part1
```
It's now remotely accessible with read and write permissions, in this example by the `home` user.

Check here how to configure a [Remote SSH/SFTP Storage Drive access on Windows 10](https://medium.com/@huvirgilio/ssh-sftp-storage-drive-on-windows-10-1e16210a919a).

>#### Notes:
> - This tutorial is based on [this](https://askubuntu.com/questions/11840/how-do-i-use-chmod-on-an-ntfs-or-fat32-partition/956072#956072) answer from [Sudodus](https://askubuntu.com/users/55537/sudodus).
> - It was patiently reviewed by [Heitor Brandão](https://m.facebook.com/heitorvinicius.guedesbrandao) at 2020-08-25
