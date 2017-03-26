---
layout: post
title: PHP 语言结构include和require比较
category: 技术
author: shenbowei
tags: PHP
---

* 目录
{:toc}

## 相关说明

`PHP`中的`include`和`require`作为一组功能相同的`语言结构`，经常被放到一起比较。
之前在网上查询相关对比时，发现了众多的说法，总结下基本是如下三项：

1. 返回值不同(`include`有返回值，而`require`没有)  **经验证是错误的！**

2. 在控制流程中的执行不同(`include`会受到控制流程影响，而`require`不会)  **经验证是错误的！**

3. 错误处理不同(`require`在出错时产生`E_COMPILE_ERROR`级别的错误,而`include`只产生警告（`E_WARNING`），脚本会继续运行。)  **正确！**

在`PHP`手册中可以看到如下描述：

> `require`和`include`几乎完全一样，除了处理失败的方式不同之外。`require`在出错时产生`E_COMPILE_ERROR`级别的错误。换句话说将导致脚本中止而`include`只产生警告（`E_WARNING`），脚本会继续运行。

这个描述是符合第`3`条描述，和`1`、`2`矛盾，下面进行验证。


## 验证

我验证使用的`PHP`环境如下：

```
PHP 5.6.8 (cli) (built: Apr 15 2015 15:07:09)
Copyright (c) 1997-2015 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2015 Zend Technologies
```

1. 返回值不同(`include`有返回值，而`require`没有)  **经验证是错误的！**

	测试代码如下：

		<?php

		$return_include = include 'test.php';
		var_dump($return_include);
		$return_require = require 'test.php';
		var_dump($return_require);

		$return_include = include 'null.php';
		var_dump($return_include);
		$return_require = require 'null.php';
		var_dump($return_require);


	若`test.php`的内容为：

		<?php

		$test_a = "test_a";

	结果输出如下：

		//test.php存在，没有返回值；null.php不存在

		int(1)
		int(1)

		Warning: include(null.php): failed to open stream: No such file or directory in
		E:\shenbowei.github\测试程序\include-require\return.php on line 8

		Warning: include(): Failed opening 'null.php' for inclusion (include_path='.;C:\
		xampp\php\PEAR') in E:\shenbowei.github\测试程序\include-require\return.php on l
		ine 8
		bool(false)

		Warning: require(null.php): failed to open stream: No such file or directory in
		E:\shenbowei.github\测试程序\include-require\return.php on line 10

		Fatal error: require(): Failed opening required 'null.php' (include_path='.;C:\x
		ampp\php\PEAR') in E:\shenbowei.github\测试程序\include-require\return.php on li
		ne 10

	若`test.php`的内容为：

		<?php

		$test_a = "test_a";

		return $test_a;

	结果输出如下：

		//test.php存在，有返回值；null.php不存在

		string(6) "test_a"
		string(6) "test_a"

		Warning: include(null.php): failed to open stream: No such file or directory in
		E:\shenbowei.github\测试程序\include-require\return.php on line 8

		Warning: include(): Failed opening 'null.php' for inclusion (include_path='.;C:\
		xampp\php\PEAR') in E:\shenbowei.github\测试程序\include-require\return.php on l
		ine 8
		bool(false)

		Warning: require(null.php): failed to open stream: No such file or directory in
		E:\shenbowei.github\测试程序\include-require\return.php on line 10

		Fatal error: require(): Failed opening required 'null.php' (include_path='.;C:\x
		ampp\php\PEAR') in E:\shenbowei.github\测试程序\include-require\return.php on li
		ne 10

	**结论：`include`和`require`都是有返回值的，且当被包含文件中没有`return`时，均会返回`int(1)`,而当被包含文件有`return`时，均返回对应值。**
	**而当被包含文件不存在时，`include`结构只会发出`Warning`，并继续执行，放回`bool(false)`;而`require`则会发出`Fatal error`,并终止程序执行。**

	**因此，本条描述被证实是错误的，而`3`是正确的。**

2. 在控制流程中的执行不同(`include`会受到控制流程影响，而`require`不会)  **经验证是错误的！**

	测试代码如下，先来测试一下`if`语句：

		<?php

		echo "include:\n";

		if (true) {
			echo "1:\n";
			include 'true.php';
		} else {
			echo "2:\n";
			include 'false.php';
		}

		echo "require:\n";

		if (false) {
			echo "3:\n";
			require 'true.php';
		} else {
			echo "4:\n";
			require 'false.php';
		}
		
	其中包含的文件`true.php`为：

		<?php

		echo "in true.php\n";
	
	`false.php`为：
	
		<?php

		echo "in false.php\n";
		
	输出的结果如下所示：
	
		include:
		1:
		in true.php
		require:
		4:
		in false.php
		
	接下来再来测试一下`for`语句：
	
		<?php

		for ($i=0; $i<5; ++$i) {
			echo "$i:\n";
			include 'true.php';
			require 'false.php';
		}
		
	输出的结果为：
	
		0:
		in true.php
		in false.php
		1:
		in true.php
		in false.php
		2:
		in true.php
		in false.php
		3:
		in true.php
		in false.php
		4:
		in true.php
		in false.php
		
	**结论：输出结果证实`include`和`require`都是受流程控制语句影响的。**
	
	**因此，本条描述被证实是错误的。**

3. 错误处理不同(`require`在出错时产生`E_COMPILE_ERROR`级别的错误,而`include`只产生警告（`E_WARNING`），脚本会继续运行。)  **正确！**

	在`1`中已经证实是`正确`的。

## 结论

经过证实，`include`和`require`两个语言结构只有上述`3`中所说的区别：

	错误处理不同：`require`在出错时产生`E_COMPILE_ERROR`级别的错误,而`include`只产生警告（`E_WARNING`），脚本会继续运行。
		

## 延伸

1. `PHP`中`include`和`require`的运行机制到底是什么样的？

	> 当一个文件被包含时，语法解析器在目标文件的开头脱离`PHP`模式并进入`HTML`模式，到文件结尾处恢复。
	
	那么就相当于把目标文件中的内容插入到`include`/`require`的位置（会在之前和之后进行模式切换），
	而假如包含的目标文件是纯`PHP`代码，就相当于直接把目标文件中的代码直接放到`include`/`require`处。
	以上的理解不知道有没有问题，欢迎指正。

2. `include`、`require`和`include_path`的关系？

	可以参考这篇文章：[深入理解PHP之require/include顺序](http://www.laruence.com/2010/05/04/1450.html "跳转")。
	总结下来就是：`include_path`会优先于`current_script_dir`作用于`include`、`require`。目录相对路径的`basedir`, 永远都是当前工作路径。
	在模块化的系统设计中, 一般应该在模块内, 通过获取模块的部署路径(`dirname(__FILE__)`, php5.3以后更是提供了`__DIR__`常量)从而使用绝对路径。
	
另外，[php-include](http://php.net/manual/zh/function.include.php "跳转")中的很多例子都是使用中值得注意的。
	

> ## 参考文献
>
>[php-require](http://php.net/manual/zh/function.require.php "跳转")
>
>[php-include](http://php.net/manual/zh/function.include.php "跳转")
>
>[重谈php的include和require](http://blog.csdn.net/mingzhou/article/details/9116079 "跳转")
>
>[深入理解PHP之require/include顺序](http://www.laruence.com/2010/05/04/1450.html "跳转")









