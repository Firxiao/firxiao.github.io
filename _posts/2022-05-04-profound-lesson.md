---
layout: post
title:  "教训一则"
date:   2022-05-04
categories: ['lesson']
---

2022年5月04日 天气晴

因为疫情, 上海已经封了一个月有余, 在家也工作了一个多月.

五一假期的前三天, 重温了*Flask*开发, 一切平和.

第四天是个神奇的一天.

早上学习了*Spluck*, 下午便急于制作Dashboard, 便拿公司生产环境操刀.

本该是一个平静的一天, 却被一个不经意的低级错误给扰乱了.

_直接更改了生产环境的*Nginx*配置文件._

现在回忆, 想不明白当时为什么会直接操作生产环境. 

思前想去，感觉可能和下面几点有关

- 在家工作时间久了 工作和生活已经分不清了 
- 对自己的操作太自信了
- 强迫症犯了 看到有不顺眼的地方 就直接上了
- 最近工作压力太大 急于求成

祸以酿成, 好在后果不是很严重, 但足够令我深思.
特此记录教训一侧, 警示自己.