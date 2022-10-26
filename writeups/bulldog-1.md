[vulnhub - easy/medium] bulldog: 1
----------------------------------

### enumeration
Since the machine's ip is already shown at its login page, I decide to check it on my browser to see if there's any useful information in it:
![bulldog 1](./images/bulldog-1.png)
But there isn't relevant info on the page (even after reading the source code) nor in the "Public Notice" link above the bulldog photo (except for text, that is, which could be used in CeWL if it comes to that). Running exiftoon on the photo also doesn't give me anything useful, just the cheeky nod:
```bash
Comment                         : Not a part of the VM, he's just cute :3 https://www.pexels.com/photo/white-and-brown-bulldog-on-brown-wood-planks-160748/
```
I then proceed to run nmap on all ports:
```bash
┌──(j㉿kali)-[~/Desktop/vulnhub/bulldog-1]
└─$ nmap -p- -T4 -oA nmap_info_allports 192.168.56.117
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-26 11:36 -03
Nmap scan report for 192.168.56.117
Host is up (0.00014s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
23/tcp   open  telnet
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 0.87 seconds
```
After that, detailed nmap on the detected ports:
```bash
┌──(j㉿kali)-[~/Desktop/vulnhub/bulldog-1]
└─$ nmap -p 23,80,8080 -sC -sV -oA nmap_info_detailed 192.168.56.117
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-26 11:38 -03
Nmap scan report for 192.168.56.117
Host is up (0.00062s latency).

PORT     STATE SERVICE VERSION
23/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20:8b:fc:9e:d9:2e:28:22:6b:2e:0e:e3:72:c5:bb:52 (RSA)
|   256 cd:bd:45:d8:5c:e4:8c:b6:91:e5:39:a9:66:cb:d7:98 (ECDSA)
|_  256 2f:ba:d5:e5:9f:a2:43:e5:3b:24:2c:10:c2:0a:da:66 (ED25519)
80/tcp   open  http    WSGIServer 0.1 (Python 2.7.12)
|_http-title: Bulldog Industries
|_http-server-header: WSGIServer/0.1 Python/2.7.12
8080/tcp open  http    WSGIServer 0.1 (Python 2.7.12)
|_http-title: Bulldog Industries
|_http-server-header: WSGIServer/0.1 Python/2.7.12
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.03 seconds
zsh: segmentation fault  nmap -p 23,80,8080 -sC -sV -oA nmap_info_detailed 192.168.56.117
```
The port 8080 is serving the same thing as the port 80, so i proceed to use gobuster to bruteforce probable directories.
```bash
``` 