---
title: NJU研究生通知RSS
date: 2020-11-19 14:39:45
tags: [python, rss, github, NJU]
categories:
- python 
- rss
- github
---

# NJU-notifeed

Nanjing University notification feed

https://github.com/purplewall1206/NJU-notifeed

**直接订阅：**    
http://idcvz.zi-c.wang:8000/rss

<!-- more -->


主要收集以下两个网站的最新通知，生成统一的feed提供给本地部署的RSS阅读器（i.e. Thunderbird） 

https://grawww.nju.edu.cn

http://pyb.nju.edu.cn/



# 设计思路

使用requests和beautifulsoup4制作爬虫，获取通知页面的信息条目news

使用PyRSS2Gen生成feed条目，并写到feed.xml文件中（这里有个问题，lib好像只提供文件写入函数了，暂时没找到其他的output方式）

使用Flask构建RESTful API，访问 http://url/rss 获取feed

使用多线程其中一个线程运行爬虫，主线程（flask单线程）维护API

文件结构：
- news.py news struct，存储消息的属性
- crawlerPYB/GRA.py 两个网站的爬虫策略
- \_\_init\_\_.py 主函数，通过多线程的方式每小时更新一次feed。
- logging.txt 记录调用日志
- requirement.txt 安装的python lib，`sudo pip install -r requirement.txt`

ubuntu 运行面对的问题 `sudo apt-get install python3-html5lib`

# 启动问题

加入description信息获取之后，服务器往往无法立即反应，需要等到logging中显示出爬虫已经完全获取信息的日志之后，才能通过网站访问。需要处理一下这个问题，应该是global被锁住，服务器比爬虫先启动导致的。