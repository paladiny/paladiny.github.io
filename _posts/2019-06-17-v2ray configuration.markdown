---

layout:     post
title:      "v2ray 搭建"
subtitle:   "服务器端安装配置"
date:       2019-06-17 14:00:00
author:     "Willynn"
header-img: "img/in-post/2019-06-17/google.png"
catalog: true
tags:
    - v2ray
    - 科学上网

---



##### 免密登录服务器
客户端命令： `ssh-keygen`<br>
按3次回车生成公钥私钥。（第一次回车时要求输入存密钥的文件名，按回车则为默认/root/.ssh/id_rsa，第二次为加密密码，按回车默认密码为空，第三次回车则是密码再次确认）。
在服务器新建文件/root/.ssh/authorized_keys，把客户端公钥/root/.ssh/id_rsa.pub的内容复制到服务器authorized_keys文件中。
客户端ssh链接服务器，则可不用输入密码免密登录。

##### GCP创建服务器
机器类型选微型，最便宜的那种。地区优先香港，其他所有配置不动，防火墙规则勾选http和https流量。
创建完使用google提供的网页ssh工具登录。修改root密码:sudo passwd root,修改完成后用命令 su  获取root权限。
修改ssh配置文件以便客户端登录
vim /etc/ssh/sshd_config
PermitRootLogin 是否允许以root账号登录
PasswordAuthentication 是否允许密码登录
service sshd restart `重启sshd`

##### nginx安装
```apt-get update```更新Debian OS源
```apt-get install nginx```从Debian OS源安装nginx
默认安装在/etc/nginx, nginx.conf为系统配置文件，nginx/sites-available/default为可用的web项目配置文件，nginx/sites-enabled/default是nginx/sites-available/default的软链接，系统读取sites-available/default作为配置文件。

##### Let's Encrypt安装ssl证书
参考https://certbot.eff.org/lets-encrypt/debianstretch-nginx
添加源 deb http://deb.debian.org/debian buster-backports main 到/etc/apt/sources.list 
```apt-get update``` 更新源
```apt-get install certbot python-certbot-nginx -t stretch-backports``` 安装Certbot 
```certbot --nginx``` 获取证书并且让certbot自动配置nginx 
安装时有几个选择和输入项：
1，输入邮箱
2，是否同意协议。
3，是否愿意公开邮箱地址给Electronic Frontier Foundation用来接收相关资讯和信息邮件
4，属于域名
安装完成后，
证书位置：/etc/letsencrypt/live/域名/fullchain.pem
密钥：/etc/letsencrypt/live/域名/privkey.pem
证书有效期3个月，Certbot会自动更新证书有效期，```certbot renew --dry-run```可用于测试自动更新是否可用。


##### v2ray安装
参考文档：https://toutyrater.github.io/prep/install.html ， https://www.v2ray.com/chapter_00/install.html 
V2Ray 提供了一个在 Linux 中的自动化安装脚本。这个脚本会自动检测有没有安装过 V2Ray，如果没有，则进行完整的安装和配置；如果之前安装过 V2Ray，则只更新 V2Ray 二进制程序而不更新配置。
```bash <(curl -L -s https://install.direct/go.sh)``` 
安装完后修改配置文件 ```vim /etc/v2ray/config.json``` 


