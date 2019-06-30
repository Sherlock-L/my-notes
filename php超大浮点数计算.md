

假如有一些超大值的浮点数需要运算，我们又不想用科学技术法表示，可以将数字转成字符串包裹起来，**利用php 提供的数学函数计算，保证精度不丢失**。https://www.php.net/manual/zh/ref.bc.php
## 思路
- 1.先将数值用字符串包裹起来
- 2.运用php 提供的数学函数计算
- 3.打印结果

## php 常用数学函数
- **bcadd** ( string \$left , string   \$right [, int $scale ] ) : string 
解释：两个字符串数做**加法**，第三个参数为小数点保留位数，结果返回字符串数字

- **bcdiv** ( string \$left , string   \$right [, int $scale ] ) : string 
解释：两个字符串数做**除法**，第三个参数为小数点保留位数，结果返回字符串数字

- **bcmod** ( string \$left , string   \$right  ) : string 
解释：对左操作数使用系数**取模**，结果返回字符串数字

- **bcmul**  ( string \$left , string   \$right [, int $scale ] ) : string 
解释：两个字符串数做**乘法**，第三个参数为小数点保留位数，结果返回字符串数字

- **bcsub**  ( string \$left , string   \$right [, int $scale ] ) : string 
解释：两个字符串数做**减法**，第三个参数为小数点保留位数，结果返回字符串数字


## 实验

我对加法和乘法做了测试，另外还对关于php中 数字和字母组合的字符串的运算做了一个测试，我使用的是php7版本，有兴趣的可以用php5试试，看看有没有意想不到的结果。测试过程和结果如下

```
<?php 
//有时候我们的数字的值会很大，小数点也很多，又不想用科学技术法表示，我们可以将数字转成字符串包裹起来，保证精度不丢失
//同时php 还提供了数学函数，专门处理这些巨大的数字
 [php数学函数](https://www.php.net/manual/zh/ref.bc.php)
 error_reporting(3);
/****
    都报出来: error_reporting(1); 
　　不要报NOTICE: error_reporting(3); 
　　任何错误都不报: error_reporting(0); 
 */
$a = "10000000000000000000000000000000000000000000000000000000.10000000000";//保持精度，用字符串保存
$b = "99999999999999999999999999999999999999999999999999999999.00000000001";
echo "a:\n";
echo $a ;
echo "\n";
echo "b:\n";
echo $b ;
echo "\n";
echo "普通加法计算a+b：\n";
echo ($a+$b);//会转成科学计数法   
echo "\n";

echo "内置加法函数计算bcadd(\$a,\$b)：\n";
echo bcadd($a,$b,30); //相加 php 内置而数学函数，返回字符串，第三个参数是小数点的位数  
echo "\n";

echo "普通乘法法计算\$a*\$b：\n";
echo ($a*$b);//会转成科学计数法   
echo "\n";

echo "内置乘法函数计算bcmul(\$a,\$b,30)：\n";
echo bcmul($a,$b,30); //相加 php 内置而数学函数，返回字符串，第三个参数是小数点的位数 
echo "\n";
$c = "1.5ac1";
$d = "02.5xxx"; 
echo ($c+$d); //php7下php会发出notice 结果是4  因为php 匹配到02.5+1.5
echo "\n";


$c = "ac1";
$d = "xxx"; 
echo ($c+$d); //php7下php会发出warning警告 结果是0，因为碰到字符之前  ，没有匹配到数字，就认为是0+0
echo "\n";


```
## 结果截图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190524175246617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xzd19kYWJhb2ppYW4=,size_16,color_FFFFFF,t_70)

 可以看到，php的数学函数确实帮我们解决了超大值数据的计算和显示问题。同时php计算的兼容性做得够到位，但是有利有弊，对于类似'acs1'+'123.54asd'这样的运算，能不做就不做。虽然php 会给你计算结果。但是这个结果的对与错 还是有待思考。

## 参考代码
[php 测试脚本地址 （持续更新）](https://github.com/Sherlock-L/php-unit-test)