---

layout:     post
title:      "Let's Encrypt通配符证书申请"
subtitle:   ""
date:       2020-06-22 18:16:00
author:     "Willynn"
header-img: ""
catalog: true
tags:
    - HTTPS
    - Let's Encrypt
    - ubuntu
    

---



### 环境
Nginx on Ubuntu 18.04 LTS (bionic)<br>

### 安装certbot
添加certbot PPA到repository
```
apt-get update
apt-get install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt-get update
```
<br>

安装certbot<br>
```apt-get install certbot python3-certbot-nginx```
<br>

### 申请证书
```
certbot certonly -d "*.domain.com" --manual --preferred-challenges dns-01   
# 无需使用server参数来指定ACME Directory Resource URI,默认即为https://acme-v02.api.letsencrypt.org/directory
```
```- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - ```
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): #在此填入邮箱地址
```
```- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - ```
```
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: #按A同意
```
```- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - ```
```
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: #按N不订阅新闻
```
```- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - ```
```
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for domain.com
```
```- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - ```
```
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: #按Y同意
```
```- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - ```
```
Please deploy a DNS TXT record under the name
_acme-challenge.domain.com with the following value:

SORJ5souY0hVvQVASTSukwo405st02tikA-TAUIAzqk

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
#此处不要按回车，复制value并在域名提供商或者DNS提供商处新增一条record,如下图
```
![新增record](/img/in-post/2020-06-22/acme-challenge.png "新增record")<br>

record建好后，另开一个ssh窗口并连接到服务器，用dig命令检查新增的record是否生效，域名提供商namesilo一般要15分钟左右生效。
```
dig -t txt _acme-challenge.domain.com @8.8.8.8

; <<>> DiG 9.11.3-1ubuntu1.12-Ubuntu <<>> -t txt _acme-challenge.domain.com @8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17145
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;_acme-challenge.domain.com. IN  TXT

;; ANSWER SECTION:
_acme-challenge.domain.com. 7206 IN  TXT "qze5qLa0my0rb8q_fPTTbD_TE5B3gZovQBuBNO3Tqf0"

;; Query time: 341 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Jun 22 09:45:24 UTC 2020
;; MSG SIZE  rcvd: 112

```
查看
```
;; ANSWER SECTION:
_acme-challenge.domain.com. 7206 IN  TXT "qze5qLa0my0rb8q_fPTTbD_TE5B3gZovQBuBNO3Tqf0"
```
这段的txt值是否是刚才新增record里填入的值，如果不是则说明record还未生效，等待一段时间后重复使用dig来验证，直至txt的value值相同。<br>

确认生效后，在申请证书的ssh窗口按回车继续
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.willynn.com with the following value:

SORJ5souY0hVvQVASTSukwo405st02tikA-TAUIAzqk

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue  #按回车继续
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/willynn.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/willynn.com/privkey.pem
   Your cert will expire on 2020-09-20. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```
到此，证书申请成功
