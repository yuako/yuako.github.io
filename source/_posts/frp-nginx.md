---
title: 使用frp和nginx实现内网穿透
date: 2019-07-01 23:02:44
tags: 
    - frp
    - nginx
categories:
    - develop
---

之前项目需要测试第三方支付，基本上都需要外网地址，如何让本机地址穿透内网绑定域名用于外网访问，于是找到了frp。
frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp, http, https 协议。


**准备**

需要一个域名，指向到外网固定IP服务器，二级域名也可以，例如：*.sub.host.com
使用泛解析，就是上面的*.

**部署**

下载对应操作系统的frp服务端&客户端至服务器和内网电脑
地址：https://github.com/fatedier/frp/releases/

```bash
# 请从github获取最新地址
wget https://github.com/fatedier/frp/releases/download/v0.27.0/frp_0.27.0_linux_amd64.tar.gz
```

解压

```bash
tar zxf ./frp_0.27.0_linux_amd64.tar.gz
cd ./frp_0.16.1_linux_amd64
```

编辑配置文件

```bash
vim ./frps.ini
```

可参考如下配置:

```bash
# 服务器端监听客户端连接请求的端口
bind_port = 7000

# 服务器端监听http请求的端口(由于80端口被nginx占用,因此指定其他端口)
vhost_http_port = 8080

# 服务器用以显示连接状态的站点端口,以下配置中可以通过访问IP:7500登录查看frp服务端状态等信息
dashboard_addr = 0.0.0.0
dashboard_port = 7500

# dashboard对应的用户名/密码
dashboard_user = use
dashboard_pwd = pwd

# 日志文件路径
log_file = ./frps.log

# 日志记录错误级别,分为:trace, debug, info, warn, erro
log_level = warn

# 日志保存最大天数
log_max_days = 3

# 客户端连接校验码(客户端需与之相同)
privilege_token = privilege_token

# heartbeat configure, it's not recommended to modify the default value
# the default value of heartbeat_timeout is 90
# heartbeat_timeout = 90

# only allow frpc to bind ports you list, if you set nothing, there won't be any limit
# privilege_allow_ports = 2000-3000,3001,3003,4000-50000

# pool_count in each proxy will change to max_pool_count if they exceed the maximum value
max_pool_count = 5

# max ports can be used for each client, default value is 0 means no limit
max_ports_per_client = 0

# authentication_timeout means the timeout interval (seconds) when the frpc connects frps
# if authentication_timeout is zero, the time is not verified, default is 900s
authentication_timeout = 900

# 支持外部访问的域名(需要将域名解析到IP)
subdomain_host = sub.host.com
```

配置nginx反向代理,将来自80端口并指向*.sub.host.com的请求分发至frp服务器http请求的监听端口

```bash
server {
    listen       80;
    server_name *.sub.host.com;

    location / {
        proxy_pass  http://127.0.0.1:8080;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_max_temp_file_size 0;
        proxy_redirect off;
        proxy_read_timeout 240s;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

启动frp服务器并后台运行,启动完成后可通过lsof -i :7000查看端口占用情况

```bash
nohup ./frps -c ./frps.ini &
```

### 客户端

1. 创建目录并解压
2. 编辑配置文件

```bash
[common]
server_addr = 服务器IP
server_port = 7000
privilege_token = privilege_token
auth_token = auth_token
[hccrm]
type = http
local_port = 80
subdomain = hccrm
```

启动frp客户端程序

```bash
./frpc -c ./frpc.ini
```

本地apache/nginx虚拟主机配置域名别名(alias),根据自己环境而定

