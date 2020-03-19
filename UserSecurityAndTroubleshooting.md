# User Security and Troubleshooting #

## Configuring Sudo ##

Configuring sudo: visudo

Use visudo to edit the /etc/sudoers file or create configuration files in /etc/sudoers.d

```
[root@dyry1c ~]# cd /etc/
[root@dyry1c etc]# ll sudo*
-rw-r-----. 1 root root 1786 Oct 16 04:04 sudo.conf
-r--r-----. 1 root root 4033 Oct 15  2018 sudoers
-r--r-----. 1 root root 4328 Sep 25  2018 sudoers.rpmnew
-rw-r-----. 1 root root 3181 Oct 16 04:04 sudo-ldap.conf

sudoers.d:
total 12
-r--r-----. 1 root root 7520 Mar 11 15:44 90-cloud-init-users
-r--r-----. 1 root root   58 Oct  8  2018 ssm-agent-users
[root@dyry1c etc]# more sudo.conf
#
# Default /etc/sudo.conf file
#
# Format:
#   Plugin plugin_name plugin_path plugin_options ...
#   Path askpass /path/to/askpass
#   Path noexec /path/to/sudo_noexec.so
#   Debug sudo /var/log/sudo_debug all@warn
#   Set disable_coredump true
#
# Sudo plugins:
#
# The plugin_path is relative to ${prefix}/libexec unless fully qualified.
# The plugin_name corresponds to a global symbol in the plugin
#   that contains the plugin interface structure.
# The plugin_options are optional.
#
# The sudoers plugin is used by default if no Plugin lines are present.
Plugin sudoers_policy sudoers.so
Plugin sudoers_io sudoers.so

...

```

```
root@dyry1c etc]# diff sudoers sudoers.rpmnew
33c33
< # Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig
---
> # Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start, /usr/bin/systemctl stop, /usr/bin/systemctl reload, /usr/bin/systemctl restart, /usr/bin/systemctl status, /usr/bin/systemctl enable, /usr/bin/systemctl disable
53,60c53
< # Disable "ssh hostname sudo <cmd>", because it will show the password in clear.
< #         You have to run "ssh -t hostname sudo <cmd>".
< #
< Defaults    requiretty
<
< #
< # Refuse to run if unable to disable echo on the tty. This setting should also be
< # changed in order to be able to use sudo without a tty. See requiretty above.
---
> # Refuse to run if unable to disable echo on the tty.
71a65,73
> Defaults    match_group_by_gid
>
> # Prior to version 1.8.15, groups listed in sudoers that were not
> # found in the system group database were passed to the group
> # plugin, if any. Starting with 1.8.15, only groups of the form
> # %:group are resolved via the group plugin by default.
> # We enable always_query_group_plugin to restore old behavior.
> # Disable this option for new behavior.

...

```

## Managing User Files and Processes ##

1. Search for and remove any files owned by testuser1: find / ~user testuser1, rm /path/filename
2. Search for any processes owned by testuser1 and kill them: ps U testuser1, kill <PID>
3. Remove testuser1 and their home directory from the system: ps U userdel -r testuser1

