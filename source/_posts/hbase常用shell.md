---
title: hbase的常用shell
date: 2016-01-20 10:43:10
tags: [hbase,markdown]
---


 ## 1.打开hbase并进入hbase shell console
<pre><code>
$HBASE_HOME/bin/start-hbase.sh
$HBASE_HOME/bin/hbase shell
hbase(main)> whoami
</code></pre>
<!-- more -->
 ## 2.表的管理
### 1）查看有哪些表
<pre><code>
hbase(main)> list
</code></pre>

### 2）创建表
{% blockquote   %}
* 语法：create <table>, {NAME => <family>, VERSIONS => <VERSIONS>}
{% endblockquote %}
<pre><code>
//例如：创建表t1，有两个family name：f1，f2，且版本数均为2
hbase(main)> create 't1',{NAME => 'f1', VERSIONS => 2},{NAME => 'f2', VERSIONS => 2}
</code></pre>

### 3）删除表
{% blockquote   %}
*  分两步：首先disable，然后drop
{% endblockquote %}
<pre><code>
//例如：删除表t1
hbase(main)> disable 't1'
hbase(main)> drop 't1'
</code></pre>

### 4）查看表的结构
{% blockquote %}
* 语法：describe <table>

{% endblockquote %}
<pre><code>
//例如：查看表t1的结构
hbase(main)> describe 't1'
</code></pre>

### 5）修改表结构
{% blockquote %}
* 修改表结构必须先disable
* 语法：alter 't1', {NAME => 'f1'}, {NAME => 'f2', METHOD => 'delete'}
{% endblockquote %}

<pre><code>
hbase(main)> disable 'test1'
hbase(main)> alter 'test1',{NAME=>'body',TTL=>'15552000'},{NAME=>'meta', TTL=>'15552000'}
hbase(main)> enable 'test1'
</code></pre>

## 3.表数据的增删改查
### 1）添加数据
{% blockquote %}
* 语法：put <table>,<rowkey>,<family:column>,<value>,<timestamp>
{% endblockquote %}
<pre><code>
//例如：给表t1的添加一行记录：rowkey是rowkey001，family name：f1，column name：col1，value：value01，timestamp：系统默认
hbase(main)> put 't1','rowkey001','f1:col1','value01'
//用法比较单一。
</code></pre>

### 2）查询数据
#### a）查询某行记录
{% blockquote %}
* 语法：get <table>,<rowkey>,[<family:column>,....]
{% endblockquote %}
<pre><code>
//例如：查询表t1，rowkey001中的f1下的col1的值
hbase(main)> get 't1','rowkey001', 'f1:col1'
//或者：
hbase(main)> get 't1','rowkey001', {COLUMN=>'f1:col1'}
//查询表t1，rowke002中的f1下的所有列值
hbase(main)> get 't1','rowkey001'
</code></pre>

#### b）扫描表
{% blockquote %}
* 语法：scan <table>, {COLUMNS => [ <family:column>,.... ], LIMIT => num}
* 另外，还可以添加STARTROW、TIMERANGE和FITLER等高级功能
{% endblockquote %}
<pre><code>
//例如：扫描表t1的前5条数据
</code></pre>
hbase(main)> scan 't1',{LIMIT=>5}
</code></pre>

#### c）查询表中的数据行数
{% blockquote %}
* 语法：count <table>, {INTERVAL => intervalNum, CACHE => cacheNum}
* INTERVAL设置多少行显示一次及对应的rowkey，默认1000；CACHE每次去取的缓存区大小，默认是10，调整该参数可提高查询速度
{% endblockquote %}
<pre><code>
//例如，查询表t1中的行数，每100条显示一次，缓存区为500
hbase(main)> count 't1', {INTERVAL => 100, CACHE => 500}
</code></pre>
### 3）删除数据

#### a )删除行中的某个列值
{% blockquote %}
* 语法：delete <table>, <rowkey>,  <family:column> , <timestamp>,必须指定列名
{% endblockquote %}
<pre><code>
//例如：删除表t1，rowkey001中的f1:col1的数据
hbase(main)> delete 't1','rowkey001','f1:col1'
//注：将删除改行f1:col1列所有版本的数据
</code></pre>

#### b )删除行
{% blockquote %}
* 语法：deleteall <table>, <rowkey>,  <family:column> , <timestamp>，可以不指定列名，删除整行数据
{% endblockquote %}
<pre><code>
//例如：删除表t1，rowk001的数据
hbase(main)> deleteall 't1','rowkey001'
</code></pre>

#### c）删除表中的所有数据
{% blockquote %}
* 语法： truncate <table>
* 其具体过程是：disable table -> drop table -> create table
{% endblockquote %}
<pre><code>
//例如：删除表t1的所有数据
hbase(main)> truncate 't1'
</code></pre>
