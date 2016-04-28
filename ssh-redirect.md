#如何在本地调试微信推送的回调消息—SSH端口转发

# 准备材料
```
1. 外网可以访问的服务器一台，在上面装好了Nginx
2. 微信公众号一个
```

# 烹饪过程
```
服务器
1. 修改服务器配置Nginx
2. SSH配置

本地
1. 本地SSH反向监听
```
---

## 服务器
### 1. 修改nginx.conf:

假设我们的微信公众号的回调url是 http://www.domain.com/weixin/

```
server {
    listen 80 ; # 微信只会发消息到回调地址的80端口
    server_name www.domain.com;

    access_log  /srv/www/www.domain.com/logs/nginx_access.log;
    error_log   /srv/www/www.domain.com/logs/nginx_error.log;

    location ^~ /weixin/ { # 由于我填的微信的回调url是 http://www.domain.com/weixin/，所以在这个location去处理微信消息
        proxy_pass http://127.0.0.1:8099; # 把weiwin回调url 发来的消息 转发到某端口，这里取8099
        access_log  /srv/www/www.domain.com/logs/nginx_weixin_access.log;
        error_log /srv/www/www.domain.com/logs/nginx_weixin_error.log;
    }

    location / {
        proxy_pass_header X-CSRFToken;
        include uwsgi_params;
        uwsgi_pass unix:///var/tmp/uwsgi_xz-test.sock;
    }
}
```

### 2. 修改/etc/ssh/sshd_config
打开gatewayports，没有的话就直接加上

```
GatewayPorts yes
```

## 本地：
### 1. 打开远程端口的转发
执行命令

```
ssh -CNfR 8099:127.0.0.1:8000 root@192.168.123.123
```
即把远程服务器192.168.123.123 8099端口的请求，转发到本地127.0.0.1:8000，端口要与服务器的Nginx转发端口保持一致。

---

#### ps： 为了保持ssh连接，添加心跳

当ssh断了之后本地重新打开ssh可能不能正常监听，一般会报以下警告

```
Warning: remote port forwarding failed for listen port 8099
```

所以添加心跳以保持连接。

```
vi ~/.ssh/config (create if not exist)
```
添加以下

```
Host *
ServerAliveInterval 240
```