```
[root@dyry1c etc]# useradd testuser1
[root@dyry1c etc]# useradd testuser2
[root@dyry1c etc]# grep testuser /etc/passwd
testuser1:x:2002:2002::/home/testuser1:/bin/bash
testuser2:x:2003:2003::/home/testuser2:/bin/bash
[root@dyry1c etc]# touch /tmp/testfile1
[root@dyry1c etc]# chown testuser1:testuser1 /tmp/testfile1
[root@dyry1c etc]# touch /var/tmp/test.sh
[root@dyry1c etc]# chown testuser2:testuser2 /var/tmp/test.sh
root@dyry1c etc]# runuser -u testuser1 sleep 1800 &
[1] 2995
[root@dyry1c etc]# runuser -u testuser2 sleep 1800 &
[2] 2997

[root@dyry1c etc]# cd /home/
[root@dyry1c home]# ll
total 8
drwx------. 10 cloud_user cloud_user 4096 Mar 19 05:08 cloud_user
drwx------.  4 ec2-user   ec2-user     85 May 14  2018 ec2-user
drwx------.  3 ssm-user   ssm-user     74 Oct  8  2018 ssm-user
drwx------.  3 test       test         74 Jun 25  2018 test
drwx------.  2 testuser1  testuser1    59 Mar 19 05:29 testuser1
drwx------.  2 testuser2  testuser2    59 Mar 19 05:29 testuser2
drwx------. 11 user       user       4096 Apr 10  2019 user
drwx------.  2 usera      usera        59 Mar 11 16:05 usera
[root@dyry1c home]# ps U testuser1
  PID TTY      STAT   TIME COMMAND
 2996 pts/0    S      0:00 sleep 1800
[root@dyry1c home]# kill 2996
Terminated
[root@dyry1c home]# userdel -r testuser1
[1]-  Exit 143                runuser -u testuser1 sleep 1800  (wd: /etc)
(wd now: /home)
[root@dyry1c home]# cd /home
[root@dyry1c home]# ll
total 8
drwx------. 10 cloud_user cloud_user 4096 Mar 19 05:08 cloud_user
drwx------.  4 ec2-user   ec2-user     85 May 14  2018 ec2-user
drwx------.  3 ssm-user   ssm-user     74 Oct  8  2018 ssm-user
drwx------.  3 test       test         74 Jun 25  2018 test
drwx------.  2 testuser2  testuser2    59 Mar 19 05:29 testuser2
drwx------. 11 user       user       4096 Apr 10  2019 user
drwx------.  2 usera      usera        59 Mar 11 16:05 usera

```

```
[root@dyry1c home]# ps U testuser2
  PID TTY      STAT   TIME COMMAND
 2998 pts/0    S      0:00 sleep 1800
[root@dyry1c home]# find / -user testuser2
/proc/2998
/proc/2998/task
/proc/2998/task/2998
/proc/2998/task/2998/fd
/proc/2998/task/2998/fd/0
/proc/2998/task/2998/fd/1

...

/var/tmp/test.sh
/home/testuser2
/home/testuser2/.bash_logout
/home/testuser2/.bash_profile
/home/testuser2/.bashrc
[root@dyry1c home]#

[root@dyry1c home]# rm /var/tmp/test.sh
rm: remove regular empty file ‘/var/tmp/test.sh’? y
[root@dyry1c home]# userdel -r testuser2
```


## Managing file permissions ##

Managing file permissions: chmod and chown

1. Create the testdir dicetory and the testfile1 file: mkdir testdir > touch testfile1
2. Change the owner fo testdir with the chown command: chown test testdir
3. Change permissions to  testdir with the chmod command: chmod a+rwx testdir > chomod 777 testdir

```
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
[root@dyry1c ~]# mkdir testdir
[root@dyry1c ~]# touch testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-r--r--. 1 root root          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chown test:test testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-r--r--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod --help
Usage: chmod [OPTION]... MODE[,MODE]... FILE...
  or:  chmod [OPTION]... OCTAL-MODE FILE...
  or:  chmod [OPTION]... --reference=RFILE FILE...
Change the mode of each FILE to MODE.
With --reference, change the mode of each FILE to that of RFILE.

  -c, --changes          like verbose but report only when a change is made
  -f, --silent, --quiet  suppress most error messages
  -v, --verbose          output a diagnostic for every file processed
      --no-preserve-root  do not treat '/' specially (the default)
      --preserve-root    fail to operate recursively on '/'
      --reference=RFILE  use RFILE's mode instead of MODE values
  -R, --recursive        change files and directories recursively
      --help     display this help and exit
      --version  output version information and exit

Each MODE is of the form '[ugoa]*([-+=]([rwxXst]*|[ugo]))+|[-+=][0-7]+'.

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
For complete documentation, run: info coreutils 'chmod invocation'
```

```
[root@dyry1c ~]# chmod u+x testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rwxr--r--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod u=rw testfile
chmod: cannot access ‘testfile’: No such file or directory
[root@dyry1c ~]# chmod u=rw testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-r--r--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod g+wx testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-rwxr--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod go-r testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw--wx---. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod go+rw testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]#
```

