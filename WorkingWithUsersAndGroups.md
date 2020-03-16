# Working with Users and Groups #

## Create, Modify, or Remove Users and Groups ##

### Creating Users ###

Creating users: useradd

```
$useradd -c "<comments>" - d <home_dir> -m -g <group> -p <password> - <user_shell> username

e.g.

$useradd -c "John Doe" -d /home/appteam -m -g wheel -p abcd§1234 -s /bin/sh jdoe

[root@dyry1c ~]# grep jdoe /etc/passwd
jdoe:x:2000:10:Jane Doe:/jane:/bin/sh
[root@dyry1c ~]# grep jford /etc/passwd
jford:x:2001:10:John Ford:/john:/bin/sh


[root@dyry1c ~]# grep 10 /etc/group
wheel:x:10:ec2-user,cloud_user,user
users:x:100:
user:x:1001:
rvm:x:1002:
ec2-user:x:1003:
test:x:1004:
cloud_user:x:1005:
ssm-user:x:1006:
usera:x:1007:
userb:x:1008:

```

### Modifying User Settings ###

Modifying users settings: usermod

```
usermod -c "<comment>" -d <home_dir> -g <group> -p <password> -s <user_shell> username

e.g. 

$ usermod -c "John Doe" -d /jane -g appadm -p abcd1234 -s /bin/sh jdoe
```

```
[root@dyry1c ~]# man usermod
USERMOD(8)                System Management Commands                USERMOD(8)

NAME
       usermod - modify a user account

SYNOPSIS
       usermod [options] LOGIN

DESCRIPTION
       The usermod command modifies the system account files to reflect the
       changes that are specified on the command line.

OPTIONS
       The options which apply to the usermod command are:

       -a, --append
           Add the user to the supplementary group(s). Use only with the -G
           option.

       -c, --comment COMMENT
           The new value of the user's password file comment field. It is
           normally modified using the chfn(1) utility.

 Manual page usermod(8) line 1 (press h for help or q to quit)
 
 ...
 
```

```
[root@dyry1c ~]# grep jdoe /etc/passwd
jdoe:x:2000:10:Jane Doe:/jane:/bin/sh
[root@dyry1c ~]# grep jdoe /etc/group
adm:x:4:ec2-user,jdoe,jford

[root@dyry1c ~]# usermod -G users jdoe
[root@dyry1c ~]# grep jdoe /etc/group
users:x:100:jdoe
```

Add user test gorup
```
[root@dyry1c ~]# usermod -G test -a jdoe
[root@dyry1c ~]# grep jdoe /etc/group
users:x:100:jdoe
test:x:1004:jdoe
```

### Removing User Accounts ###

Removing User Accounts: userdel

Remove the user and ther home directory: 
```
userdel -r <home_dir> username
```

Search the server for files created by the user and remove the files approced for removal:
```
find <directory to serarch> -user username
```

```
[root@dyry1c ~]# man userdel
USERDEL(8)                System Management Commands                USERDEL(8)

NAME
       userdel - delete a user account and related files

SYNOPSIS
       userdel [options] LOGIN

DESCRIPTION
       The userdel command modifies the system account files, deleting all
       entries that refer to the user name LOGIN. The named user must exist.

OPTIONS
       The options which apply to the userdel command are:

       -f, --force
           This option forces the removal of the user account, even if the
           user is still logged in. It also forces userdel to remove the
           user's home directory and mail spool, even if another user uses the
           same home directory or if the mail spool is not owned by the
           specified user. If USERGROUPS_ENAB is defined to yes in
           /etc/login.defs and if a group exists with the same name as the
           deleted user, then this group will be removed, even if it is still
 Manual page userdel(8) line 1 (press h for help or q to quit)
 
 ...
 
 ```
 
 ### Creating Groups and Secondary Groups ###
 
 Creating Groups: groupadd 
 
```
groupadd -g <GID> groupname

e.g.

groupadd -g 30000 appadm

gropuadd -g 40000 dba

useradd -g appadm -G dba user1
```

``` 
root@dyry1c ~]# man groupadd
GROUPADD(8)               System Management Commands               GROUPADD(8)

NAME
       groupadd - create a new group

SYNOPSIS
       groupadd [options] group
	   
...

       -p, --password PASSWORD
           The encrypted password, as returned by crypt(3). The default is to disable the password.

           Note: This option is not recommended because the password (or encrypted password) will be visible by users listing the processes.

           You should make sure the password respects the system's password policy.

...

```

