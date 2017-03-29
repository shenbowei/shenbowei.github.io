---
layout: post
title: PHP Array Functions
category: 技术
author: shenbowei
tags: PHP
---

* 目录
{:toc}

最近看`PHP`相关代码的时候，发现遇到了很多功能强大的数组函数，而且有些函数理解起来还是比较复杂的，因此重新学习和总结下`PHP`中的数组函数。
下面将学习过程中感觉比较难理解的数组函数记录下，其它较为简单的函数可以查看手册：[PHP手册-函数参考-变量与类型相关扩展-数组](http://php.net/manual/zh/ref.array.php "跳转")。

## array_chunk

> `array array_chunk ( array \$input , int \$size [, bool \$preserve_keys = false ] )`

将一个数组分割成多个数组，其中每个数组的单元数目由`size`决定。最后一个数组的单元数目可能会少于`size`个。

值得注意的是第三个参数：`$preserve_keys`:
设为`TRUE`，可以使`PHP`保留输入数组中原来的键名。如果你指定了`FALSE`，那每个结果数组将用从零开始的新数字索引。默认值是`FALSE`。

那么对于索引为字符串的关联数组，如果第三个参数为`FALSE`，那么结果如何，下面进行测试：

```
<?php

$input_array = array("Apple"=>"1","Boy"=>"2","Cat"=>"3","Dog"=>"4","Egg"=>"5");
print_r(array_chunk($input_array, 2, true));
print_r(array_chunk($input_array, 2, false));

/*
output：
Array
(
    [0] => Array
        (
            [Apple] => 1
            [Boy] => 2
        )

    [1] => Array
        (
            [Cat] => 3
            [Dog] => 4
        )

    [2] => Array
        (
            [Egg] => 5
        )

)
Array
(
    [0] => Array
        (
            [0] => 1
            [1] => 2
        )

    [1] => Array
        (
            [0] => 3
            [1] => 4
        )

    [2] => Array
        (
            [0] => 5
        )

)
*/
```

**结论：当`\$preserve_keys = false`时，无论数组键值是不是数字，子数组都会按照数字从`0`开始重新索引。**

## array_filter

> `array array_filter ( array \$array [, callable \$callback [, int \$flag = 0 ]] )`

用回调函数过滤数组中的单元。依次将`array`数组中的每个值传递到`callback`函数。如果`callback`函数返回`TRUE`，则`input`数组的当前值会被包含在返回的结果数组中。数组的键名保留不变。

其中的参数：`flag`决定callback接收的参数形式:

- `ARRAY_FILTER_USE_KEY` - callback接受键名作为的唯一参数
- `ARRAY_FILTER_USE_BOTH` - callback同时接受键名和键值

下面实例测试下：

```
<?php

function filter($val)
{
    return !($val === 1);
}

function filter_key($key)
{
    return !($key === "e");
}

function filter_assoc($val, $key) //注意这里顺序
{
    return !($val == 2 || $key == "a");
}

$array = array("a"=>1, "b"=>2, "c"=>3, "d"=>4, "e"=>5);

echo "filter :\n";
print_r(array_filter($array, "filter"));
echo "filter_key :\n";
print_r(array_filter($array, "filter_key", ARRAY_FILTER_USE_KEY));
echo "filter_assoc :\n";
print_r(array_filter($array, "filter_assoc", ARRAY_FILTER_USE_BOTH));

/*
output:
filter :
Array
(
    [b] => 2
    [c] => 3
    [d] => 4
    [e] => 5
)
filter_key :
Array
(
    [a] => 1
    [b] => 2
    [c] => 3
    [d] => 4
)
filter_assoc :
Array
(
    [c] => 3
    [d] => 4
    [e] => 5
)
*/
```

## array_map

> `array array_map ( callable \$callback , array \$array1 [, array \$... ] )`

为数组的每个元素应用回调函数。返回值为`array1`每个元素应用`callback`函数之后的数组。`callback`函数形参的数量和传给`array_map()`数组数量，两者必须一样。

下面测试下两个参数的回调函数，对应两个输入数组：

```
<?php

function show_English($n, $m)
{
    return("The number $n is called $m in English");
}

function map_English($n, $m)
{
    return(array($n => $m));
}

$number = array(1, 2, 3, 4, 5);
$english = array("one", "two", "three", "four", "five");

$array_show = array_map("show_English", $number, $english);
print_r($array_show);

$array_map = array_map("map_English", $number, $english);
print_r($array_map);

/*
output:
Array
(
    [0] => The number 1 is called one in English
    [1] => The number 2 is called two in English
    [2] => The number 3 is called three in English
    [3] => The number 4 is called four in English
    [4] => The number 5 is called five in English
)
Array
(
    [0] => Array
        (
            [1] => one
        )

    [1] => Array
        (
            [2] => two
        )

    [2] => Array
        (
            [3] => three
        )

    [3] => Array
        (
            [4] => four
        )

    [4] => Array
        (
            [5] => five
        )

)
*/
```

## array_multisort

> `bool array_multisort ( array &\$arr [, mixed \$arg = SORT_ASC [, mixed \$arg = SORT_REGULAR [, mixed \$... ]]] )`

对多个数组或多维数组进行排序。
但是值得注意的是，当给出多个数组的时候，并不是对每个数组分别排序（很容易产生这种误导）。而是对第一个数组排序，后边的数组的每个值和第一个保持一致调整。
即这几个数组需要维数一致。下面通过例子展示：

```
<?php

$ar1 = array(10, 100, 100, 0);
$ar2 = array(1, 3, 2, 4);
array_multisort($ar1, SORT_ASC, SORT_REGULAR, $ar2, SORT_ASC, SORT_REGULAR);
//array_multisort($ar2, SORT_DESC, SORT_REGULAR);

var_dump($ar1);
var_dump($ar2);

/*
output:
array(4) {
  [0]=>
  int(0)
  [1]=>
  int(10)
  [2]=>
  int(100)
  [3]=>
  int(100)
}
array(4) {
  [0]=>
  int(4)
  [1]=>
  int(1)
  [2]=>
  int(2)
  [3]=>
  int(3)
}
*/
```

**注意：两个数组的对应关系为10->1、100->3、100->2、0->4。所以ar2才会出现4123的排序。array_multisort可以保证多维数组列保持一致。**

## array_reduce

> `mixed array_reduce ( array \$input , callable \$function [, mixed \$initial = NULL ] )`

用回调函数迭代地将数组简化为单一的值。`array_reduce()`将回调函数`function`迭代地作用到`input`数组中的每一个单元中，从而将数组简化为单一的值。
看实例和测试后，发现这里的回调函数需要有两个参数，这样才能实现将数组归并为一个值，
即将数组中的两个值通过回调函数返回一个值，其作为一个参数继续和数组下一个值传入到回调函数，最终得到一个值。
下面的例子是通过`array_reduce`来模拟一个处理管道：

```
<?php

function f0()
{
	echo "--in f0--";
}

function f1($f)
{
	echo "--in f1--\n";
	$f();
}

function f2($f)
{
	echo "--in f2--\n";
	$f();
}

function f3($f)
{
	echo "--in f3--\n";
	$f();
}

function f4($f)
{
	echo "--in f4--\n";
	$f();
}

function f5($f)
{
	echo "--in f5--\n";
	$f();
}

$array = array("f1","f2","f3","f4","f5");

call_user_func(array_reduce(
	$array, 
	function($stack, $pipe){ 
		return function() use($stack, $pipe){ return $pipe($stack); };
	}, 
	"f0"));
	
/*
outout:
--in f5--
--in f4--
--in f3--
--in f2--
--in f1--
--in f0--
*/
```

## array_walk

> `bool array_walk ( array &\$array , callable \$callback [, mixed \$userdata = NULL ] )`

使用用户自定义函数对数组中的每个元素做回调处理。典型情况下`callback`接受两个参数:`array`参数的值作为第一个，键名作为第二个。
如果提供了可选参数`userdata`，将被作为第三个参数传递给回调函数。

这里主要说下`array_walk`和`array_map`的区别。这两个函数单看文字描述可能不是很好区分。主要区别为：

- 返回值：`array_walk`的返回值是`bool`,而`array_map`的返回值为`array`。即`array_walk`只是要使用这个数组中的数据，而`array_map`则不仅仅要使用，
	还要通过这些数据构造一个新的数组返回。

- 回调函数：`array_walk`的回调函数可以没有返回值，而`array_map`的回调函数则必须要有返回值，因为这个返回值要构成返回的新数组。

- 额外参数：`array_walk`可以为回调函数提供可选参数`userdata`，而`array_map`不可以。

这里贴一个官方手册的例子：

```
<?php
$fruits = array("d" => "lemon", "a" => "orange", "b" => "banana", "c" => "apple");

function test_alter(&$item1, $key, $prefix) //注意这里的引用
{
    $item1 = "$prefix: $item1";
}

function test_print($item2, $key)
{
    echo "$key. $item2\n";
}

echo "Before ...:\n";
array_walk($fruits, 'test_print');

array_walk($fruits, 'test_alter', 'fruit');
echo "... and after:\n";

array_walk($fruits, 'test_print');

/*
output:
Before ...:
d. lemon
a. orange
b. banana
c. apple
... and after:
d. fruit: lemon
a. fruit: orange
b. fruit: banana
c. fruit: apple
*/
```

## compact

> `array compact ( mixed \$varname [, mixed \$... ] )`

创建一个数组，数组的键值为传入参数的名称，对应的值为参数对应变量的值。和`extract()`正好相反。
会自动忽略找不到变量的参数。官方实例如下：

```
<?php
$city  = "San Francisco";
$state = "CA";
$event = "SIGGRAPH";

$location_vars = array("city", "state");

$result = compact("event", "nothing_here", $location_vars);
print_r($result);

/*
output:
Array
(
    [event] => SIGGRAPH
    [city] => San Francisco
    [state] => CA
)
*/
```

## count

> `int count ( mixed \$var [, int \$mode = COUNT_NORMAL ] )`

这个函数应该很常用，这里主要讲下`mode`参数的用法：

- `COUNT_RECURSIVE（或 1）`：`count()`将递归地对数组计数,对计算多维数组的所有单元尤其有用。

- `默认值0`：不递归计数。

下面看个例子：

```
<?php
$food = array('fruits' => array('orange', 'banana', 'apple'),
              'veggie' => array('carrot', 'collard', 'pea'),
			  'favourite' => 'apple');

// recursive count
echo count($food, COUNT_RECURSIVE)."\n"; //fruits+1,递归里面的值+3;veggie+1,递归里面的值+3,favourite+1 = 9

// normal count
echo count($food); // fruits+1,不递归;veggie+1,不递归,favourite+1 = 3

/*
output:
9
3
*/
```

## extract

> `int extract ( array &\$var_array [, int \$extract_type = EXTR_OVERWRITE [, string \$prefix = NULL ]] )`

从数组中将变量导入到当前的符号表,这里的符号表为当前环境的变量集合。这里的`extract_type`可取值请参考：[extract_type](http://php.net/manual/zh/function.extract.php#refsect1-function.extract-parameters "跳转")。
`extract`会检查每个键名看是否可以作为一个合法的变量名，同时也检查和符号表中已有的变量名的冲突。返回值为成功导入到符号表中的变量数目。

```
<?php

$size = "large";
$var_array = array("color" => "blue",
                   "size"  => "medium",
                   "shape" => "sphere");
extract($var_array, EXTR_PREFIX_SAME, "wddx");//如果有冲突，在变量名前加上前缀"wddx"，所以会产生变量$wddx_size = "medium"

echo "$color, $size, $shape, $wddx_size\n";

/*
output:
blue, large, sphere, medium
*/
```

## list

> `array list ( mixed \$varname [, mixed \$... ] )`

把数组中的值赋给一些变量。像`array()`一样，这不是真正的函数，而是语言结构。`list()`用一步操作给一组变量进行赋值。

```
<?php

$info = array('coffee', 'brown', 'caffeine');

// 列出所有变量
list($drink, $color, $power) = $info;
echo "$drink, $color, $power\n";

// 列出他们的其中一个
list($drink2, , $power2) = $info;
echo "$drink2, $power2\n";

// 或者让我们跳到仅第三个
list( , , $power3) = $info;
echo "$power3\n";

// list() 不能对字符串起作用
list($bar) = "abcde";
var_dump($bar); // NULL

/*
output:
coffee, brown, caffeine
coffee, caffeine
caffeine
NULL

*/
```

## array pointer 函数集

关于数组指针，有一系列函数可以使用：

- `current` ：返回数组中的当前单元，初始指向插入到数组中的第一个单元。

- `each` : 返回数组中当前的键／值对并将数组指针移动到下一个单元。

- `end` : 将数组的内部指针移动到最后一个单元并返回其值。

- `key` : 返回数组中当前单元的键名。

- `next` : 数组中的内部指针移动到下一个单元并返回其值，或当没有更多单元时返回`FALSE`。

- `pos` : `current()`的别名。

- `prev` : 将数组的内部指针倒回一位并返回其值，或当没有更多单元时返回`FALSE`。

- `reset` ：将数组的内部指针指向第一个单元。

每个函数的具体用法和实例可以参考官方手册。

> ## 参考文献
>
>[PHP手册-函数参考-变量与类型相关扩展-数组](http://php.net/manual/zh/ref.array.php "跳转")
>
>[PHP Array 函数](http://www.w3school.com.cn/php/php_ref_array.asp "跳转")
>








