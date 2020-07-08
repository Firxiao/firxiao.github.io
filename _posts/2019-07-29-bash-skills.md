---
layout: post
title: "Bash 小技巧"
date:   2019-07-29 20:00:00 +0800
categories: ['bash']
---

收录些`Bash`小技巧.

# 好玩的放在最前面
```bash
# 星球大战
telnet towel.blinkenlights.nl

# 命令行生成二维码
# http://qrenco.de
curl qrenco.de/'http://blog.firxiao.com'

# 命令行录像
https://asciinema.org

# cd忽略大小写/纠错
shopt -s cdspell
```

# 文本处理
```bash
# 统计file1中没有，file2中有的行
grep -vwf file1 file2

# 抓取IP
grep -oP "(\d+\.){3}\d+"

# 抓取主机名(长)
grep -oP --color=auto "([a-z]+[0-9]+\.)([a-z]+\.)([a-z]+)"

# 抓取主机名(短)(适合短主机名为字母加数字)
grep -oP --color=auto "([a-z]+[0-9]+)"

# 两行变一行
sed 'N;s/\n/ /'

# tail -f 过滤
tail -f file|grep --line-buffered 'key words' 

# n行变一行 n>1
awk 'NR%n{printf "%s ",$0;next;}1'

# 小写转大写并每隔2个字符添加':'
echo e947a256d474c3f18fbb7b6ab30750f3e6706914|tr 'a-z' 'A-Z'|sed -e 's/\(..\)/\1:/g' -e 's/:$//g'
E9:47:A2:56:D4:74:C3:F1:8F:BB:7B:6A:B3:07:50:F3:E6:70:69:14

# awk 按指定列统计
awk '{a[$1]+=1}END{for (i in a)print i,a[i]}' file

# sort 按列排序
sort -u -t, -k1,1 file

# awk转换txt为csv
awk '{for(i=1;i<=NF;i++) {if(i<NF) printf $i",";else print $i}}' file

# 多行变一行
awk BEGIN{RS=EOF}'{gsub(/\n/," ");print}' file

# 注释行(指定行首添加#)
sed 's/^keyword/#&/g' file
# 消除注释(指定行首删除#)
sed 's/^#\(keyword\)/\1/g' file
```

# Json 处理
```bash
# jq https://stedolan.github.io/jq/
# test.json
{
    "id": 1,
    "details": {
        "username": "jamesbrown",
        "name": "James Brown"
    }
}
# 过滤 filter
cat test.json |jq 'select(.details.name == "James Brown")|.id'
# 过滤值 contains
cat test.json |jq 'select(.details.name | contains ("James"))|.details.name'
# 取值 keys
jq -r "keys[]"
```

# 进程查看
```bash
#查看进程启动时间
ps -p $(ps -ef|grep process_name |grep -v grep|awk '{print $2}') -o lstart

# 查看进程cgroup
ps xawf -eo pid,user,cgroup,args

# 查看占用内存前5进程
ps -eo pmem,pcpu,vsize,pid,cmd | sort -k 1 -nr | head -5
```

# 文件处理
```bash
# 免交互更改sudoer
sudo bash -c 'echo "foobar ALL=(ALL:ALL) ALL" | (EDITOR="tee -a" visudo)'

# 查找可执行文件
find <dir> -executable
find <dir> -executable -type f

# 删除7天前的文件
find /cache  -type f -atime +7 -print0 | xargs -0 rm

# rsync over ssh 通过ssh使用rsync
rsync --progress -ave ssh  user@remoteserver:/src_dir /dest_dir

# 删除大量小文件
rsync -a --delete blanktest/ test/

# 去重(通过文件大小)
cd $PATH
ls -l |awk '{print $5,$NF}'|sort -k 1| awk 'a[$1]++{print $2}'| xargs -t -i rm -f {}

# 去重(通过md5)
find $PATH -type f -exec md5sum {} \; | sort -k 1 | awk 'a[$1]++{print $2}' | xargs -t -i rm -f {}
```

# 网络
```bash
# 配置代理
# /etc/profile.d/proxy.sh
export http_proxy="http://username:password@proxy:port"
export https_proxy="http://username:password@proxy:port"
export no_proxy="127.0.0.1, localhost, *.example.com"

# 下载目录
wget  -e robots=off -r -nH --cut-dirs=目录层级 --no-parent --reject="index.html*"  url

# 查看广播包
tcpdump -e -i eth1 ip    broadcast
tcpdump -e -i eth1 ether broadcast

# 统计指定端口tcp连接数
netstat -atn|grep "ES"|grep 端口|awk '{print $5}'|cut -d : -f 1|sort -n|uniq -c

# tcp端口检查
nc -tvz [ ip or hostname] port

# udp端口检查
nc -uvz [ ip or hostsname ] port

# iptables 限制单ip tcp连接数至1000
-A INPUT -p tcp -m state --state NEW -m connlimit --connlimit-above 1000 --connlimit-mask 32 -j LOG_REJECT
-A LOG_REJECT -j LOG --log-prefix "LOG_REJECT:" --log-level 6
-A LOG_REJECT -p tcp -j REJECT --reject-with tcp-reset

# 定制 iftop
# 保存.iftoprc至家目录
# config file for iftop, save it to .iftorc in home directory
dns-resolution: no
port-resolution: no
show-bars: yes
promiscuous: yes
port-display: on
hide-source: no
hide-destination: no
use-bytes: yes
line-display: one-line-both
show-totals: yes
log-scale: yes

#tcpdump 检查ping是否响应
tcpdump -i eth0 icmp 
```

# ssh
```bash
# 本地转发
# C 可以ssh访问 B , B 可以访问 A 的 3389 端口 
# 现在 C 要访问 A 的 3389 端口
# 转发 A 的 3389 端口至 C 的 7001 端口
# 在C上执行
ssh -L 7001:A:3389 user@B

# .ssh/config
Host B
HostName B
LocalForward 7001 A:3389

# 远程转发
# B 可以ssh访问 A ， A 无法ssh访问 B
# 现在 A 要ssh访问 B
# 转发 B 的22端口至 A 的2222
# 在 B 上执行
ssh -R 2222:B:22 user@A

# .ssh/config
Host B
Hostname B
RemoteForward 2222 B:22

# socks5 proxy
DynamicForward 8080

# A通过跳板机B scp C上的文件到本地 
# 在A上执行
scp -o ProxyCommand="ssh root@B nc C 22" -r user@C:/tmp/xxx /tmp/xxx
```

# 时间
```bash
# date获取上个月
date '+%Y %m' | awk '{if($2==1){$1--;$2=12}else $2--;printf "%d%02d\n",$1,$2}'

# 时区
ln -s -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 查看当前时区
date +%:z
```

# 磁盘
```bash
# 清空分区
dd if=/dev/urandom of=/dev/sdx bs=512 count=64
```

# SSL 证书
```bash
# 免费ssl证书
https://letsencrypt.org/getting-started/
https://certbot.eff.org/

# rhel 安装自制证书
cp *.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust

# 签发通配符证书
certbot-auto certonly  -d "*.example.com" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory
```

# Git
```
# 自定义commit时间
git commit --date="10 day ago" -m "Your commit message" 
git commit --date="10 hour ago" -m "Your commit message" 
```

# 乱入
```
# Windows命令 验证文件MD5/SHA1/SHA256
certutil -hashfile file -?

certutil -hashfile file MD5
```