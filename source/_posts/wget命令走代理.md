---
title: wget命令走代理
date: 2023-10-03 16:26:47
description: wget下载文件走代理
tags:
- linux工具
- wget
categories:
- linux工具
mathjax: true
---

# Wget走代理

## 方法一 设置环境变量

```bash
export http_proxy=http://127.0.0.1:7890
```

## 方法二 修改配置文件

修改**/etc/wgetrc**文件，或者将其与proxy有关的语句复制到~/.wgetrc中，然后做如下修改

```
#You can set the default proxies for Wget to use for http, https, and ftp.
# They will override the value in the environment.
https_proxy = http://127.0.0.1:8087/
http_proxy = http://127.0.0.1:8087/
ftp_proxy = http://127.0.0.1:8087/
 
# If you do not want to use proxy at all, set this to off.
use_proxy = on
```

也可以使用命令行修改

**-Y, --proxy=on/off           打开或关闭代理**
