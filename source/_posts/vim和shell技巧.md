---
title:      vim和shell技巧
author:     liuyan
catalog:    true
tags:
  - linux 
date:       2018-01-03 19:44:05
urlname:
categories: linux
---

## linux

### shell快捷键
```shell
ctrl+a:		移到行首
ctrl+e:		移到行尾
ctrl+xx:	行首到当前光标替换
ctrl+w:		删除光标前一个单词相当于vim里db
ctrl+h:		删除光标前一个字符（相当于backspace）
ctrl+d: 	删除光标后一个字符（相当于delete）
ctrl+u: 	删除光标左边所有
ctrl+k: 	删除光标右边所有
ctrl+l: 	清屏
```

<!-- more -->

### awk获取文本某列
```sh
awk '{print $0}' filename    	#完整的文件内容
awk '{print $1}' filename    	#取第一列
awk '{print $1,$2}' filename 	#取前两列
awk '{print $NF}' filename  	#取最后一列
awk '{print $NF-1}' filename  	#取倒数第二列
```

### sed修改文本内容
```sh
sed 's/^/HEAD&/g' filename		    #在每行的头添加字符"HEAD"
sed 's/$/&TAIL/g' filename		    #在每行的尾添加字符“TAIL”
sed  's/a/b/g'  filename		    #将文本中的"a"换成"b"
sed -e 'nc/just do' filename	    #把第n行替换成just do
sed -e '1,10c/I can do' filename    #把1到10行替换成一行：I can do

#"^"代表行首，"$"代表行尾，字符g代表每行出现的字符全部替换
#如果想在原文件上更改，添加选项-i
```

### grep搜索文件内容
#### 格式
```shell
grep [options] PATTERN [FILE...]
[options]主要参数：
－c：只输出匹配行的计数
－I：不区分大小写
－h：查询多文件时不显示文件名
－l：查询多文件时只输出包含匹配字符的文件名
－n：显示匹配行及行号
－v：显示不包含匹配文本的所有行
```

#### 实例
```shell
ls -l | grep '^a'		    #只显示以a开头的行
ls -l | grep '$a'		    #只显示以a结尾的行
grep 'test' d*			    #显示所有以d开头的文件中包含test的行
grep 'test' aa bb cc        #显示在aa，bb，cc文件中匹配test的行
grep '[a-z]\{5\}' aa    	#显示aa中所有有5个连续小写字符的行
grep -i '^s' a			    #不区分大小以s/S开头的行
grep ':[0-9]:' a		    #两个冒号中间一个数字的行
```

### 环境变量
```shell
echo $PATH（查看）
export PATH=/usr/local/bin:$PATH（临时添加）
将export PATH="/usr/local/bin:$PATH"添加到/etc/profile或者~/.bashrc，然后source一下（永久添加）
```



## vim

### vim快捷键
```shell
e: 前移一个单词
b: 后移一个单词
hjkl 
gg: 到文件头部
G: 到文件尾部
ctrl+d: 下翻半屏
ctrl+u: 上翻半屏
d$ or D: 删除（剪切）当前位置到行尾的内容
d0: 删除（剪切）当前位置到行首的内容
%: 不仅能移动到匹配的(),{}或[]上，而且能在#if，#else， #endif之间跳跃
*: 向下搜索光标所在词
```

### 多行缩进
```sh
1 //在这里按下'v'进入选择模式
2 ...
3 //光标移动到这里，再按一次大于号'>'缩进一次，按'6>'缩进六次，按'<'回缩。
```

### 多行注释
```
1 //在这里按下'v'进入选择模式
2 ...
3 //光标移动到这里，按'ctrl+v'（win下面'ctrl+q'）进入列模式
4 //按大写'I'进入插入模式，输入注释符'#'或者是'//'，然后立刻按两下ESC。
```

### 多行取消注释
```
1 //在这里按下'ctrl+v'进入块选择模式
2 ...
3 //光标移动到这里，选中'//'或者'##'，之后按d即可。(删除)
```

## chrome快捷键

- 跳转到下一个打开的标签页 ⌘ +Option+向右箭头键
- 跳转到特定标签页 ⌘ +1 到 ⌘ +8
