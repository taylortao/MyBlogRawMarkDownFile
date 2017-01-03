---
title: SCI期刊信息整理爬虫
date: 2012-03-23 11:59:00
categories:
 - IT Technology
tags:
 - Spider
---
### 1、需求

按ISSN号进行搜索，整理一个特定期刊列表里的所有期刊的年文章、投稿难易和一审周期等信息。使用PHP脚本编写。
网址：http://www.medsci.cn/sci/

<!-- more -->

（1）按ISSN号搜索
![search by ISSN number](searchbyissn.gif)

（2）得到结果，以及需要提取的部分
![search result](searchresult.gif)

### 2、分析问题

** 三个步骤： **
（1）使用网络蜘蛛从 http://www.medsci.cn/sci/ 网站抓取信息，模拟输入要搜索的ISSN号，提交表单，获得查询结果的页面。
（2）使用正则表达式搜索结果页面，分析页面代码结构，把需要的信息提取出来。
（3）输出到文件，因为此输出文件无需考虑单元格的排版，为了简便，直接输出为XML的逗号表达式格式。这种格式可以使用Excel直接打开。

** 选用的类库： **
> *（1）网络蜘蛛Snoopy：
Snoopy是一个php类，用来模拟浏览器的功能，可以获取网页内容，发送表单。

> *（2）Excel_XML
把array中的数据输出为XML格式的电子表格。

其格式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet" xmlns:x="urn:schemas-microsoft-com:office:excel"   
  
xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet" xmlns:html="http://www.w3.org/TR/REC-html40">  
<Worksheet ss:Name="sci">  
<Table>  
<Row>  
<Cell><Data ss:Type="String">ISSN</Data></Cell>  
<Cell><Data ss:Type="String">年文章</Data></Cell>  
<Cell><Data ss:Type="String">投稿难易</Data></Cell>  
<Cell><Data ss:Type="String">一审周期</Data></Cell>  
</Row>  
</Table>  
</Worksheet>  
</Workbook>  
```

### 3、关键问题

正则表达式匹配测试网站：http://www.rubular.com/
匹配所有汉字：
utf-8编码：

```php
preg_match("/^[\x{4e00}-\x{9fa5}A-Za-z0-9_]+$/u",$str)  
```

gbk编码：

```php
preg_match("/^[".chr(0xa1)."-".chr(0xff)."A-Za-z0-9_]+$/",$str)  
```

匹配特定汉字（比如候鸟）：
utf-8编码：

```php
<?php  
header("Content-Type:text/html; charset=utf-8");  
$gb = "可你跟随那南归的候鸟飞的那么远";  
$utf8 = iconv('GB2312', 'UTF-8', $gb);  
  
preg_match("/\x{5019}\x{9E1F}/u",$utf8, $match1);  
echo "<pre>";  
print_r($match1);  
echo "</pre>";  
?>  
```

gbk编码：
```php
<?php  
//header("Content-Type:text/html; charset=utf-8");  
$gb = "可你跟随那南归的候鸟飞的那么远";  
preg_match("/候鸟/",$gb, $match2);  
echo "<pre>";  
print_r($match2);  
echo "</pre>";  
?>  
```

### 4、使用步骤

（1）输入格式为：
1751-8628,0308-5961,1472-3581,……
小技巧：可以通过拷贝Excel的ISSN号序列放到EditPlus中，Ctrl+H快捷键选择正则表达式复选框，把所有的\n替换为，即可。

（2）运行myspider.php脚本
![input of myspider.php](inputofmyspider.gif)

（3）得到结果表格
![output table](output.gif)

### 5、程序源代码下载

期刊爬虫下载地址: http://download.csdn.net/detail/taylor_tao/4166037