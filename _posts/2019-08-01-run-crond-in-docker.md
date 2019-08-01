---
layout: post
title:  "Docker环境中运行crond"
date:   2019-08-01 08:00:00 +0800
categories: ['docker']
---

`crond`是`Linux`下知名的计划任务执行工具, 但是大多数`Docker`镜像为了精简默认并不会安装.

这里记录下如何在`Docker`中使用`crond`.

# crond任务和启动脚本

首先把cron任务和启动脚本保存到app目录中, 如下所示:

`cron任务`
```bash
cat app/cron
# check status every 10 min
*/10 * * * * root /root/app/check.sh
```

`启动脚本`
```bash
cat app/start_app.sh
#!/usr/bin/env bash

# Start crond
/usr/sbin/crond
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start crond: $status"
  exit $status
fi

# Start your app
start_your_app
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start app: $status"
  exit $status
fi

# Naive check runs checks once a minute to see if either of the processes exited.
# This illustrates part of the heavy lifting you need to do if you want to run
# more than one service in a container. The container exits with an error
# if it detects that either of the processes has exited.
# Otherwise it loops forever, waking up every 60 seconds

while sleep 60; do
  ps aux |grep crond |grep -q -v grep
  PROCESS_1_STATUS=$?
  ps aux |grep your_app |grep -q -v grep
  PROCESS_2_STATUS=$?
  # If the greps above find anything, they exit with 0 status
  # If they are not both 0, then something is wrong
  if [ $PROCESS_1_STATUS -ne 0 -o $PROCESS_2_STATUS -ne 0 ]; then
    echo "One of the processes has already exited."
    exit 1
  fi
```


# 打包至镜像中

下面是使用`centos7`的`Dockerfile`示例:

`Dockerfile`
```bash
FROM centos:7
COPY . /app
RUN yum install cronie -y
RUN cp /app/cron /var/spool/cron/root
CMD ./app/start_app.sh
```

`打包`
```
docker build -t your_app .
```