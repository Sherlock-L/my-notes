

## 故事起因
php 是弱类型语言，在某些特殊的值进行空判断要特别的小心。下面我对php **0 和“0”** 进行了多种判空测试。做这个测试的原因 主要是目前mysql数据表字段类型被我们的开发设计得比较随性，再加上撸码判空也继承了这种随性的作风。两方弱类型的缠绵，导致bug 出现得如此简单。

## 测试 
[测试代码地址](https://github.com/Sherlock-L/php-unit-test/blob/master/empty_fun_test.php)
测试的版本 php5.6 、 7.1
先甩一波测试结果：
```
/**
 *@description   7.1 version
 * 测试结果：
    $a = 0 满足条件 if($a == "") 
    $a = 0 不满足条件 if($a === "") 
    $a = 0 满足条件 if($a == "0") 
    $a = 0 不满足条件 if($a === "0") 
    $a = 0 满足条件 if($a == null) 
    $a = 0 不满足条件 if($a === null) 
    $a = 0 不满足条件 if($a) 
    $a = 0 满足条件 if(empty($a)) 

    $a = "0" 不满足条件if($a == "") 
    $a = "0" 不满足条件 if($a === "") 
    $a = "0" 满足条件 if($a == 0) 
    $a = "0" 不满足条件 if($a === 0) 
    $a = "0" 不满足条件 if($a == null) 
    $a = "0" 不满足条件 if($a === null) 
    $a = "0" 不满足条件 if($a) 
    $a = "0" 满足条件 if(empty($a)) 
**/

/**
 * @description   5.6 version
 * 测试结果：
    $a = 0 满足条件 if($a == "") 
    $a = 0 不满足条件 if($a === "") 
    $a = 0 满足条件 if($a == "0") 
    $a = 0 不满足条件 if($a === "0") 
    $a = 0 满足条件 if($a == null) 
    $a = 0 不满足条件 if($a === null) 
    $a = 0 不满足条件 if($a) 
    $a = 0 满足条件 if(empty($a)) 

    $a = "0" 不满足条件if($a == "") 
    $a = "0" 不满足条件 if($a === "") 
    $a = "0" 满足条件 if($a == 0) 
    $a = "0" 不满足条件 if($a === 0) 
    $a = "0" 不满足条件 if($a == null) 
    $a = "0" 不满足条件 if($a === null) 
    $a = "0" 不满足条件 if($a) 
    $a = "0" 满足条件 if(empty($a)) 
 */
```

## 结论
- 5.6 版本开始判空结果没有版本差异，后续5.4以下版本再补充验证
- 从测试结果看出对于类型要求严格的 最好用 **===** 全等。
- 0 和 “0”  empty() 都满足条件 返回true 
- 0 在非全等情况下，也认为是null和''   在if($a == null) 、if($a == "")  是成立的
-  当变量有可能为0 或者“0” 但是含义又一样时  避免使用if($var)这种写法

更多测试内容欢迎关注 [https://github.com/Sherlock-L/php-unit-test](https://github.com/Sherlock-L/php-unit-test)