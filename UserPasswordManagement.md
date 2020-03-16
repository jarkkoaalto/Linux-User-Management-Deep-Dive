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

