---
layout: post
title: 谈谈namespace,use和autoload
category: 技术
author: shenbowei
tags: PHP
---

这里不再重复`PHP`中的`namespace`,`use`和`autoload`的定义，网上已经有充足的参考文章，下面直接上例子：

## 先看例子

`./na/interfacea.php`:

```
<?php
namespace NA;

interface InterfaceA {
    public function doSomeThing();
}
```

`./na/classa.php`:

```
<?php
namespace NA;

class ClassA implements InterfaceA
{
	public function __construct()
	{
		echo "instantiate NA-ClassA\n";
	}
	
	public function doSomeThing()
	{
		echo "NA-ClassA do some thing\n";
	}
}
```

`./nb/classa.php`:

```
<?php
namespace NB;

class ClassA implements \NA\InterfaceA
{
	public function __construct()
	{
		echo "instantiate NB-ClassA\n";
	}
	
	public function doSomeThing()
	{
		echo "NB-ClassA do some thing\n";
	}
}
```

`./test.php`:

```
<?php

// 注册自动加载
spl_autoload_register('autoload');

function autoload($class)
{
	$file = dirname($_SERVER['SCRIPT_FILENAME']) . DIRECTORY_SEPARATOR . str_replace('\\', DIRECTORY_SEPARATOR, $class) . '.php';
	if (file_exists($file)) {
		echo "autoload:".$file."\n";
		require_once($file);
	} else {
		echo "autoload error: not exist:".$file."\n";
	}
   
}

use NA\ClassA;
use NB\ClassA as NB_ClassA;

$classA = new ClassA();
$classA->doSomeThing();

$classA_NB = new NB_ClassA();
$classA_NB->doSomeThing();
```

运行`php test.php`,输出结果为：

```
autoload:.\NA\ClassA.php
autoload:.\NA\InterfaceA.php
instantiate NA-ClassA
NA-ClassA do some thing
autoload:.\NB\ClassA.php
instantiate NB-ClassA
NB-ClassA do some thing
```

## 分析

- 上面定义了两个命名空间`NA`和`NB`.在命名空间`NA`下定义了接口`InterfaceA`和实现该接口的类`ClassA`；
	在命名空间`NB`下定义了一个实现了`NA`中接口`InterfaceA`的类`ClassA`。这里注意在`NB`中使用`NA`中数据的方法,
	例如上面`./nb/classa.php`中的`\NA\InterfaceA`（**注意，不能是`NA\InterfaceA`，这相当于`\NB\NA\InterfaceA`**），
	当然也可以使用`use`关键字导入，例如`use \NA\InterfaceA;`。
	
- 上述接口和类的定义文件中并没有出现`require(_once)`或者`include(_once)`，那么在没看`./test.php`文件的内容时，
	可能会让人产生错觉：文件`./na/classa.php`中使用的`InterfaceA`是如何被找到的？毕竟是在另一个文件中定义的，
	难道关键字`namespace`能够将一个命名空间中的其他数据引入到当前文件吗？可以尝试注释掉`./test.php`中的代码：
	`//spl_autoload_register('autoload');`，再次运行`php test.php`，输出为：
	
		Fatal error: Class 'NA\ClassA' not found in ...test.php on line 21
	
	这里是错误提示是找不到类`NA\ClassA`，还不能解释上面的问题，我们不妨再在`./test.php`的开头加上一句`require_once './na/classa.php';`。
	之后运行，结果为：
	
		Fatal error: Interface 'NA\InterfaceA' not found in ...\na\classa.php on line 5
		
	好了，这下明白了，文件`./na/classa.php`中使用的`InterfaceA`找不到了，所以`namespace`并没有我们猜测的那么“神通广大”，它只是起到了一个类似文件目录的分割组织作用。
	而真正“神通广大”的是`autoload`。
	
- 下面我们来看`./test.php`，关于自动加载的定义，这里不细说，可参考：[PHP手册-类的自动加载](http://www.php.net/manual/zh/language.oop5.autoload.php "跳转")。
	这里解释下`autoload`函数，它的作用是当找不到一个类、接口时，会将完整包括命名空间绝对路径的类名传进来，例如`NA\InterfaceA`。
	然后会将命名空间转换为文件路径（相对于`test.php`所在目录），类名或接口名转换为以`.php`为后缀的文件，例如`NA\InterfaceA`转换为文件路径`.\NA\InterfaceA.php`。
	最后，使用`file_exists`判断文件是否存在，若存在则调用`require_once`包含进来。这就解释了文件`./na/classa.php`中使用的`InterfaceA`是如何被找到的问题了。
	
- 再看下`./test.php`中的`use`使用，很简单，将命名空间中的数据导入。这里需要注意的是如果出现重名的数据，需要使用`use as`取别名，否则会引发`fatal error`。

- 最后，我们来看下输出，其中调用了自动加载三次，分别是实例化`ClassA`时寻找`.\NA\ClassA.php`,以及实例化`ClassA`时需要寻找其实现的接口`.\NA\InterfaceA.php`，
	最后是实例化`NB_ClassA`时寻找其定义`.\NB\ClassA.php`，因为之前已经引入了接口`.\NA\InterfaceA.php`，所以在实例化`NB_ClassA`时就不会再调用`autoload`了。
	
## 总结

- `namespace`只是起到了一个类似文件目录的分割组织作用。

- `use`起到将命名空间的数据导入（**注意：**这里的导入和`require(_once)`、`include(_once)`是不一样的）并起别名以防止重名或者简洁书写的作用。

- 一旦任何类或者接口找不到，注册的`autoload`函数会接受完整的类或者接口的名称进行相关处理。

> ## 参考文献
>
>[PHP命名空间 namespace 如何实现自动加载](https://segmentfault.com/q/1010000002426493 "跳转")
>
>[PHP手册-命名空间](http://php.net/manual/zh/language.namespaces.php "跳转")
>
>[PHP手册-类的自动加载](http://www.php.net/manual/zh/language.oop5.autoload.php "跳转")