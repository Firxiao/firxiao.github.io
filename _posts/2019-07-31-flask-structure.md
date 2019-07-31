---
layout: post
title:  "Flask 结构"
date:   2019-07-31 08:00:00 +0800
categories: ['flask']
---

很早就开始接触`Python` `web` 框架. 也算自学了`Django`, 但终归没有实际应用. 
去年开始接触`Flask`, 根据前人的项目, 总算是照猫画虎做出个能看的`web`应用.

目录结构和官方的`Flask`有点出入, 可以配置多个环境和app, 记录在此方便以后使用.


## 目录结构

```bash
├── README.md
├── app
│   ├── __init__.py
│   ├── bar
│   │   ├── __init__.py
│   │   └── views.py
│   ├── foo
│   │   ├── __init__.py
│   │   └── views.py
│   ├── static
│   └── templates
├── config.py
├── manage.py
└── requirements.txt
```



# 代码
- [https://github.com/Firxiao/flask-demo](https://github.com/Firxiao/flask-demo)