```
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rwxr--r--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod u=rw testfile
chmod: cannot access ‘testfile’: No such file or directory
[root@dyry1c ~]# chmod u=rw testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-r--r--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod g+wx testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-rwxr--. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod go-r testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw--wx---. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod go+rw testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:24 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# touch testdir/testfile2
[root@dyry1c ~]# chmod -R 777 testdir
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwx. 2 root root         22 Mar 19 06:31 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# ll testdir
total 0
-rwxrwxrwx. 1 root root 0 Mar 19 06:31 testfile2
[root@dyry1c ~]# chmod u+s testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwx. 2 root root         22 Mar 19 06:31 testdir
-rwSrwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod u-s testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwx. 2 root root         22 Mar 19 06:31 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod g+s testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwx. 2 root root         22 Mar 19 06:31 testdir
-rw-rwsrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod g-s testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwx. 2 root root         22 Mar 19 06:31 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]# chmod o+t testdir
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwt. 2 root root         22 Mar 19 06:31 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:24 testfile1
[root@dyry1c ~]#
```

## Intro to Access Control List (ACLs) ##

Intro to Access Control List (ACLs): getfacl and setfacl

- Get the file access list for the testfile1 file: getfacl testfile1
- Set the file access list for the testfile1 file: setfacl -m g:test:rw testfile1

```
root@dyry1c ~]# touch testfile1
[root@dyry1c ~]# touch testdir/testfile1
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxrwxrwt. 2 root root         38 Mar 19 06:39 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:39 testfile1
[root@dyry1c ~]# umask
0022
[root@dyry1c ~]# mkdir test2
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:42 test2
drwxrwxrwt. 2 root root         38 Mar 19 06:39 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:39 testfile1
[root@dyry1c ~]# touch testnewfile
[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:42 test2
drwxrwxrwt. 2 root root         38 Mar 19 06:39 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:39 testfile1
-rw-r--r--. 1 root root          0 Mar 19 06:43 testnewfile
[root@dyry1c ~]# getfacl testfile1
# file: testfile1
# owner: test
# group: test
user::rw-
group::rwx
other::rw-

[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:42 test2
drwxrwxrwt. 2 root root         38 Mar 19 06:39 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:39 testfile1
-rw-r--r--. 1 root root          0 Mar 19 06:43 testnewfile

[root@dyry1c ~]# getfacl testfile1
# file: testfile1
# owner: test
# group: test
user::rw-
group::rwx
other::rw-

[root@dyry1c ~]# ll
total 3970208
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:42 test2
drwxrwxrwt. 2 root root         38 Mar 19 06:39 testdir
-rw-rwxrw-. 1 test test          0 Mar 19 06:39 testfile1
-rw-r--r--. 1 root root          0 Mar 19 06:43 testnewfile
[root@dyry1c ~]# setfacl -m mask:rw testfile1
[root@dyry1c ~]# ll
total 3970212
-rw-------. 1 root root       9227 Oct 17  2014 anaconda-ks.cfg
-rw-r--r--. 1 root root  175262413 Apr  3  2018 jdk-8u171-linux-x64.rpm
-rw-r--r--. 1 root root 3890216960 Feb 19  2015 rhel-server-7.1-x86_64-dvd.iso
drwxr-xr-x. 2 root root          6 Mar 19 06:42 test2
drwxrwxrwt. 2 root root         38 Mar 19 06:39 testdir
-rw-rw-rw-+ 1 test test          0 Mar 19 06:39 testfile1
-rw-r--r--. 1 root root          0 Mar 19 06:43 testnewfile
[root@dyry1c ~]# getfacl testfile1
# file: testfile1
# owner: test
# group: test
user::rw-
group::rwx                      #effective:rw-
mask::rw-
other::rw-
```

## Enabling User and Group Disk Quotas ##

Enabling users and Gropu Disk Quotas: gettacheck, edquota, quotaon, repquota

Gonfigure a system disk with a new 2GB partion. Sync the system disks, and view the newly created partion: system disk >
- lsblk
- fdisk /dev/xvdb
- partprobe /dev/xvdb
- lsblk

Create the filesystem and then mount point: 
- mkfs -t ext4 /dev/svdb1
- mkdir /app

Get the UUID for the disk and use it to configure /etc/fstab with a group and user quota. Then mount the filesystem and change permissions on the mount point. 
- blkid /dev/svdbl
- vim /etc/fstab
- mount -a
- chown test:test /app

Install the quota package, crate the quota files, and assign the user and group qoutas for the filesystem. 
- yum install -y quota
- qoutacheck -cug /app
- edquota test
- edquota -g test

Enable the user and group qoutas on the /app filesystem and verify the qouta configuration.
- quotaon -vug /app
- repquota /app

