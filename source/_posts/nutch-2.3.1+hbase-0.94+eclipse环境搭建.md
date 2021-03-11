---
title: nutch-2.3.1+hbase-0.94+eclipse 环境搭建教程
date: 2016-03-08 23:52:10
tags: [nutch, hbase]
---

## 1.开发环境及软件版本
* centos 6.7
* apache-nutch-2.3.1
* hbase-0.94
* eclipse javaEE
* jdk8

## 2.安装并配置jdk
<!-- more -->
### 1) 在Oracle网站wget下载jdk8
{% codeblock jdk8_installing %}
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u73-b02/jdk-8u73-linux-x64.tar.gz
{% endcodeblock %}

由于Oracle需要点击accept licence的才能下载，所以必须使用下面的命令，才可以直接下载。
否则下载的jdk文件将无法解压缩。

### 2) 配置JAVA_HOME
{% codeblock jdk8_installing %}
mkdir/usr/java/
tar -zxf jdk-8u73-linux-x64.tar.gz //不适用-v指令以加快解压缩速度
mv jdk-8u73-linux-x64/ /usr/java/
vi /etc/profile  //打开vi文本编辑器编辑环境变量
{% endcodeblock %}

按i开始编辑，在文件的末尾输入
{% codeblock jdk8_installing %}
  #set JDK environment
  
JAVA_HOME=/usr/java/jdk1.8.0_45
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOMECLASS_PATH PATH
{% endcodeblock %}
先按Esc退出编辑模式，然后输入 :wq 保存退出
{% codeblock jdk8_installing %}
source /etc/profile  //使环境生效
echo $JAVA_HOME //检验是否安装成功
/usr/java/jdk1.8.0_45  //环境生效！！！
{% endcodeblock %}

## 3.安装并配置hbase-0.94

### 1) 在hbase页面下载hbase-0.94解压缩并重命名
{% codeblock hbase_installing %}
wget http://www-eu.apache.org/dist/hbase/hbase-0.94.27/
tar -zxf hbase-0.94.27/   //减少输出，加快速度
mv hbase-0.94.27/ hbase/  //重命名，简短文件名称
mv hbase/ /usr/           //将hbase/文件夹移动到/usr/路径下
cd /usr/                  //进入/usr目录
chmod -R 777 hbase/       //给hbase目录分配权限

cd hbase/
vi conf/hbase-site.xml    //配置hbase-site.xml文件
<configuration>
<property>
    <name>hbase.rootdir</name>
    <value>file:////usr/hbase/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/usr/hbase/zookeeper</value>
  </property>
</configuration>
{% endcodeblock %}
之后的nutch爬出的数据将存储到rootdir文件夹中，所以必须配置zookeeper的同理。
测试是否安装成功
{% codeblock hbase_installing %}
./bin/start-hbase.sh //打开hbase进程
./bin/hbase shell //可以进入hbase命令行模式
//检验成功可以运行后暂时关闭hbase进程
./stop-hbase.sh
{% endcodeblock %}
至此hbase安装成功，hbase-0.94可以直接在本地运行，不需要hadoop集群（个人暂时是这么理解的欢迎指正）

## 4.安装并配置apache-nutch-2.3.1

### 1)wget下载apache-nutch-2.3.1并解压缩
{% codeblock nutch-2_installing %}
wget http://www-eu.apache.org/dist/nutch/2.3.1/apache-nutch-2.3.1-src.tar.gz
//速度不够的话去官网看他给你推荐的链接
tar -zxf apache-nutch-2.3.1-src.tar.gz
mv apache-nutch-2.3.1/ nutch-2/
mv nutch-2/ /usr/ //移动文件
cd /usr/
chmod -R 777 nutch-2
{% endcodeblock %}

* 下面开始重头戏，配置nutch文件
#### (1)conf目录下的nutch-site.xml文件(我的配置)
{% codeblock nutch-2_installing %}
<configuration>
<property>
<name>storage.data.store.class</name>
<value>org.apache.gora.hbase.store.HBaseStore</value>
<description>Default class for storing data</description>
</property>

<property>
  <name>plugin.includes</name>
  <value>protocol-httpclient|urlfilter-regex|index-(basic|more)|query-(basic|site|url|lang)|indexer-solr|nutch-extensionpoints|protocol-httpclient|urlfilter-regex|parse-(text|html|msexcel|msword|mspowerpoint|pdf)|summary-basic|scoring-opic|urlnormalizer-(pass|regex|basic)protocol-http|urlfilter-regex|parse-(html|tika|metatags)|index-(basic|anchor|more|metadata)</value>
</property>


 <property>
    <name>http.agent.name</name>
    <value>Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.117 Safari</value>
</property>
<property>
    <name>http.agent.version</name>
    <value>537.36</value>
</property>


<property>
   <name>http.robots.agents</name>
   <value>Test-Crawler</value>
 </property>
 
 <property>
 
 <property>
  <name>plugin.folders</name>
  <value>./build/plugins</value>
  <description>Directories where nutch plugins are located.  Each
  element may be a relative or absolute path.  If absolute, it is used
  as is.  If relative, it is searched for on the classpath.</description>
