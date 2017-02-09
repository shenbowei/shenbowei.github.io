---
layout: post
title: PHP 设计模式 IoC - Dependency Injection
category: 技术
author: shenbowei
tags: PHP
---

* 目录
{:toc}

## 相关概念

`IoC`（Inversion of Control）模式又称`依赖注入`（Dependency Injection）模式。`控制反转`是将组件间的依赖关系从程序内部提到外部容器来管理，
而`依赖注入`是指组件的依赖通过外部以参数或者其他形式注入，两种说法本质上是一样的。
系统中通过引入实现了`IoC`模式的`IoC`容器，即可由`IoC`容器来管理对象的生命周期、依赖关系等，从而使得应用程序的配置和依赖性规范与实际的应用程序代码分开。
其中一个特点就是通过文本的配置文件进行应用程序组件间相互关系的配置，而不用重新修改并编译具体的代码。


## 一个例子

```
<?php
//IoC容器类，参考《Laravel框架关键技术解析》第六章，可以直接运行测试。
class Container
{
	//存储注册的回调函数，为键值对关联数组，
	//键为$abstract，值为对应注册的回调函数数组array("concrete"=>回调函数，"shared"=>boolean)
	protected $bindings = [];
	
	//注册函数，最开始需要调用本函数绑定接口和对应回调函数到$bindings
	public function bind($abstract, $concrete = null, $shared = false)
	{
		if (!$concrete instanceof Closure) {
			$concrete = $this->getClosure($abstract, $concrete);
		}
		$this->bindings[$abstract] = compact('concrete','shared');
	}

	//默认生成实例的回调函数
	protected function getClosure($abstract, $concrete)
	{
		return function($c) use ($abstract, $concrete)
		{
			$method = ($abstract == $concrete) ? 'build' : 'make';
			return $c->$method($concrete);
		};
	}

	//生成实例化的对象，被用户直接调用的方法。首先解决接口和要实例化类之间的依赖关系
	public function make($abstract)
	{
		$concrete = $this->getConcrete($abstract);
		$object;
		if ($this->isBuildable($concrete, $abstract)) {
			$object = $this->build($concrete);
		} else {
			$object = $this->make($concrete);
		}
		return $object;
	}

	//判断是否$concrete可以直接使用build实例化
	protected function isBuildable($concrete, $abstract)
	{
		return $concrete === $abstract || $concrete instanceof Closure;
	}

	//获取在$bindings中的回调函数，如果不存在说明之前并未注册，则直接返回，尝试使用反射实例化
	protected function getConcrete($abstract)
	{
		if (!isset($this->bindings[$abstract])) {
			return $abstract;
		}
		return $this->bindings[$abstract]['concrete'];
	}

	//实例化对象，如果是回调函数，直接调用其进行实例化
	//如果不是则尝试通过反射实例化
	public function build($concrete) 
	{
		if ($concrete instanceof Closure) {
			return $concrete($this);
		}
		//生成反射类
		$reflector = new ReflectionClass($concrete);
		//判断是否可以实例化
		if (!$reflector->isInstantiable()) {
			echo $message = "Target ($concrete) is not instantiable";
			return NULL;
		}
		//获取构造函数
		$constructor = $reflector->getConstructor();
		if (is_null($constructor)) {
			return new $concrete;
		}
		//获取构造函数需要的参数
		$dependencies = $constructor->getParameters();
		//生成参数，同样是调用了make来生成
		$instances =$this->getDependencies($dependencies);
		//生成实例化对象，并返回
		return $reflector->newInstanceArgs($instances);
	} 

	//根据array(ReflectionParameter)来获取构造函数需要的参数数组
	protected function getDependencies($parameters)
	{
		$dependencies = [];
		foreach ($parameters as $parameter) 
		{
			//获取参数的类名，除了自定义类对象，均会返回NULL
			$dependency = $parameter->getClass();
			if (is_null($dependency)){
				$dependencies[] = NULL;
			} else {
				//自动生成对象参数
				$dependencies[] = $this->resolveClass($parameter);
			}
		}
		return (array) $dependencies;
	}
	
	protected function resolveClass(ReflectionParameter $parameter)
	{
		//调用make生成对象
		return $this->make($parameter->getClass()->name);
	}
	
}

//IoC容器使用方法
//定义一个接口
interface Visit
{
	public function go();
}
//实现了接口的两个类
class Train implements Visit
{
	public function go(){
		echo "go to by Train!\n";
	}
}

class Bus implements Visit
{
	public function go(){
		echo "go to by Bus!\n";
	}
}

//需要使用IoC容器的类
class Traveller
{
	protected $trafficTool;
	public function __construct(Visit $trafficTool)
	{
		$this->trafficTool = $trafficTool;
	}
	public function visit()
	{
		$this->trafficTool->go();
	}
	
}

//创建一个IoC容器
$container= new Container();
//注册一个"Visit"为Train类的接口
$container->bind("Visit", "Train");
//注册一个"traveller"为Traveller类的接口
$container->bind("traveller","Traveller");
//实例化一个"traveller"的对象，实际是实例化Traveller对象，而Traveller需要"Visit"的对象参数，
//则传进去之前注册的Train
$tra = $container->make("traveller");
//输出：go to by Train!
$tra->visit();

//注册一个"Visit"为BUS类的接口
$container->bind("Visit", "Bus");
//此时需要的参数"Visit"已经被替换为Bus
$tra = $container->make("traveller");
//输出：go to by Bus!
$tra->visit();
```

上边这个例子是从`Laravel`中简化出来的，去掉了一些不太相关的部分，所以看上去可能有些繁琐，有些代码判断有些多余。
只想看`IoC`容器简单实现的可以参考下面列出的参考文件，其中前4个都有代码示例。

> ## 参考文献
>
>[PHP程序员如何理解依赖注入容器(dependency injection container)](https://segmentfault.com/a/1190000002424023 "跳转")
>
>[What is Dependency Injection?](http://fabien.potencier.org/what-is-dependency-injection.html "跳转")
>
>[理解PHP 依赖注入\|Laravel IoC容器](http://blog.csdn.net/realghost/article/details/35212285 "跳转")
>
>[Laravel 学习笔记 —— 神奇的服务容器](https://laravel-china.org/topics/789 "跳转")
>
>[控制反转](http://baike.baidu.com/link?url=dpmEyW8L30y-OWoBGCwZcJvdc_8DmaX_-lsPxOCKBaJ3roMDc4ozcjYn83XTejWFj9yZobwsVbLZT1Cob5sdpfKYAK6sag2siZoM6cEFVYB-czc0s51YyAlhbE3eKPSaemmB8skbTO2xA667sKfTc8QPQ9_Vm-p_mmwP_ncvvr7 "跳转")
>
>《Laravel框架关键技术解析》第六章







