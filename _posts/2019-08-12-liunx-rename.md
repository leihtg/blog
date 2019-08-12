---
layout: post
title: "Linux命令批量修改文件名"
categories: Linux
tags: Linux sed
author: leihtg
---

* content
{:toc}

对于在Linux中修改文件名的方式一般我们会用mv命令进行修改，但是mv命令是无法处理大量文件修改名称。

但是在处理大量文件的时候该如何进行批量修改呢？



###  **方法一：mv配合for循环方式进行修改**
```
[root@show day74]# for name in `ls *.html`;do echo $name ${name%.html}.jpg;done
00.html 00.jpg
01.html 01.jpg
02.html 02.jpg
03.html 03.jpg
04.html 04.jpg
05.html 05.jpg
06.html 06.jpg
07.html 07.jpg
08.html 08.jpg
09.html 09.jpg
10.html 10.jpg
[root@show day74]# for name in `ls *.html`;do mv $name ${name%.html}.jpg;done
[root@show day74]# ls 
00.jpg 01.jpg 02.jpg 03.jpg 04.jpg 05.jpg 06.jpg 07.jpg 08.jpg 09.jpg 10.jpg
```

###  **方法二：sed命令**

    ls *jpg|sed -r 's#(.*).jpg#mv &  \1.mp4#'|bash

### **方法三：rename命令**

rename命令用字符串替换的方式批量改变文件名。

格式：rename  原名  替换名  要改的文件 

原字符串：将文件名需要替换的字符串； 目标字符串：将文件名中含有的原字符替换成目标字符串； 文件：指定要改变文件名的文件列表。

    [root@cache01 test]# ls 
    01.txt  03.txt  05.txt  07.txt  09.txt
    02.txt  04.txt  06.txt  08.txt  10.txt
    [root@cache01 test]# rename txt jpg *
    [root@cache01 test]# ls 
    01.jpg  03.jpg  05.jpg  07.jpg  09.jpg
    02.jpg  04.jpg  06.jpg  08.jpg  10.jpg