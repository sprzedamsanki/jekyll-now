---
layout: post
title: JIS-CTF VulnUpload
---
Hello! Welcome to another writeup. Today I'm gonna show how I broke [VulnUpload](https://www.vulnhub.com/entry/jis-ctf-vulnupload,228/). According to description:
>Description: There are five flags on this machine. Try to find them. It takes 1.5 hour on average to find all flags.

## Recon
Without futher addo let's see what services are visible on the machine:
```bash
# nmap 192.168.0.101 -p- -Pn -sT -sV -O -T4

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-16 13:48 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.0.101
Host is up (0.00031s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
MAC Address: 08:00:27:46:17:5E (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.74 seconds
```

As expected there's a HTTP server on port 80. Main page presents a login page without any obvious flaws so I try my luck in robots.txt file:

```
User-agent: *
Disallow: /
Disallow: /backup
Disallow: /admin
Disallow: /admin_area
Disallow: /r00t
Disallow: /uploads
Disallow: /uploaded_files
Disallow: /flag
```

## 1st flag
Navigating to http://192.168.0.101/flag/ presents us the first flag:
```
The 1st flag is : {8734509128730458630012095}
```

## 2nd flag
Under http://192.168.0.101/admin_area/ (also known from robots.txt) there's another flag and login credentials:
```
<!--	username : admin
	password : 3v1l_H@ck3r
	The 2nd flag is : {7412574125871236547895214}
-->
```

## 3rd flag
Login credentials from admin_area can be used on index page of server. Doing so gives access to a file upload form. Quick test reveals that uploaded files appear in /uploaded_files folder under their respectful names. I decide to upload PentestMonkey's php-reverse-shell available [here](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php). Navigating to the auploaded shell script opens connection to limited shell.

Using the shell we can snoop around victim's disk, and discover another flag:
```
$ cd /var/www/html
$ ls -al
total 60
drwxr-xr-x 8 www-data www-data 4096 Apr 21  2017 .
drwxr-xr-x 3 www-data www-data 4096 Apr 18  2017 ..
drwxrwxr-x 2 www-data www-data 4096 Apr 21  2017 admin_area
drwx------ 5 www-data www-data 4096 Apr 19  2017 assets
-rw-r--r-- 1 www-data www-data  306 Apr 19  2017 check_login.php
drwx------ 2 www-data www-data 4096 Apr 19  2017 css
drwxr-xr-x 2 www-data www-data 4096 Apr 21  2017 flag
-rw-r----- 1 technawi technawi  132 Apr 21  2017 flag.txt
-rw-r--r-- 1 www-data www-data  145 Apr 21  2017 hint.txt
-rw-rw-r-- 1 www-data www-data 1966 Apr 19  2017 index.php
drwx------ 2 www-data www-data 4096 Apr 19  2017 js
-rw-rw-r-- 1 www-data www-data 1485 Apr 19  2017 login.php
-rw-r--r-- 1 www-data www-data  128 Apr 19  2017 logout.php
-rw-rw-r-- 1 www-data www-data  160 Apr 19  2017 robots.txt
drwxrwxr-x 2 www-data www-data 4096 Mar 16 20:26 uploaded_files
$ cat hint.txt
try to find user technawi password to read the flag.txt file, you can find it in a hidden file ;)

The 3rd flag is : {7645110034526579012345670}
```

## 4th flsh
Above location also contains last flag but in order to read it we need to login as technawi user. Hint claims that technawi password is stored in some hidden file. I don't feel like searching it by hand so I decide to automate the process:
```
$ grep -A 1 -B 1 -ri password /etc 2>/dev/null
[...]
/etc/mysql/conf.d/credentials.txt-username : technawi
/etc/mysql/conf.d/credentials.txt:password : 3vilH@ksor
[...]
$ cat /etc/mysql/conf.d/credentials.txt
The 4th flag is : {7845658974123568974185412}

username : technawi
password : 3vilH@ksor
```

## 5th flag

Now, all that's left to do is to log in through SSH using given credentials and claim last flag:
```
root@Jordaninfosec-CTF01:~# cd /var/www/html
root@Jordaninfosec-CTF01:/var/www/html# cat flag.txt
The 5th flag is : {5473215946785213456975249}

Good job :)

You find 5 flags and got their points and finish the first scenario....
```

## Summary

That's all! Thanks for reading, and kudos to the challenge's Author, [Mohammad Khreesha](https://www.vulnhub.com/author/mohammad-khreesha,579/)! :)
## Flags
```
{8734509128730458630012095}
{7412574125871236547895214}
{7645110034526579012345670}
{7845658974123568974185412}
{5473215946785213456975249}
```
