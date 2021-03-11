---
title: Nutch2.3.1+hbase-0.98-hadoop+solr5.2.1的自动部署脚本
date: 2016-10-18 17:50:10
tags: markdown
---



在阳总的帮助下，终于搞出了这个自动部署脚本，感谢阳总让我第一次看到真人写的bash脚本，很涨姿势。

* hbase-0.98-hadoop 的选择原因是[http://wiki.apache.org/nutch/Nutch2Tutorial ](nutch turorial)上这么推荐的
* solr5.x 的玩法似乎和solr4.9 差别不小，之后应该会单独写一篇分析的
* 测试环境是 centos6环境，使用ubuntu或者其他发行版的兄弟记得改下 那堆yum，rpm
<!-- more -->
<pre><code> #!/bin/bash </code></pre>
这个写到一开头表示这是<strong>.sh</strong>那一类文件，剩下的就是用重定向之类的linux下的特殊写法追加写入文本的

* 脚本中的所有文件都将被部署到 <strong>/usr</strong> 目录下，如果改目录需要连着配置文件一起改，权限不够加<strong>sudo</strong>
* nutch crawl solr hbase 都写入环境变量，可以在任何目录下运行。
* nutch 执行的是 /usr/nutch-2/runtime/local/bin/nutch 目录的文件，而不是deploy中
* 之后集成hadoop再修改配置文件。

以下是自动部署脚本，需要复制下来制作成 <strong>.sh</strong> 文件，使用 <pre><code> sudo bash xxx.sh </code></pre> 执行

{% codeblock 自动部署脚本 lang:bash  %}
#!/bin/bash

rpm -Uvh https://centos6.iuscommunity.org/ius-release.rpm
yum install epel-release -y
yum install git wget screen vim -y
yum update -y

iptables -F
iptables-save
service iptables save
ip6tables -F
ip6tables-save
service ip6tables save

cd /opt
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-x64.rpm"
rpm -ivh jdk-8u102-linux-x64.rpm
rm -rf jdk-8u102-linux-x64.rpm

wget http://archive.apache.org/dist/hbase/hbase-0.98.8/hbase-0.98.8-hadoop2-bin.tar.gz
tar xzf hbase-0.98.8-hadoop2-bin.tar.gz
rm -rf hbase-0.98.8-hadoop2-bin.tar.gz
mv hbase-0.98.8-hadoop2 /usr/hbase

wget http://www-eu.apache.org/dist/nutch/2.3.1/apache-nutch-2.3.1-src.tar.gz
tar xzf apache-nutch-2.3.1-src.tar.gz
rm -rf apache-nutch-2.3.1-src.tar.gz
mv apache-nutch-2.3.1 /usr/nutch-2
chmod -R 777 /usr/nutch-2

wget http://www-us.apache.org/dist/ant/binaries/apache-ant-1.9.7-bin.tar.gz
tar xzf apache-ant-1.9.7-bin.tar.gz
rm -rf apache-ant-1.9.7-bin.tar.gz
mv apache-ant-1.9.7 /usr/ant

wget http://archive.apache.org/dist/lucene/solr/5.2.1/solr-5.2.1.tgz
tar xzf solr-5.2.1.tgz
rm -rf solr-5.2.1.tgz
mv solr-5.2.1 /usr/solr

rm -rf ~/envtemp
cat > ~/envtemp<<'EOF'
#set JDK environment
JAVA_HOME=/usr/java/jdk1.8.0_102
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
#set ANT environment
ANT_HOME=/usr/ant
#set HBASE environment
HBASE_HOME=/usr/hbase
#set LOCAL_NUTCH environment
NUTCH_HOME=/usr/nutch-2/runtime/local
#set SOLR environment
SOLR_HOME=/usr/solr
#set HADOOP environment
HADOOP_HOME=/usr/hadoop
HADOOP_INSTALL=$HADOOP_HOME
HADOOP_MAPRED_HOME=$HADOOP_HOME
HADOOP_COMMON_HOME=$HADOOP_HOME
HADOOP_HDFS_HOME=$HADOOP_HOME
YARN_HOME=$HADOOP_HOME
HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib:$HADOOP_COMMON_LIB_NATIVE_DIR"
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$ANT_HOME/bin:$HBASE_HOME/bin:$NUTCH_HOME/bin:$SOLR_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export JAVA_HOME JRE_HOMECLASS_PATH ANT_HOME HBASE_HOME NUTCH_HOME SOLR_HOME HADOOP_HOME HADOOP_INSTALL HADOOP_MAPRED_HOME HADOOP_COMMON_HOME HADOOP_HDFS_HOME YARN_HOME HADOOP_COMMON_LIB_NATIVE_DIR HADOOP_OPTS PATH
EOF

rm -rf ~/profile
cp /etc/profile ~/profile
cat ~/profile ~/envtemp >/etc/profile

rm -rf ~/envtemp ~/profile

source /etc/profile

cd ~
wget http://www.senra.me/nutch-solr/nutch-site.xml
wget http://www.senra.me/nutch-solr/gora.properties
wget http://www.senra.me/nutch-solr/ivy.xml
wget http://www.senra.me/nutch-solr/hbase-site.xml
wget http://www.senra.me/nutch-solr/solr.xml
\cp -a ~/nutch-site.xml /usr/nutch-2/conf/
\cp -a ~/gora.properties /usr/nutch-2/conf/
\cp -a ~/ivy.xml /usr/nutch-2/ivy/
\cp -a ~/hbase-site.xml /usr/hbase/conf/
\cp -a ~/solr.xml /usr/solr/
rm -rf ~/nutch-site.xml ~/gora.properties ~/ivy.xml ~/hbase-site.xml ~/solr.xml

cd /usr/nutch-2
ant runtime

/usr/hbase/bin/start-hbase.sh
/usr/solr/bin/solr start
/usr/solr/bin/solr create_core -c demo

{% endcodeblock %}


* 20号之后的大创答辩加油！争取国家级！