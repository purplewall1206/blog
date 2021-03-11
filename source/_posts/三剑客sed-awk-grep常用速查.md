---
title: 三剑客sed/awk/grep常用速查
date: 2020-11-16 14:22:23
tags: [bash, linux, tool]
categories:
- linux
- bash
---


sed和awk的区别，都是按行读入，但是awk会拆分行里面的元素，sed则直接用正则做匹配，基本上sed可以做到的awk都可以。

# awk
`awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' netstat.txt` awk 使用print和printf打印每行中按列分布的元素。

`awk '$3==0 && $6=="ESTABLISHED"' netstat.txt` 过滤记录

`awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%02s %s %-20s %-20s %s\n",NR, FNR, $4,$5,$6}' netstat.txt` 其中NR，FNR为内建变量
<!-- more -->
- $0	当前记录（这个变量中存放着整个行的内容）
- $1~$n	当前记录的第n个字段，字段间由FS分隔
- FS	输入字段分隔符 默认是空格或Tab
- NF	当前记录中的字段个数，就是有多少列
- NR	已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。
- FNR	当前记录数，与NR不同的是，这个值会是各个文件自己的行号
- RS	输入的记录分隔符， 默认为换行符
- OFS	输出字段分隔符， 默认也是空格
- ORS	输出的记录分隔符，默认为换行符
- FILENAME	当前输入文件的名字


`awk  'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd` 指定分隔符

`awk '$6 ~ /E/ || NR==1 {print NR, $4,$5,$6}' netstat.txt` ~表示开始，//表示匹配，这行命令表示匹配第六个元素开始包含E的行和第一行，然后输出行数和第456个元素。

`awk 'NR!=1{print $4,$5 > $6}' netstat.txt` 不处理第一行，按照第六列元素分类拆分成若干个文件，文件的内容是第45列元素。

## awk 脚本
```
#!/bin/awk -f
# 运行前
BEGIN {
	math = 0
	english = 0
	computer = 0

	printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
	printf "---------------------------------------------\n"
}
# 运行中
{
	math += $3
	english += $4
	computer += $5
	printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
# 运行之后
END {
	printf "---------------------------------------------\n"
	printf "  TOTAL:%10d %8d %8d \n", math, english, computer
	printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}

```

`awk -v val=$x '{print $1, $2, $3, $4+val, $5+ENVIRON["y"]}' OFS="\t" score.txt` 其中x，y为环境变量，为了使用环境变量需要 -v

`echo $PATH| awk 'BEGIN{RS=":"}{print NR,$0}'` 把单行拆分成多行

# sed
`sed [-hnV][-e<script>][-f<script文件>][文本文件]`

- e\<script\>或--expression=\<script\> 以选项中指定的script来处理输入的文本文件。
- f\<script文件\>或--file=\<script文件\> 以选项中指定的script文件来处理输入的文本文件。
- h或--help 显示帮助。
- n或--quiet或--silent 仅显示script处理后的结果。
- V或--version 显示版本信息。

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！


`sed "s/my/ppw 's/g" pets.txt` 把pets.txt文件的my改成ppw's，并不修改文件。/g 表示一行上的替换所有的匹配

加入-i `sed -i "s/my/ppw 's/g" pets.txt`可以直接修改文件

`sed 's/^/---/g' pets.txt` 在开头加上---

`sed 's/$/---/g' pets.txt` 在结尾加上---

同理类似的有：
- \\< 表示词首。 如：\<abc 表示以 abc 为首的詞。
- \\> 表示词尾。 如：abc\> 表示以 abc 結尾的詞。
- . 表示任何单个字符。
- \* 表示某个字符出现了0次或多次。
- [ ] 字符集合。 如：[abc] 表示匹配a或b或c，还有 [a-zA-Z] 表示匹配所有的26个字符。如果其中有^表示反，如 [^a] 表示非a的字符

`sed 's/<[^>]*>//g' html.txt` 取消html文件中的<>tags,不能使用`'s/<.*>'//g`，因为会贪婪匹配掉第一个《和最后一个》之间所有的内容，现在给出的示例是匹配除了>意外的字符。

`sed '3s/my/your/g' pets.txt` 和 `sed '3,6s/my/your/g' pets.txt`，分别表示匹配第三行和第三到六行。

`sed 's/s/S/1' pets.txt` 匹配每一行的第一个s

`sed '1,3s/my/your/g; 3,$s/This/That/g' pets.txt` 匹配多个

`sed 's/my/[&]/g' pets.txt` 使用&来当做被匹配的变量，然后可以在基本左右加点东西

`sed 's/This is my \([^,]*\),.*is \(.*\)/\1:\2/g' my.txt` 圆括号括起来的正则表达式所匹配的字符串会可以当成变量来使用，sed中使用的是\1,\2…

`sed 'N;s/my/your/' pets.txt` 原文本中的偶数行纳入奇数行匹配，而s只匹配并替换一次，所以最后只有奇数行被修改。 N命令把下一行的内容纳入当成缓冲区做匹配。

`sed "1 i This is my monkey, my monkey's name is wukong" my.txt` 和 `sed "$ a This is my monkey, my monkey's name is wukong" my.txt`  命令i和a分别表示insert 和 append，添加行

`sed "/my/a ----" my.txt` 在每行结尾都添加 ----

`sed "2 c This is my monkey, my monkey's name is wukong" my.txt` 和 `sed "/fish/c This is my monkey, my monkey's name is wukong" my.txt` 命令c表示替换，这两个命令分别表示替换第二行和有’fish’存在的那一行。

`sed '/fish/d' my.txt` 和 `sed '2d' my.txt` 和 `sed '2,$d' my.txt` 命令d表示删除匹配行，这三个命令分别表示删除有fish的一行，删除第二行和删除第二到最后的所有行。

`sed -n '/dog/,/fish/p' my.txt` 命令-n表示只显示处理后的结果，命令p表示打印，图中命令表示打印有dog的一行和有fish的一行。


# grep

`grep -i -c '.*ret.*' linux-exec.s` 统计内核二级制文件中ret命令的个数，不区分大小写，直接输出匹配行数。