## User Authentication and Logging ##

User authentication logging: /var/log/secure

Linux log are stored in the /var/log directory:
- The secure file logs all authentication related system events.
- The maillog file logs all main server system event.
- The cron file lgos all cron scheculer system events.
- The boot.log file logs all system startup events.
- The messages file logs all ther system events.

```
[root@dyry1c ~]# more /var/log

*** /var/log: directory ***

[root@dyry1c ~]# grep "uid >= 1000" /etc/pam.d/*
/etc/pam.d/password-auth:auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
/etc/pam.d/password-auth-ac:auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
/etc/pam.d/system-auth:auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
/etc/pam.d/system-auth-ac:auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
[root@dyry1c ~]# grep gloud_user /var/log/secure
[root@dyry1c ~]# grep cloud_user /var/log/messages
Mar 11 17:22:09 dyry1c systemd: Stopping Session c2 of user cloud_user.
Mar 12 04:16:15 dyry1c systemd: Created slice User Slice of cloud_user.
Mar 12 04:16:15 dyry1c systemd: Started Session c1 of user cloud_user.
Mar 12 04:16:20 dyry1c runuser: New 'dyry1c.mylabserver.com:1 (cloud_user)' desktop is dyry1c.mylabserver.com:1
Mar 12 04:16:20 dyry1c runuser: Starting applications specified in /home/cloud_user/.vnc/xstartup
Mar 12 04:16:20 dyry1c runuser: Log file is /home/cloud_user/.vnc/dyry1c.mylabserver.com:1.log
Mar 12 04:16:23 dyry1c systemd: Started Session 1 of user cloud_user.
Mar 12 04:16:23 dyry1c systemd-logind: New session 1 of user cloud_user.
Mar 12 04:16:31 dyry1c journal: Unable to get contents of the bookmarks file: Error opening file /home/cloud_user/.gtk-bookmarks: No such file or directory
Mar 12 04:16:40 dyry1c systemd-logind: New session 2 of user cloud_user.
Mar 12 04:16:40 dyry1c systemd: Started Session 2 of user cloud_user.
Mar 12 04:17:45 dyry1c su: (to root) cloud_user on pts/0
Mar 12 06:54:46 dyry1c systemd: Stopping Session c1 of user cloud_user.
Mar 17 04:36:08 dyry1c systemd: Created slice User Slice of cloud_user.
Mar 17 04:36:08 dyry1c systemd: Started Session c1 of user cloud_user.
Mar 17 04:36:12 dyry1c runuser: New 'dyry1c.mylabserver.com:1 (cloud_user)' desktop is dyry1c.mylabserver.com:1
Mar 17 04:36:12 dyry1c runuser: Starting applications specified in /home/cloud_user/.vnc/xstartup
Mar 17 04:36:12 dyry1c runuser: Log file is /home/cloud_user/.vnc/dyry1c.mylabserver.com:1.log
Mar 17 04:36:21 dyry1c systemd-logind: New session 1 of user cloud_user.
Mar 17 04:36:21 dyry1c systemd: Started Session 1 of user cloud_user.
Mar 17 04:36:22 dyry1c journal: Unable to get contents of the bookmarks file: Error opening file /home/cloud_user/.gtk-bookmarks: No such file or directory
Mar 17 04:36:23 dyry1c systemd-logind: New session 2 of user cloud_user.
Mar 17 04:36:23 dyry1c systemd: Started Session 2 of user cloud_user.
Mar 19 05:08:48 dyry1c systemd: Created slice User Slice of cloud_user.
Mar 19 05:08:48 dyry1c systemd: Started Session c1 of user cloud_user.
Mar 19 05:08:50 dyry1c systemd: Started Session 1 of user cloud_user.
Mar 19 05:08:50 dyry1c systemd-logind: New session 1 of user cloud_user.
Mar 19 05:08:53 dyry1c runuser: New 'dyry1c.mylabserver.com:1 (cloud_user)' desktop is dyry1c.mylabserver.com:1
Mar 19 05:08:53 dyry1c runuser: Starting applications specified in /home/cloud_user/.vnc/xstartup
Mar 19 05:08:53 dyry1c runuser: Log file is /home/cloud_user/.vnc/dyry1c.mylabserver.com:1.log
Mar 19 05:09:04 dyry1c journal: Unable to get contents of the bookmarks file: Error opening file /home/cloud_user/.gtk-bookmarks: No such file or directory
Mar 19 05:11:29 dyry1c systemd-logind: New session 5 of user cloud_user.
Mar 19 05:11:29 dyry1c systemd: Started Session 5 of user cloud_user.
Mar 19 06:22:49 dyry1c systemd: Created slice User Slice of cloud_user.
Mar 19 06:22:49 dyry1c systemd: Started Session c1 of user cloud_user.
Mar 19 06:22:51 dyry1c systemd: Started Session 1 of user cloud_user.
Mar 19 06:22:51 dyry1c systemd-logind: New session 1 of user cloud_user.
Mar 19 06:22:53 dyry1c runuser: New 'dyry1c.mylabserver.com:1 (cloud_user)' desktop is dyry1c.mylabserver.com:1
Mar 19 06:22:53 dyry1c runuser: Starting applications specified in /home/cloud_user/.vnc/xstartup
Mar 19 06:22:53 dyry1c runuser: Log file is /home/cloud_user/.vnc/dyry1c.mylabserver.com:1.log
Mar 19 06:23:05 dyry1c journal: Unable to get contents of the bookmarks file: Error opening file /home/cloud_user/.gtk-bookmarks: No such file or directory
Mar 19 06:23:55 dyry1c systemd: Started Session 3 of user cloud_user.
Mar 19 06:23:55 dyry1c systemd-logind: New session 3 of user cloud_user.
[root@dyry1c ~]#
```

