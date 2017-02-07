---
layout: post
title: PHP 反射机制--Reflection API
category: 技术
author: shenbowei
tags: PHP
---

* 目录
{:toc}

## 相关概念

> `PHP5`具有完整的反射`API`，添加了对类、接口、函数、方法和扩展进行反向工程的能力。此外，反射`API`提供了方法来取出函数、类和方法中的文档注释。

通过查阅相关资料，对于`php`中的反射，我的理解如下：

- `PHP`在运行时，动态地扩展分析`PHP`程序中的数据信息（包括`class, interface, trait, property, method`），导出或提取关于类、方法、属性、参数、注释等详细信息。

- 这种动态不仅体现在获取信息方面，还可以根据类名等信息动态地实例化对象（不是通过`new`来创建），以及调用对象的方法。

- `PHP`反射的实现包括：`Introspection Functions`（内省函数，可参考[Laravel 学习笔记之 PHP 反射 (Reflection) (上)](https://laravel-china.org/topics/2910 "跳转")）和`Reflection Api`（即本文所写的笔记）。

反射的概念不难理解，如何运用通过下面的实例学习。

## 实例

下面这个`PHP`脚本展示了反射的一些用法，可以直接运行：

```
<?php
class Person {  
	/** 
     * 注释：person的属性id
     */ 
    private $id = 0;
 
    /** type=varchar length=255 null */
    protected $name = "new born";
 
    /** type=text null */
    public $biography;
	
	public function __construct($id, $name)
	{
		$this->id = $id;
		$this->name = $name;
	}
 
	public function getId()
	{
		return $this->id;
	}
	public function setId($v)
	{
		$this->id = $v;
	}
	public function getName()
	{
		return $this->name;
	}
	public function setName($v)
	{
		$this->name = $v;
	}
	public function getBiography()
	{
		return $this->biography;
	}
	public function setBiography($v)
	{
		$this->biography = $v;
	}
}

//构造实例化--------------------------------------------------------------

//建立Person这个类的反射类  
$reflector = new ReflectionClass('Person');
//实例化Person类对象，根据构造函数传递参数
$instance = $reflector->newInstanceArgs(array(100,"shenbowei"));
//输出:id:100 name:shenbowei
echo "id:".$instance->getId()." name:".$instance->getName()."\n";
//或者可以这样调用方法
//获取Person类中的getName方法
$getNameMethod=$reflector->getmethod('getName'); 
//传入实例$instance，调用对应方法。输出：name:shenbowei   
echo "name:".$getNameMethod->invoke($instance)."\n";

//获取Person类的构造函数
$construtor = $reflector->getConstructor();
//获取函数方法需要的参数array
$denpendencies = $construtor->getParameters();
/*输出：
array(2) {
  [0]=>
  &object(ReflectionParameter)#4 (1) {
    ["name"]=>
    string(2) "id"
  }
  [1]=>
  &object(ReflectionParameter)#5 (1) {
    ["name"]=>
    string(4) "name"
  }
}
*/
var_dump($denpendencies);
/*上例的输出为两个NULL，为了测试，我在构造函数的第一个参数$id前加上类型限制为Person对象，则会输出如下：
object(ReflectionClass)#5 (1) {
  ["name"]=>
  string(6) "Person"
}
NULL
*/
foreach($denpendencies as $parameter)
{
	//如果没有限定参数类型为具体一个类的对象，则会输出NULL
	var_dump($parameter->getClass());
}

//获取属性--------------------------------------------------------------

//获取Person类的所有属性，不限定访问控制
$properties = $reflector->getProperties();  
/*输出：
id
name
biography
*/
foreach($properties as $property) {  
    echo $property->getName()."\n";  
}
//如果只要获取public和protect的属性
/*
ReflectionProperty::IS_STATIC
ReflectionProperty::IS_PUBLIC
ReflectionProperty::IS_PROTECTED
ReflectionProperty::IS_PRIVATE
*/
$without_private_properties = $reflector->getProperties(ReflectionProperty::IS_PUBLIC|ReflectionProperty::IS_PROTECTED);
/*输出：
name
biography
*/
foreach($without_private_properties as $property) {  
    echo $property->getName()."\n";  
}

//获取注释--------------------------------------------------------------

/*
输出：varchar
*/
foreach($properties as $property) {  
    if($property->isProtected()) {  
        $docblock = $property->getDocComment();  
		preg_match('/type\=([a-z_]*)/', $docblock, $matches);  
        echo $matches[1]."\n";
    }  
} 

//获取方法--------------------------------------------------------------
/*和getProperties方法类似，可以添加如下的filter作为参数：
ReflectionMethod::IS_STATIC
ReflectionMethod::IS_PUBLIC
ReflectionMethod::IS_PROTECTED
ReflectionMethod::IS_PRIVATE
ReflectionMethod::IS_ABSTRACT 
ReflectionMethod::IS_FINAL
*/
$methods = $reflector->getMethods(); 
/*输出：
__construct
getId
setId
getName
setName
getBiography
setBiography
*/
foreach($methods as $method) {  
    echo $method->getName()."\n";  
}
```

## 附录

`Reflection Api`相关的类、`api`接口可以在如下的链接中查阅到：

[php manual 反射](http://php.net/manual/zh/book.reflection.php "跳转")

> ## 参考文献
>
>[php manual 反射](http://php.net/manual/zh/book.reflection.php "跳转")
>
>[理解php反射机制-2](http://blog.csdn.net/clh604/article/details/24623841 "跳转")
>
>[Laravel 学习笔记之 PHP 反射 (Reflection) (上)](https://laravel-china.org/topics/2910 "跳转")







