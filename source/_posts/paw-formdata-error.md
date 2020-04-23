---
title: paw formdata提交问题
date: 2019-07-23 12:09:42
tags:
    - paw
    - form-data
---

paw如果默认multipart提交数据，后端无法获取正常数据

问题来源于header中的content-type引起，重新设置后正常