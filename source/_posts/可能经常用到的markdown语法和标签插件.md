---
title: 可能经常用到的markdown语法和标签插件
date: 2016-01-20 10:43:10
tags: markdown
---
## 摘要
* 本文没有系统介绍markdown语法和标签插件，而是仅仅列举的可能经常使用的markdown语法和标签插件样式

l\><li\>1</li\><li\>2</li\></ol\>" 都可以表示分点
+ First
+ Second
+ Third
+ false
<!-- more -->
## 关于换行
* 单一段落( <p\>) 用一个空白行
* 连续两个空格 会变成一个 <br\>
* 连续3个符号，然后是空行，表示 hr横线


## 关于插入代码
 可以是使用<pre\><code\>code</code\></pre\>形式插入代码，不过比较简陋
<pre><code>cout << "Hello World!!" << endl; </code></pre>
 另外的一种插入代码的方法
 
 \{\% codeblock \[title\] \[lang:language\] \[url\] \[link text\] \%\}
code snippet
\{\% endcodeblock \%\}
{% codeblock HelloWorld lang:c++  %}
cout << "Hello World!!" << endl;
{% endcodeblock %}


## 关于插入链接
 直接写 \[锚文本\]\(url "可选的title"\)  
 [www.zi-c.wang](http://www.zi-c.wang "紫城君的Blog")

 
## 关于插入图片url
\!\[alt_text\]\(url "可选的title"\)
![LiveLongAndProsper](http://i8.tietuku.com/78a8923ba6584002.jpg "LiveLongAndProsper")


## 特殊符号
直接在文本前面加上转义符 \ 就可以

## 插入Youtube视频
\{\% youtube video_id \%\}
{% youtube https://www.youtube.com/watch?v=YG77IYQquB0 %}