### Modifying Gropu Settings ###

Modifying Gropu Settings: groupmod

```
groupmod -g <GID> groupname

grep appadm /etc/group appadm:x:30000:

groupmod -g 35000 -n appdev appadm

grep appdev /etc/group appdev:x:35000:
```

### Removing Groups ###

Removing Groups: groupdel
```
groupdel -<options> gropuname

grep appdev /etc/group appdev:x:35000:

gropudel appdev

grep appdev /etc/group
```

```
root@dyry1c ~]# man groupdel
GROUPDEL(8)               System Management Commands               GROUPDEL(8)

NAME
       groupdel - delete a group

SYNOPSIS
       groupdel [options] GROUP

DESCRIPTION
       The groupdel command modifies the system account files, deleting all
       entries that refer to GROUP. The named group must exist.

OPTIONS
       The options which apply to the groupdel command are:

       -h, --help
           Display help message and exit.

       -R, --root CHROOT_DIR
           Apply changes in the CHROOT_DIR directory and use the configuration
           files from the CHROOT_DIR directory.
...

```
 
### User Management tools ###

- gpassed - /etc/group and /etc/gshadow
- pwck - /etc/passwd and /etc/shadow
- grpck	- /etc/group and /etc/gshadow
- pwconv, pwunconv - /etc/passwd and /etc/shadow
- grpconv, grpunconv - /etc/group and /etc/gshadow


Learning Objectives

- Create the appusers group
- Create the appuser1 user account with a UID of 2001. The gecos field should contain, “Admin1 Account for ABC Application.”
- Create the appuser2 user account with a UID of 2002. The gecos field should contain, “Admin2 Account for ABC Application."
 
```
[cloud_user@node1 ~]$ sudo su
[root@node1 cloud_user]# groupadd appusers
[root@node1 cloud_user]# useradd -u 2001 -c "Admin1 Account for ABC Application" -g appusers appuser1
[root@node1 cloud_user]# useradd -u 2002 -c "Admin2 Account for ABC Application" -g appusers appuser2
[root@node1 cloud_user]# grep appuser /etc/passwd
appuser1:x:2001:1004:Admin1 Account for ABC Application:/home/appuser1:/bin/bash
appuser2:x:2002:1004:Admin2 Account for ABC Application:/home/appuser2:/bin/bash
[root@node1 cloud_user]#
```