</property>
 </property>
 
 <property>
  <name>plugin.includes</name>
 <value>protocol-http|urlfilter-regex|parse-(html|tika)|index-(basic|anchor)|urlnormalizer-(pass|regex|basic)|scoring-opic|index-anchor|index-more|languageidentifier|subcollection|feed|creativecommons|tld</value>
 
</property>


</configuration>
{% endcodeblock %}
* 留心plugin.folders，配置不对花式报错

#### (2)修改conf/gora.properties
加入<pre><code>gora.datastore.default=org.apache.gora.hbase.store.HBaseStore</code></pre>
之后注释掉所有其他内容

#### (3)修改ivy/ivy.xml文件
取消注释
<pre><code><dependency org="org.apache.gora" name="gora-hbase" rev="0.5" conf="*->default" /></code></pre>
表示使用hbase数据库

* 另外regex-urlfliter.txt网页内容过滤等可以暂时不进行配置

* 返回nutch-2根目录
{% codeblock nutch-2_installing %}
yum -y install ant //yum直接安装ant，对nutch进行搭建
ant eclipse
ant runtime //命令行操作方式构建，进入runtime/local/bin/文件夹中有nutch和crawl感兴趣的关注文章结尾处推荐链接
{% endcodeblock %}

{% blockquote 重要的事情说三遍  %}
maven资源被墙了，建议翻墙使用
maven资源被墙了，建议翻墙使用
maven资源被墙了，建议翻墙使用
{% endblockquote %}
安装完成之后就可以导入eclipse中了
{% blockquote  %}
* 革命尚未成功，同志仍需努力。
* 行百里者半九十
* 最后一步不认真的话可能会超级麻烦的！！（认真脸）
{% endblockquote %}
* 别忘了最后一步
* 在nutch根目录中新建文件夹urls/，在文件夹中新建seed.txt文件 http://www.zi-c.wang 保存退出

## 5.导入eclipse中（想导入idea也可以这样玩，貌似）

### 1)创建工程java project
进入eclipse->file->new->project->java project
直接修改项目路径（/usr/nutch-2）
点击next，在build path配置中order&export选项卡，选中nutch-2/conf文件夹，单击top

### 2)配置run configuration
点击run configurations，进入页面。
右键java application，新建application命名为injectJob
project：nutch-2
main class：org.apache.nutch.crawl.InjectorJob
进入run configurations的arguments标签中，在Program Arguments中添加
urls/seed.txt -crawlId search
点击apply

同理
application：generateJob
main class：org.apache.nutch.crawl.GeneratorJob
arguments:-topN 10 -crawlId search

application：fetcheJob
main class：org.apache.nutch.fetcher.FetcherJob
arguments:-all -crawlId search -thread 10

application：parseJob
main class：org.apache.nutch.parse.ParserJob
arguments:-all -crawlId search

application：updatedbJob
main class：org.apache.nutch.crawl.DbUpdaterJob
arguments:-all -crawlId search

### 3)跑起来吧！！奔放的eclipse

### 4)查看爬取得数据
{% codeblock nutch-2_installing %}
cd /usr/hbase/
./bin/hbase shell
list //hbase指令请参考我的上一篇文章 hbase常用指令
scan 'search_webpage' //灰常灰常壮观的数据闪过去了=_+
exit
{% endcodeblock %}
成功！！！

## 6.感想
  搭建这套环境对我来说是很大的一个修行！！
  前前后后使用的时间可能超过50h了，真是笨死了。
  耗时这么长的原因：主要是我太粗心了，很多应该完成的配置并没有实现成功，浪费了很多时间
  另一方面：不会看log
  现在看起来非常可笑的事情就是我的环境一旦搭建失败了首先看的是文档而不是日志，真是笨死了
  同时我也终于意识到Android application中为什么强调写好log。
  
  搭建这套环境确实提升了个人能力一大截，超级期待下一步的搜索引擎实践！！
  加油奔放的少年！！！
  
## 7.推荐链接

1.[Apache Nutch Wiki](https://wiki.apache.org/nutch/#Nutch_2.X_tutorial.28s.29)
2.[【Nutch2.2.1基础教程之2.1】集成Nutch/Hbase/Solr构建搜索引擎之一：安装及运行【单机环境】](http://blog.csdn.net/jediael_lu/article/details/37329731)
3.[在Eclipse中运行Nutch2.3](http://blog.csdn.net/jediael_lu/article/details/43232449)
4.[about云社区](http://www.aboutyun.com/forum.php)
5.[hbase wiki](https://hbase.apache.org/book.html)
6. 杨尚川老师的nutch系列视频（虽然有点旧，但是给你仔细分析每个过程）
7.[Linux 使用wget 命令下载JDK的方法](http://www.oschina.net/code/snippet_875267_44726)
8.[神圣网站stackoverflow，救过我无数次](http://stackoverflow.com/)

