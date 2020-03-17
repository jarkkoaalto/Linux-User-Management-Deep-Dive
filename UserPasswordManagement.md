# User Password Management #

## User password - Hashes and Salted ##

Hashed passwords in /etc/shadow:
```
cloud_user:$5$awdlGKLOTro$folyiksgllh:19249:0:99999:7:::
$id$salt$hashed
```
username hashed-password-with-salt  last min warn inactive expire

- id for the hashing algorithm 1) is MD5, 2a is blowfish, 3y is Blowfish, 5 is SHA-256, 6 is SHA-512
- Salt: A salt is a fixed-lenght cryptograpgically-strong random value that is added to the password hash to make the password stronger and prevent hacking attempts.
- hash: hashing password string

## Managing User Passwords ##

- change password - passwd
- unlocking password - passwd -u <user>
- delete password - passwd -d <user>
- expire password - password - e <user>
- rxpire days password - passwd - i 30 <user>
- minimum time when user can change password - passwd -n 1 <user>
- maximum time when user have to change password - passwd -x 90 <user>
- warning user - passwd -w 7 <user>
- show password status - passwd -S <user> 

## Password Aging ##

Password aging in /etc/shadow: 
```
cloud_user:$5$awdlGKLOTro$folyiksgllh:19249:0:99999:7:::
```

Password aging in chage:

chage -l cloud_user

|   Description                                     |            time            |
| ------------------------------------------------- | -------------------------- |
| Last password change                              | Dec 19, 2019               |
| Password expires                                  | Jan 18, 2020               |
| Password inactivate                               | Mar 18, 2020               |
| Account expires                                   | Never                      |
| Minimum number of days between password change    | 0                          |
| Maximum number of days between password change    | 30                         |
| Number of days of warning before password expires | 7                          |

```
[cloud_user@dyry1c ~]$ chage -h
Usage: chage [options] LOGIN

Options:
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -h, --help                    display this help message and exit
  -I, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -l, --list                    show account aging information
  -m, --mindays MIN_DAYS        set minimum number of days before password
                                change to MIN_DAYS
  -M, --maxdays MAX_DAYS        set maximum number of days before password
                                change to MAX_DAYS
  -R, --root CHROOT_DIR         directory to chroot into
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
  ```

  ## Suspending User Account ##

  - usermod
  - passwd
  - chage

  The passwd -l usernaeme and usermod -L username commands lock a user's password, but not the full account. It enters a ! in the /etc/shadow file at the beginning of the encrypted password to make it unreadable. Users can still log in by other means, such as SSH keys.

  Warning:

  Never edit the /etc/shadow file manually. If you mudt edit the file, make a backup copy first with cp /etc/shadow /etc/shadow.bak.date

  User chage -E 0 username for full account locking: To set a date for the account to expire, you can use: usermod -e YYYY-MM-DD username

  ##  Hands-on Lab: Linux User Management: Configuring Password Aging ##

  ```
  [cloud_user@node1 ~]$ sudo -i
[sudo] password for cloud_user:
[root@node1 ~]# chage -l cloud_user
Last password change                                    : Mar 17, 2020
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[root@node1 ~]# chage -M 30 cloud_user
[root@node1 ~]# chage -l cloud_user
Last password change                                    : Mar 17, 2020
Password expires                                        : Apr 16, 2020
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 30
Number of days of warning before password expires       : 7
[root@node1 ~]# chage -I 60 cloud_user
[root@node1 ~]# chage -l cloud_user
Last password change                                    : Mar 17, 2020
Password expires                                        : Apr 16, 2020
Password inactive                                       : Jun 15, 2020
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 30
Number of days of warning before password expires       : 7
```
