# Nully Cybersecurity 1: Writeup

Hello, this is my first writeup, while studying for the OSCP certification. This VM (found [here](https://www.vulnhub.com/entry/nully-cybersecurity-1,549/)) was the most enjoyable so far, so I decided to create a writeup on it. It is a nice easy-intermediate level machine, that taught me a lot about pivoting and the metasploit framework.


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

lets do that first. I will use netcat to connect to the pop3 port of the mail server and some basic pop3 commands:
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
exit
-ERR Unknown command: EXIT
^C
teo@ThinkPad:~$ 
```