## Hands-on Lab: Linux user management: Configure sudo ##

### Learning Objectives
- Create userA and userB with the wheel Group as Their Secondary Group
- Add the wheel Group as a Secondary Group for the ec2-user. Since This User Is Already Configured on the System, Make Sure to Append the New Secondary Group as to Not Overwrite Prior Existing Secondary Groups
- Verify All Users Are Part of the wheel Group
- Configure the wheel Group in /etc/sudoers. Since This Is Configured by Default to Provide Users in the wheel Group with root Access for All Commands, No Action Is Needed. Just Verify the Wheel Entry 
- Set the Password for userA and userB. Grep ec2-user from the /etc/shadow File and if a Password isn't Set, then Set One.
- Switch to Each User and Verify sudo Commands Are Executed by root

```
root@node1 ~]# useradd -G wheel userA
[root@node1 ~]# useradd -G wheel userB
[root@node1 ~]# usermod -aG wheel ec2-user
[root@node1 ~]# groups userA userB ec2-user
userA : userA wheel
userB : userB wheel
ec2-user : ec2-user adm wheel systemd-journal

[root@node1 ~]# grep wheel /etc/group
wheel:x:10:ec2-user,linuxacademy,cloud_user,userA,userB

[root@node1 ~]# visudo
visudo: /etc/sudoers.tmp unchanged

[root@node1 ~]# passwd userA
Changing password for user userA.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

[root@node1 ~]# passwd userB
Changing password for user userB.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

[root@node1 ~]# grep ec2-user /etc/shadow
ec2-user:!!:17639:0:99999:7:::
[root@node1 ~]# passwd ec2-user
Changing password for user ec2-user.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

[root@node1 ~]# su - userA
[userA@node1 ~]$ sudo whoami

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for userA:
root
[userA@node1 ~]$ whoami
userA
[userA@node1 ~]$ sudo whoami
root
[userA@node1 ~]$ exit
logout

[root@node1 ~]# su - userB
[userB@node1 ~]$ su - userB
Password:
Last login: Thu Mar 19 11:23:27 UTC 2020 on pts/0
[userB@node1 ~]$ sudo whoami

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for userB:
root
[userB@node1 ~]$ exit
logout

[userB@node1 ~]$ su - ec2-user
Password:
[ec2-user@node1 ~]$ sudo whoami
root
```

## Hands-on lab: Linux user management: Managing user files and processes ##
### Learning Objectives ###

- Search for Any Running Processes Owned by the test User. Kill the Process by the Process ID (PID)
- Search the System for All Files Owned by the test User, Then Change the Owner and Group to ec2-user on Any Files That Are outside of the Home Directory and the Mail Spool
- Remove the test User and Home Directory from the System
- Verify That the Home Directory and the Mail Spool Were Removed. Also Verify That There Are No Other Files Owned by the test User on the System

