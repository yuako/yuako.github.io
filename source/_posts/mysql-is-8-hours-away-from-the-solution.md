---
title: mysql相差8小时解决方案
date: 2018-03-29 11:40:55
tags: 
    - mysql
categories:
    - mysql
---

1. 修改连接池
```
jdbc:mysql://localhost:3306/dcoj?serverTimezone=Asia/Shanghai
```

2. 修改Mysql配置
```
show variables like '%time_zone%';//查询当前时区
set global time_zone='+8:00';//在标准时区上加+8小时,即东8区时间
```

3. 修改配置文件
```
[mysqld]   
default-time-zone=+8:00
```