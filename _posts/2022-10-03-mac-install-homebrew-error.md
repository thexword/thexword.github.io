---
layout: post
title: "mac 安装homebrew出错: Failed to connect to raw.githubusercontent.com port 443: Connection refused error 解决办法"
date: 2022-10-03 08:37:14 +0800
categories: jekyll
---

解决办法：通过修改hosts解决此问题。 

查询真实IP:

在 [ipaddress](https://www.ipaddress.com/) 查询 `raw.githubusercontent.com` 的真实IP。

修改 hosts :

`sudo vim /etc/hosts`

添加如下内容：

```
140.82.112.4 github.com 
185.199.110.133 raw.githubusercontent.com 
199.232.69.194 github.global.ssl.fastly.net
```