```
[root@server1 ~]# useradd test
[root@server1 ~]# ps U test
  PID TTY      STAT   TIME COMMAND
[root@server1 ~]# find / -user test
find: ‘/proc/11324/task/11324/fd/6’: No such file or directory
find: ‘/proc/11324/task/11324/fdinfo/6’: No such file or directory
find: ‘/proc/11324/fd/5’: No such file or directory
find: ‘/proc/11324/fdinfo/5’: No such file or directory
/var/tmp/appconfig.sh
/var/spool/mail/test
/tmp/list.txt
/home/test
/home/test/.bash_logout
/home/test/.bash_profile
/home/test/.bashrc
[root@server1 ~]# chown ec2-user:ec2-user /var/tmp/appconfig.sh
[root@server1 ~]# chown ec2-user:ec2-user /tmp/list.txt
[root@server1 ~]# chown ec2-user:ec2-user /var/tmp/appconfig.sh
[root@server1 ~]# find / -user test
find: ‘/proc/11418/task/11418/fd/6’: No such file or directory
find: ‘/proc/11418/task/11418/fdinfo/6’: No such file or directory
find: ‘/proc/11418/fd/5’: No such file or directory
find: ‘/proc/11418/fdinfo/5’: No such file or directory
/var/spool/mail/test
/home/test
/home/test/.bash_logout
/home/test/.bash_profile
/home/test/.bashrc
[root@server1 ~]# userdel -r test
[root@server1 ~]# find / -user test
find: ‘test’ is not the name of a known user
[root@server1 ~]# cd /home
[root@server1 home]# ls
cloud_user  ec2-user  linuxacademy
[root@server1 home]# ls /var/spool/mail
```

## Hands-on lab: Linux user management: Configuring Group Disk Quotas ##

 ### Learning Objectives ###

- Create the app Group as Well as appuser1 and appuser2, and Set the app Group as the Primary Group for Both Users
- Create a Filesystem on One of the 2 GB Drives Configured on the System
- Create the Directory for the Mount Point and Change the Group to app
- Configure /etc/fstab with the UUID of /dev/xvdb1, Mount the Filesystem, and change the group of the filesystem to app.
- Install the quota Package, Create the Quota Files for the /app Filesystem, and Generate the Table of Current Disk Usage for Each Filesystem
- Assign a Soft Quota of 512 KB and a Hard Quota of 1M (1024KB) to the app Group, Then Turn Quotas on and Check the Group Quota Configuration for /app