## Lab - Linux User Management - Modify user settings and removing groups ##
 
 ```
 [root@node1 ~]# grep dbadmin /etc/passwd
dbadmin:x:1003:1003::/home/dbadmin:/bin/bash
[root@node1 ~]# usermod -s /sbin/nologin dbadmin
[root@node1 ~]# grep dbadmin /etc/passwd
dbadmin:x:1003:1003::/home/dbadmin:/sbin/nologin
[root@node1 ~]# grep dba /etc/group
dbadmin:x:1003:
[root@node1 ~]# groupadd dba
[root@node1 ~]# grep dba /etc/group
dbadmin:x:1003:
dba:x:1004:
[root@node1 ~]# usermod -g dba dbadmin
[root@node1 ~]# grep dbadmin /etc/passwd
dbadmin:x:1003:1004::/home/dbadmin:/sbin/nologin
[root@node1 ~]# id dbadmin
uid=1003(dbadmin) gid=1004(dba) groups=1004(dba)
[root@node1 ~]# ls /
bin  boot  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@node1 ~]# mkdir /dba
[root@node1 ~]# ls
anaconda-ks.cfg  original-ks.cfg
[root@node1 ~]# ls /
bin  boot  data  dba  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@node1 ~]# ls -al /|grep dba
drwxr-xr-x.  2 root root    6 Mar 16 15:46 dba
[root@node1 ~]# chmod 740 /dba
[root@node1 ~]# ls -al /|grep dba
drwxr-----.  2 dbadmin dba     6 Mar 16 15:46 dba
[root@node1 home]# cd dbadmin
[root@node1 dbadmin]# ls -al
total 12
drwx------. 2 dbadmin dba   62 Mar 16 15:08 .
drwxr-xr-x. 6 root    root  75 Mar 16 15:08 ..
-rw-r--r--. 1 dbadmin dba   18 Mar 12  2019 .bash_logout
-rw-r--r--. 1 dbadmin dba  193 Mar 12  2019 .bash_profile
-rw-r--r--. 1 dbadmin dba  231 Mar 12  2019 .bashrc
[root@node1 dbadmin]# cp .bash* /dba
[root@node1 dbadmin]# cd /dba
[root@node1 dba]# ls -la
total 12
drwxr-----.  2 dbadmin dba   62 Mar 16 15:52 .
dr-xr-xr-x. 19 root    root 247 Mar 16 15:46 ..
-rw-r--r--.  1 root    root  18 Mar 16 15:52 .bash_logout
-rw-r--r--.  1 root    root 193 Mar 16 15:52 .bash_profile
-rw-r--r--.  1 root    root 231 Mar 16 15:52 .bashrc
[root@node1 dba]#
[root@node1 dba]# chown dbadmin:dba .bash*
[root@node1 dba]# ls -la
total 12
drwxr-----.  2 dbadmin dba   62 Mar 16 15:52 .
dr-xr-xr-x. 19 root    root 247 Mar 16 15:46 ..
-rw-r--r--.  1 dbadmin dba   18 Mar 16 15:52 .bash_logout
-rw-r--r--.  1 dbadmin dba  193 Mar 16 15:52 .bash_profile
-rw-r--r--.  1 dbadmin dba  231 Mar 16 15:52 .bashrc
[root@node1 dba]#
[root@node1 dba]# usermod -d /dba dbadmin
[root@node1 dba]# grep dbadmin /etc/passwd
dbadmin:x:1003:1004::/dba:/sbin/nologin
[root@node1 dba]#
[root@node1 /]# rm -R /home/dbadmin
rm: descend into directory ‘/home/dbadmin’? y
rm: remove regular file ‘/home/dbadmin/.bash_logout’? y
rm: remove regular file ‘/home/dbadmin/.bash_profile’? y
rm: remove regular file ‘/home/dbadmin/.bashrc’? y
rm: remove directory ‘/home/dbadmin’? y
[root@node1 /]#
```

### Lab - Linux User Management - Working with Secondary Groups ###

```
[root@node1 ~]# groupadd -g 3000 appadm
[root@node1 ~]# groupadd -g 4000 dba
[root@node1 ~]# useradd -g appadm -G dba user1
[root@node1 ~]# useradd -g appadm -G dba user2
[root@node1 ~]# useradd -g appadm -G dba user3
[root@node1 ~]# mkdir /app
[root@node1 ~]# chgrp appadm /app
[root@node1 ~]# chmod 760 /app
[root@node1 ~]# ls -al / |grep app
drwxrw----.  2 root appadm    6 Mar 16 21:32 app
[root@node1 ~]# mkdir /db
[root@node1 ~]# chgrp dba /db
[root@node1 ~]# chmod 760 /db
[root@node1 ~]# ls -al /|grep db
drwxrw----.  2 root dba       6 Mar 16 21:33 db
[root@node1 ~]# echo "This file is reserved for application configuration." > /app/app1.conf
[root@node1 ~]# ls -al /app/app1.conf
-rw-r--r--. 1 root root 53 Mar 16 21:35 /app/app1.conf
[root@node1 ~]# chgrp appadm /app/app1.conf
[root@node1 ~]# chmod 760 /app/app1.conf
[root@node1 ~]# ls -al /app/app1.conf
-rwxrw----. 1 root appadm 53 Mar 16 21:35 /app/app1.conf
[root@node1 ~]# echo "This file is reserved for database configuration." > /db/db1.conf
[root@node1 ~]# ls -al /db/db1.conf
-rw-r--r--. 1 root root 50 Mar 16 21:37 /db/db1.conf
[root@node1 ~]# chgrp dba /db/db1.conf
[root@node1 ~]# chmod 760 /db/db1.conf
[root@node1 ~]# ls -al /db
total 4
drwxrw----.  2 root dba   22 Mar 16 21:37 .
dr-xr-xr-x. 20 root root 257 Mar 16 21:33 ..
-rwxrw----.  1 root dba   50 Mar 16 21:37 db1.conf
[root@node1 ~]#
```

