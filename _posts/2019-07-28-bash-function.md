---
layout: post
title: "Bash 常用函数"
date:   2019-07-28 22:00:00 +0800
categories: ['bash']
---

以下是工作中常用的`Bash`函数, 有的来源自网上, 有的自己写的, 记录于此, 避免重复造轮子.


# 输出颜色
```bash
# Color
RESTORE=$(echo -en '\033[0m')
RED=$(echo -en '\033[00;31m')
GREEN=$(echo -en '\033[00;32m')
YELLOW=$(echo -en '\033[00;33m')
BLUE=$(echo -en '\033[00;34m')
MAGENTA=$(echo -en '\033[00;35m')
PURPLE=$(echo -en '\033[00;35m')
CYAN=$(echo -en '\033[00;36m')
LIGHTGRAY=$(echo -en '\033[00;37m')
LRED=$(echo -en '\033[01;31m')
LGREEN=$(echo -en '\033[01;32m')
LYELLOW=$(echo -en '\033[01;33m')
LBLUE=$(echo -en '\033[01;34m')
LMAGENTA=$(echo -en '\033[01;35m')
LPURPLE=$(echo -en '\033[01;35m')
LCYAN=$(echo -en '\033[01;36m')
WHITE=$(echo -en '\033[01;37m')
RESTORE=$(echo -en '\033[0m')

# usage
echo "${BLUE} It's blue.${RESTORE}"
```

# 动态变量名
```bash
args="a b c"
for arg in $args
do
   # 赋值 Assignment
    declare $arg=0
    let $arg++
done

for arg in $args
do
   # 提取 Extract
   echo $arg: ${!arg}
done
```

# 日志
```bash
# Define log file
logfile=/tmp/log

# Function: Setup logfile and redirect stdout/stderr.
log_setup() {
     # Check if logfile exists and is writable.
     ( [ -e "$logfile" ] || touch "$logfile" ) && [ ! -w "$logfile" ] && echo "ERROR: Cannot write to $logfile. Check permissions or sudo access." && exit 1

     tmplog=$(tail -n $logfile_max_lines $logfile 2>/dev/null) && echo "${tmplog}" > $logfile
     exec >  >(tee -a $logfile)
     exec 2>&1
}

# Function: Log an event.
log() {
     echo "[$(date +"%Y-%m-%d"+"%T")]: $*"
}
```

# 验证json via [jq](https://stedolan.github.io/jq/)
```bash
# validate json via jq
function check_json()
{
JSON_FILE=$1
if [ ! -f $JSON_FILE ]; then
    printf "${JSON_FILE} is not exits. Please check. \n"
    exit 2
elif ! jq . $JSON_FILE >/dev/null 2>&1; then
    printf "jq cannot parse ${JSON_FILE}, Please check. \n"
    exit 2
fi
}

# usage
check_json /tmp/test.json
```

# 参考:

- [https://stackoverflow.com/questions/16553089/bash-dynamic-variable-names](https://stackoverflow.com/questions/16553089/bash-dynamic-variable-names)