```
[root@server1 ~]# groupadd app
[root@server1 ~]# useradd -g app appuser1
[root@server1 ~]# useradd -g app appuser2
[root@server1 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk
├─xvda1 202:1    0   1M  0 part
└─xvda2 202:2    0  10G  0 part /
xvdb    202:16   0   2G  0 disk
xvdc    202:32   0   2G  0 disk
xvdd    202:48   0   2G  0 disk
[root@server1 ~]# fdisk /dev/xvdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xed952eab.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-4194303, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303):
Using default value 4194303
Partition 1 of type Linux and of size 2 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@server1 ~]# partprobe
[root@server1 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk
├─xvda1 202:1    0   1M  0 part
└─xvda2 202:2    0  10G  0 part /
xvdb    202:16   0   2G  0 disk
└─xvdb1 202:17   0   2G  0 part
xvdc    202:32   0   2G  0 disk
xvdd    202:48   0   2G  0 disk
[root@server1 ~]# mkfs -t ext4 /dev/xvdb1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131072 inodes, 524032 blocks
26201 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[root@server1 ~]# mkdir /app
[root@server1 ~]# blkid /dev/xvdb1
/dev/xvdb1: UUID="5d4a3006-6095-4e11-b207-cf8311a93e9e" TYPE="ext4"
[root@server1 ~]#


login as: cloud_user
Keyboard-interactive authentication prompts from server:
| Password:
End of keyboard-interactive prompts from server
[cloud_user@server1 ~]$ sudo -i
[sudo] password for cloud_user:
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]#
[root@server1 ~]# groupadd app
[root@server1 ~]# useradd -g app appuser1
[root@server1 ~]# useradd -g app appuser2
[root@server1 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk
├─xvda1 202:1    0   1M  0 part
└─xvda2 202:2    0  10G  0 part /
xvdb    202:16   0   2G  0 disk
xvdc    202:32   0   2G  0 disk
xvdd    202:48   0   2G  0 disk
[root@server1 ~]# fdisk /dev/xvdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xed952eab.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-4194303, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303):
Using default value 4194303
Partition 1 of type Linux and of size 2 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@server1 ~]# partprobe
[root@server1 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  10G  0 disk
├─xvda1 202:1    0   1M  0 part
└─xvda2 202:2    0  10G  0 part /
xvdb    202:16   0   2G  0 disk
└─xvdb1 202:17   0   2G  0 part
xvdc    202:32   0   2G  0 disk
xvdd    202:48   0   2G  0 disk
[root@server1 ~]# mkfs -t ext4 /dev/xvdb1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131072 inodes, 524032 blocks
26201 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[root@server1 ~]# mkdir /app
[root@server1 ~]# blkid /dev/xvdb1
/dev/xvdb1: UUID="5d4a3006-6095-4e11-b207-cf8311a93e9e" TYPE="ext4"
[root@server1 ~]# vim /etc/fstab

#
# /etc/fstab
# Created by anaconda on Fri Mar 23 17:41:14 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=50a9826b-3a50-44d0-ad12-28f2056e9927 /                       xfs     defaults        0 0
UUID=5d4a3006-6095-4e11-b207-cf8311a93e9e /app ext4 defaults,grpquota 1 2
                         
[root@server1 ~]# mount -a	
[root@server1 ~]# chgrp app /app
[root@server1 ~]# yum install -y quota
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
rhui-REGION-client-config-server-7                                                                                                                                                                                    | 2.9 kB  00:00:00
rhui-REGION-rhel-server-extras                                                                                                                                                                                        | 3.4 kB  00:00:00
rhui-REGION-rhel-server-releases                                                                                                                                                                                      | 3.5 kB  00:00:00
rhui-REGION-rhel-server-rh-common                     

....

[root@server1 ~]# edquota -g app
Disk quotas for group app (gid 1004):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/xvdb1                        4         512         1024          1        0        0
~
~

[root@server1 ~]# quotaon -vug /app
/dev/xvdb1 [/app]: group quotas turned on
[root@server1 ~]#

[root@server1 ~]# repquota -g /app
*** Report for group quotas on device /dev/xvdb1
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
Group           used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --      16       0       0              1     0     0
app       --       4     512    1024              1     0     0

```

## Hands-on lab: Linux user management: Troubleshooting Login Issues ##


### Learning Objectives ###

- List the Users in the app Group, Then Verify the User Configuration in the /etc/passwd and /etc/shadow Files
- The Double Exclamation Points (!!) in the Password Field of /etc/shadow Indicates That user1 Has Never Had a Password Set, so Set the Password for user1
- The 0 in the 8th Field of the /etc/shadow File for user2 Indicates that Account Has Expired and Needs to Be Unlocked
- The Configuration for user3 Is Missing from the /etc/shadow File, so Recreate the shadow File with the pwconv Command
- Verify That All Users Can Log into the System

### Additional Information and Resources ###
ABC Company has just hired a team of application administrators to manage a new application. Today is their first day, but none of the new hires are able to log in to the server. The user accounts and the group for the team were created by a consultant who has since left the organization. The team manager has asked that you look at the logins for the appteam group that was created and resolve their issues.

To complete this lab, check the authentication logs as well as the /etc/passwd and /etc/shadow files to diagnose the issue for each of the users in the app group. Hint: each account could have a separate login issue. Good luck!

Please use the lab environment for this exercise, and not the Cloud Playground. To gain root access, log into the lab environment with the cloud_user account and issue sudo -i.

