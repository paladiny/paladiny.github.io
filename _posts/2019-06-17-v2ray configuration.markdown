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



### GCP创建服务器
机器类型选微型，最便宜的那种。地区优先香港，其他所有配置不动，防火墙规则勾选http和https流量。<br>
创建完使用google提供的网页ssh工具登录。修改root密码:```sudo passwd root```,修改完成后用命令 su  获取root权限。<br>
修改ssh配置文件以便客户端登录```vim /etc/ssh/sshd_config```<br>
PermitRootLogin 是否允许以root账号登录<br>
PasswordAuthentication 是否允许密码登录<br>
service sshd restart `重启sshd`<br>


### 免密登录服务器
客户端命令： `ssh-keygen`<br>
按3次回车生成公钥私钥。（第一次回车时要求输入存密钥的文件名，按回车则为默认/root/.ssh/id_rsa，第二次为加密密码，按回车默认密码为空，第三次回车则是密码再次确认）。<br>
在服务器新建文件/root/.ssh/authorized_keys，把客户端公钥/root/.ssh/id_rsa.pub的内容复制到服务器authorized_keys文件中。<br>
客户端ssh链接服务器，则可不用输入密码免密登录。<br>

### 查看服务器回城线路 
参考文档：https://www.xiaoz.me/archives/11769<br>
```wget https://cdn.ipip.net/17mon/besttrace4linux.zip```下载BestTrace，其他对应版本可在https://www.ipip.net/ 找到<br>
```unzip besttrace4linux.zip```解压<br>
```chmod +x besttrace```给besttrace文件添加执行权限<br>
```./besttrace -q1 -g cn IP```执行该命令可查看回程路线 <br>
```./besttrace --help```可查看命令参数<br>

### nginx安装
```
apt-get update #更新Debian OS源
apt-get install nginx #从Debian OS源安装nginx
```
默认安装在/etc/nginx, nginx.conf为系统配置文件，nginx/sites-available/<br>default为可用的web项目配置文件，nginx/sites-enabled/default是nginx/sites-available/default的软链接，系统读取sites-available/default作为配置文件。<br>

### Let's Encrypt安装ssl证书
参考https://certbot.eff.org/lets-encrypt/debianstretch-nginx<br>
```
deb http://deb.debian.org/debian buster-backports main #添加源到/etc/apt/sources.list
apt-get update #更新源
apt-get install certbot python-certbot-nginx -t stretch-backports #安装Certbot
certbot --nginx #获取证书并且让certbot自动配置nginx
```
安装时有几个选择和输入项：<br>
1. 输入邮箱<br>
2. 是否同意协议。<br>
3. 是否愿意公开邮箱地址给Electronic Frontier Foundation用来接收相关资讯和信息邮件<br>
4. 输入域名<br>
安装完成后，<br>
证书位置：/etc/letsencrypt/live/域名/fullchain.pem<br>
密钥：/etc/letsencrypt/live/域名/privkey.pem<br>
证书有效期3个月，Certbot会自动更新证书有效期，```certbot renew --dry-run```可用于测试自动更新是否可用。<br>


### v2ray安装
参考文档：https://toutyrater.github.io/prep/install.html ， https://www.v2ray.com/chapter_00/install.html <br>
V2Ray 提供了一个在 Linux 中的自动化安装脚本。这个脚本会自动检测有没有安装过 V2Ray，如果没有，则进行完整的安装和配置；如果之前安装过 V2Ray，则只更新 V2Ray 二进制程序而不更新配置。<br>
```bash <(curl -L -s https://install.direct/go.sh)``` <br>


### v2ray+tls+websocket配置
参考文档：新V2Ray白话文指南 https://guide.v2fly.org/<br>

##### nginx配置
修改nginx安装目录下sites-available/default 文件，在SSL configuration配置项里，添加<br>
```
location /test { # 随便填，与 V2Ray 配置中的 path 保持一致即可
        proxy_redirect off;
        proxy_pass http://127.0.0.1:10000; #假设WebSocket监听在环回地址的10000端口上
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        # Show real IP in v2ray access.log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

##### v2ray配置
修改配置文件 ```vim /etc/v2ray/config.json``` <br>
```
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [{
    "port": 10000,
    "listen": "127.0.0.1", //只监听 127.0.0.1，避免除本机外的机器探测到开放了 10000 端口
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "33333333-3333-3333-3333-333333333333", //可以自行找工具生成uuid，也可用安装v2ray时自带的工具生成 /usr/bin/v2ray/v2ctl uuid
          "level": 1,
          "alterId": 64
        },
        {
          "id": "22222222-2222-2222-2222-222222222222",
          "level": 1,
          "alterId": 64
        },
        {
          "id": "11111111-1111-1111-1111-111111111111",
          "level": 1,
          "alterId": 64
        }

      ]
    },
    "streamSettings": {
        "network": "ws",
        "wsSettings": {
            "path":"/test" //与nginx的location保持一致
        }
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}

```

### 开启Google BBR
参考文档：https://zocodev.com/debian-9-enable-google-bbr.html<br>
BBR需要Linux系统内核为4.9以上，Debian 9 系统内核为4.9，自带BBR，无需安装，只需配置开启即可。
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```
如果正常执行之后没有报错的话，需要重启一下系统，让配置生效。<br>
重启之后，执行如下命令查看配置是否成功生效：<br>
```
root@zocodev:~# lsmod | grep bbr
tcp_bbr                20480  0
```
如果有如上类似输出，则证明Google BBR配置成功了。<br>

### 客户端设置
官网有各平台图形客户端推荐：https://www.v2ray.com/awesome/tools.html<br>
本人使用工具：<br>
iOS：[Shadowrocket](https://itunes.apple.com/us/app/shadowrocket/id932747118?mt=8)<br>
MacOS：[V2rayU](https://github.com/yanue/V2rayU)<br>
Windows：[V2RayN](https://github.com/2dust/v2rayN)<br>
路由：梅林改固件软件中心里的科学上网插件可使用v2ray<br>
配置参数（不同工具的参数名不一定完全一样）：<br>
节点类型:Vmess<br>
address:填写nginx配置的域名<br>
port:443<br>
id:对应服务器v2ray配置文件中的id<br>
alterId:对应服务器v2ray配置文件中的alterId<br>
security:auto即可,也可用chacha20<br>
network:ws<br>
remarks:别名，随便填，方便自己识别<br>
path:对应服务器v2ray配置文件中的path<br>
底层传输安全:tls<br>

### 监测是否被墙
https://www.toolsdaquan.com/ipcheck/ <br>
http://ping.pe/ <br>