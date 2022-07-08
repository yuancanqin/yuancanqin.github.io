---
layout:     post
title:      Cooperation-EEAProject-SettingUp-via-EA&MySQL
subtitle:   Server-Deployment-MySQL
date:       2022-07-08
author:     Richie
header-img: img/server-command.jpg
catalog: true
tags:
    - EEAProject MySQL
---

# 1.	Apply the Server Resource or Workstation 
Log into Server or start up the workstation,  install MySQL Server, and set up Database.

# 2.	Installation Package Preparation
1) MySQL Server Download,
https://downloads.mysql.com/archives/community/
Version Recommend: 5.5.62, 64 bit

2) Enterprise Architect Schema Creation Scripts Download,
https://sparxsystems.com/resources/repositories/index.html
Version Recommend: EASchema_1220_MySQL.sql

# 3.	MySQL Server Installation
MySQL Server Installation reference as below,
https://www.cnblogs.com/DeepInThought/p/10726857.html

# 4.	Create a Project in a MySQL Database
1) Open MySQL Client and log in with PWD;

2) Create Database and import Database script
```
mysql> create database abc;      # Create Database, named: abc
mysql> use abc;                  # using the database just been created 
mysql> set names utf8;           # set encoding
mysql> source /home/abc/EASchema_1220_MySQL.sql       # import the EEA project script
```

# 5.	troubleshooting
The error message 'Got a packet bigger than 'MAX_allowed_packet' bytes' may be displayed when the local project is imported to the Server Database. The solution is as follows: 
```
show VARIABLES like ‘%max_allowed_packet%’;     # check the max_allowed_packet value
Result showing as ：
+——————–+———+
| Variable_name | Value |
+——————–+———+
| max_allowed_packet | 1048576 |
+——————–+———+
set global max_allowed_packet = 1048576*10;       # Set the max_allowed_packet value to 10 times larger
show VARIABLES like ‘%max_allowed_packet%’;    # Log out and log in again the MySQL Command Line, and then the max_allowed_packet variable has been increased. But, the default values will be restore after restarting the Server or Workstation. 
```