```
[root@server1 ~]# grep app /etc/group
app:x:1004:
[root@server1 ~]# grep 1004 /etc/passwd
user1:x:1004:1004::/home/user1:/bin/bash
user2:x:1005:1004::/home/user2:/bin/bash
user3:x:1006:1004::/home/user3:/bin/bash
[root@server1 ~]# grep user* /etc/shadow
ec2-user:!!:17639:0:99999:7:::
cloud_user:$6$SMjOLrgR$vBQktz5hzZMUe6gvNqj.IllpP4QHXRgM7dBX/3tZyIUrB/Irm8d02yjawTW.Ws80ui4LCupCuTO5TAInbUSrA0:18340:0:99999:7:::
user1:!!:18340:0:99999:7:::
user2:$6$fnw08MEV$mLBgk5fU8daQVb1v7FPN3KbGCYAD/FS49atnKRVj0sa9/6hmEVzM/S6cZEqaLFSr1dKsOSSj1zlnS9EmIasoJ/:18340:0:99999:7::0:
[root@server1 ~]# passwd user1
Changing password for user user1.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@server1 ~]# grep user* /etc/shadow
ec2-user:!!:17639:0:99999:7:::
cloud_user:$6$SMjOLrgR$vBQktz5hzZMUe6gvNqj.IllpP4QHXRgM7dBX/3tZyIUrB/Irm8d02yjawTW.Ws80ui4LCupCuTO5TAInbUSrA0:18340:0:99999:7:::
user1:$6$HTELocU8$.3mB/GFXheAumQZqf6/UpQhM8hIn9O6gHln3uroIGhnN2GSmDSuzbP85dKADUp6FNbb4O0w.XrX6eifP78oSV0:18340:0:99999:7:::
user2:$6$fnw08MEV$mLBgk5fU8daQVb1v7FPN3KbGCYAD/FS49atnKRVj0sa9/6hmEVzM/S6cZEqaLFSr1dKsOSSj1zlnS9EmIasoJ/:18340:0:99999:7::0:
[root@server1 ~]# chage -E -1 user2
[root@server1 ~]# grep user* /etc/shadow
ec2-user:!!:17639:0:99999:7:::
cloud_user:$6$SMjOLrgR$vBQktz5hzZMUe6gvNqj.IllpP4QHXRgM7dBX/3tZyIUrB/Irm8d02yjawTW.Ws80ui4LCupCuTO5TAInbUSrA0:18340:0:99999:7:::
user1:$6$HTELocU8$.3mB/GFXheAumQZqf6/UpQhM8hIn9O6gHln3uroIGhnN2GSmDSuzbP85dKADUp6FNbb4O0w.XrX6eifP78oSV0:18340:0:99999:7:::
user2:$6$fnw08MEV$mLBgk5fU8daQVb1v7FPN3KbGCYAD/FS49atnKRVj0sa9/6hmEVzM/S6cZEqaLFSr1dKsOSSj1zlnS9EmIasoJ/:18340:0:99999:7:::
[root@server1 ~]# grep user3 /etc/passwd
user3:x:1006:1004::/home/user3:/bin/bash
[root@server1 ~]# pwconv
[root@server1 ~]# grep user3 /etc/shadow
user3:x:18340:0:99999:7:::
[root@server1 ~]# passwd user3
Changing password for user user3.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@server1 ~]# grep user* /etc/shadow
ec2-user:!!:17639:0:99999:7:::
cloud_user:$6$SMjOLrgR$vBQktz5hzZMUe6gvNqj.IllpP4QHXRgM7dBX/3tZyIUrB/Irm8d02yjawTW.Ws80ui4LCupCuTO5TAInbUSrA0:18340:0:99999:7:::
user1:$6$HTELocU8$.3mB/GFXheAumQZqf6/UpQhM8hIn9O6gHln3uroIGhnN2GSmDSuzbP85dKADUp6FNbb4O0w.XrX6eifP78oSV0:18340:0:99999:7:::
user2:$6$fnw08MEV$mLBgk5fU8daQVb1v7FPN3KbGCYAD/FS49atnKRVj0sa9/6hmEVzM/S6cZEqaLFSr1dKsOSSj1zlnS9EmIasoJ/:18340:0:99999:7:::
user3:$6$wtsnXWxq$AhFdgmoUXXQ0Mi1KrEZgydZDrCsQRXTxIZfUSWVuembrD9GxDrcCfY/ZX1ZqJC3iTsgliQEW/aLZsh1UZFFKV.:18340:0:99999:7:::
[root@server1 ~]# grep user* /etc/passwd
ec2-user:x:1000:1000:Cloud User:/home/ec2-user:/bin/bash
cloud_user:x:1002:1002::/home/cloud_user:/bin/bash
user1:x:1004:1004::/home/user1:/bin/bash
user2:x:1005:1004::/home/user2:/bin/bash
user3:x:1006:1004::/home/user3:/bin/bash
[root@server1 ~]#
```





