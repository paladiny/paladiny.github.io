---
layout:     post
title:      "v2ray 搭建"
subtitle:   "服务器端安装配置"
date:       2019-06-17 14:00:00
author:     "Willynn"
header-img: "img/in-post/2019-06-17/google.jpg"
catalog: true
tags:
    - v2ray
    - 科学上网
---



# 免密登录服务器
客户端命令： ```ssh-keygen```<br>
按3次回车生成公钥私钥。（第一次要求输入存密钥的文件名，按回车则为默认/root/.ssh/id_rsa，第二次为加密密码，按回车默认密码为空，第三次回车则是密码再次确认）。<br>
在服务器新建文件/root/.ssh/authorized_keys，把客户端公钥/root/.ssh/id_rsa.pub的内容复制到服务器authorized_keys文件中。<br>
客户端ssh链接服务器，则可不用输入密码免密登录。<br>

# GCP创建服务器
机器类型选微型，最便宜的那种。地区优先香港，其他所有配置不动，防火墙规则勾选http和https流量。<br>
创建完使用google提供的网页ssh工具登录。修改root密码:sudo passwd root,修改完成后用命令 su  获取root权限。<br>
修改ssh配置文件以便客户端登录<br>
vim /etc/ssh/sshd_config<br>
PermitRootLogin 是否允许以root账号登录<br>
PasswordAuthentication 是否允许密码登录<br>

# v2ray安装
参考文档：https://toutyrater.github.io/prep/install.html ， https://www.v2ray.com/chapter_00/install.html <br>
V2Ray 提供了一个在 Linux 中的自动化安装脚本。这个脚本会自动检测有没有安装过 V2Ray，如果没有，则进行完整的安装和配置；如果之前安装过 V2Ray，则只更新 V2Ray 二进制程序而不更新配置。<br>
```bash <(curl -L -s https://install.direct/go.sh)``` <br>
安装完后修改配置文件 ```vim /etc/v2ray/config.json``` <br>


