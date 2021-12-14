---
layout: post
title:  "Docker私有镜像仓库"
date:   2021-12-13
categories: ['raspberry', 'docker', 'private docker registry']
---
记录下轻量级的官方方式安装*私有Docker镜像*的步骤, 把常用的镜像缓存下来.

- 系统 _Ubuntu 20.04 LTS_ 
- 硬件 _树莓派4B 2G_

# 安装*Docker*
```bash
sudo apt update
sudo apt install docker.io
```

# 安装*Registry*

```bash
mkdir /etc/docker/registry/
## 把里面的账号密码换成自己的
sudo tee /etc/docker/registry/config.yml << EOF
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
proxy:
  remoteurl: https://registry-1.docker.io
  user: <user>
  password: <password>
EOF

sudo chmod 600 /etc/docker/registry/config.yml 
sudo docker run -d -p 5000:5000 --restart=always --name registry \
             -v /etc/docker/registry/config.yml:/etc/docker/registry/config.yml \
             registry:2

## 查看日志
docker logs -f registry
## 重启registry
docker restart registry
```

这样其实就可以通过下面的方式直接用了
```bash
docker pull 127.0.0.1:5000/library/alpine
```
但是不够方便.

# 安装配置Nginx
我们可以在前面放个*Nginx*反向代理

```bash
sudo apt install nginx-full
## 证书这一块用openssl ca搞得docker不认 所以我用了Windows Server PKI生成的
## 你也可以用letsencrypt生成 https://letsencrypt.org/zh-cn/getting-started/
sudo tee /etc/nginx/sites-available/docker << EOF
server {
  listen   443 ssl;
  server_name  docker.home.lan;

  ssl_certificate      ssl/server.crt;
  ssl_certificate_key  ssl/server.key;
  access_log   /var/log/nginx/docker-access.log;

  location / {
    proxy_pass http://127.0.0.1:5000/;
    proxy_set_header Host \$host;
    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
  }
}
EOF

# 验证下nginx 配置文件
sudo nginx -t
# 重启Nginx
systemctl restart nginx
```

# 配置客户端
然后在本机上或你想使用这个仓库的机器上配置下*registry-mirrors*

```bash
## 如果你是私有证书需要安装下CA
mkdir /usr/local/share/ca-certificates/extra/
cp home-ca.crt /usr/local/share/ca-certificates/extra/
update-ca-certificates

sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://docker.home.lan"]
}
EOF
sudo systemctl restart docker

## 测试下
sudo docker pull centos

## 可以去看下nginx的日志
tail -f /var/log/nginx/docker-access.log
```

这样默认就会去配置好的*mirrors*里面拿镜像.



参考:
- https://docs.docker.com/registry/configuration/
- https://docs.docker.com/registry/recipes/mirror/