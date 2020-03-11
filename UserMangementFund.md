# User Management Fundamentals #

## Introduction to Users and Groups ##

### User and Group IDs ###

UID Ranges

- UID 0 is always assigned tothe superuser account root. (superuser)
- UID 1-200 is a user ID rande for system users for system processes. There are statically assigned by the system. (system users)
- UID 201-999 is a user ID range given to system users for system processes but don't own files on the system. These are dynamically assigned when the packages that require them are insalled. These user IDs have restricted access only to the resources they need for operating. (system users)
- UID 1000+ is the user ID range assigned for all regular users. (reqular users)

```
[cloud_user@dyry1c ~]$ id
uid=1004(cloud_user) gid=1005(cloud_user) groups=1005(cloud_user),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

### User shell ###

What is shell?

A shell is a program that acts as an interface between the user and the operating system (OS) kernel. A shell is started each time a user logs in and is responsible for executing programs based on user input.

The shell also provides a user environment that can be customized by configuring the profile initialization files for each user. These fiels contain user settings for:
- Paths where commands are located 
- Devining variables
- Customizable values, such as terminal prompt

Availale shells:
- The Bourne shell (/bin/sh)
- The Korn Shell (/bin/ksh)
- The C Shell (/bin/csh)
- The Bourne-again shell (/bin/bash)

User shell set to /sbin/nologin: Print the following message (/etc/nologin.txt) and exit: "This account is currently not available."

User shell set to /bin/false:

The user is immediately logged out. This file is just a binary the immediately exist, returning false.


### Workig with Home Directiories ###

User home directories

a user's home directry is the directory in which user enters upon login. It is intended to store a user's files, directories, and executables.
- Identified by and $Home
- Resides on the / partition

The /etc/skel Direcotry

/etc/skel contains files and directories that are copied over to a new user's home direcotry when it is created with the useradd command.

User Home Directory

Example home directory:
```
[cloud_user@dyry1c ~]$ ls -al
total 48
drwx------. 10 cloud_user cloud_user 4096 Mar 11 15:45 .
drwxr-xr-x.  7 root       root         75 Oct  8  2018 ..
-rw-------.  1 cloud_user cloud_user   36 Mar 11 16:02 .bash_history
-rw-r--r--.  1 cloud_user cloud_user   18 Sep  7  2018 .bash_logout
-rw-r--r--.  1 cloud_user cloud_user  193 Sep  7  2018 .bash_profile
-rw-r--r--.  1 cloud_user cloud_user  231 Sep  7  2018 .bashrc
drwx------.  7 cloud_user cloud_user 4096 Mar 11 15:45 .cache
drwxr-xr-x. 10 cloud_user cloud_user 4096 Mar  8  2019 .config
drwx------.  3 cloud_user cloud_user   24 Oct 15  2018 .dbus
drwxr-xr-x.  2 cloud_user cloud_user    6 Oct 15  2018 Desktop
-rw-------.  1 cloud_user cloud_user   16 Oct 15  2018 .esd_auth
-rw-------.  1 cloud_user cloud_user  310 Mar 11 15:44 .ICEauthority
drwx------.  3 cloud_user cloud_user   18 Oct 15  2018 .local
drwxr-xr-x.  4 cloud_user cloud_user   37 Dec 19  2015 .mozilla
drwx------.  2 cloud_user cloud_user   28 Oct  8  2018 .ssh
-rw-------.  1 cloud_user cloud_user  693 Oct  8  2018 .viminfo
drwxr-xr-x.  2 cloud_user cloud_user   82 Mar 11 15:44 .vnc
-rw-------.  1 cloud_user cloud_user 4507 Mar 11 15:44 .Xauthority
```

```
[root@dyry1c ~]# useradd usera
[root@dyry1c ~]# cd /home/
[root@dyry1c home]# ls
cloud_user  ec2-user  ssm-user  test  user  usera

[root@dyry1c home]# useradd -d /userb userb
[root@dyry1c home]# usermod -d /userb_new userb

[root@dyry1c home]# ls
cloud_user  ec2-user  ssm-user  test  user  usera

[root@dyry1c home]# mkdir /userb_new
[root@dyry1c home]# chown userb:userb /userb_new

[root@dyry1c home]# chmod 700 /userb_new

[root@dyry1c /]# cd userb
[root@dyry1c userb]# ls -la
total 16
drwx------.  2 userb userb   59 Mar 11 16:06 .
dr-xr-xr-x. 20 root  root  4096 Mar 11 16:07 ..
-rw-r--r--.  1 userb userb   18 Mar 12  2019 .bash_logout
-rw-r--r--.  1 userb userb  193 Mar 12  2019 .bash_profile
-rw-r--r--.  1 userb userb  231 Mar 12  2019 .bashrc

```
### User Management files: /etc/passwd, /etc/shadow, /etc/group ###

Useer Management Files

- /etc/passwd is a text file that contains the attributes for all user accounts on the system. This includes name, password, UID, GID, gecos, home directory, and shell.
- /etc/shadow is a text file that stores the passwords for all user accounts, in encrypted format, as well as a few configurable password properties.
- /etc/group is a text file that contains all of the groups configured on a system, and the users who belong to those groups.
- /etc/gshadow is a text file that contains all of the groups configured on a system, and the encrypted password, of in is set. It also lists group administrators and group members.

