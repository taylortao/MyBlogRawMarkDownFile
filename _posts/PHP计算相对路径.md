---
title: PHP计算相对路径
date: 2012-06-12 17:55:00
categories:
 - IT Technology
tags:
 - PHP
toc: false
---

PHP中计算相对路径的代码

<!-- more -->

```php
function transfer($apa, $apb, &$rpa, &$rpb)  
{  
    $apa = str_replace('\\', '/', $apa); // 把\预处理为/  
    $apb = str_replace('\\', '/', $apb);  
    // 对于某一个串为空串或者只含有一个/或者\  
    if($apa == '' || $apb == '' || $apa == '/' ||   
        $apb == '/' || $apa == '\\' || $apb == '\\')   
        return false;  
    $apa_arr = explode('/', trim($apa, '/'));  
    $apb_arr = explode('/', trim($apb, '/'));  
    $al = count($apa_arr);  
    $bl = count($apb_arr);  
    $i=0;  
    while(true)  
    {  
        if($i>=$al || $i>=$bl)  
        {  
            break;  
        }  
        if($apa_arr[$i] != $apb_arr[$i])  
        {  
            break;    
        }  
        $i++;  
    }  
    $rpa = '';  
    $rpb = '';  
    for($j=$i; $j<$al-1; $j++)  
    {  
        $rpb .= '../';  
        $rpa .= $apa_arr[$j].'/';  
    }  
    for($j=$i; $j<$bl-1; $j++)  
    {  
        $rpa = '../'.$rpa;  
        $rpb .= $apb_arr[$j].'/';  
    }  
    $rpa .= $apa_arr[$al-1];  
    $rpb .= $apb_arr[$bl-1];  
    return true;  
}  
```

使用样例进行测试：
分别为：

1、两个路径中其中一个为空串

2、两个路径在不同的目录下

3、两个路径在同一目录下

4、其中一个路径的父路径和另一条路径在同一目录下

测试程序：

```php
<?php  
echo "绝对路径 to 相对路径：<br/>";  
  
function output($apa, $apb)  
{  
    echo "串a：$apa<br/>串b:$apb<br/>";  
    if(transfer($apa, $apb, $a, $b))  
        echo 'a相对于b的路径：'.$a.'<br/>b相对于a的路径：'.$b.'<br/>';  
    else  
        echo '不存在相对路径';  
    echo '<br/><br/>';  
}  
  
output('/home/web/test/a.php','');  
output('/home/web/test/a.php','/home/data/d.png');  
output('/home/web/test/a.php','/home/web/test/d.png');  
output('/home/web/test/myyy/a.php','/home/web/test/d.png');  
?>  
```

结果：

```
绝对路径 to 相对路径：  
串a：/home/web/test/a.php  
串b:  
不存在相对路径  
  
串a：/home/web/test/a.php  
串b:/home/data/d.png  
a相对于b的路径：../web/test/a.php  
b相对于a的路径：../../data/d.png  
  
串a：/home/web/test/a.php  
串b:/home/web/test/d.png  
a相对于b的路径：a.php  
b相对于a的路径：d.png  
  
串a：/home/web/test/myyy/a.php  
串b:/home/web/test/d.png  
a相对于b的路径：myyy/a.php  
b相对于a的路径：../d.png  
```
