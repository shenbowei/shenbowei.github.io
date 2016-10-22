---
layout: post
title: Latex常见数学符号记录
category: 技术
author: 沈伯伟
tags: Latex
---

* 目录
{:toc}

Latex 是一种排版系统，适用于生成高印刷质量的科技和数学类文档。
下面将粗略记录一下常见的Latex数学符号，
当作之后记录数学公式时的查询的一个备忘录。

## 数学环境符号

编辑的数学公式应该位于 Latex 数学环境符号内，包括：

```
1. 行内公式

	- \( 公式 \)
	
	- $ 公式 $

2. 行间公式

	- \begin{displaymath} 公式 \end{displaymath}
	
	- \[ 公式 \]
	
	- $$ 公式 $$
```

测试：

1. 行内公式

	- \\( 公式 \\)
	
	- $ 公式 $

2. 行间公式

	- \begin{equation} 公式 \end{equation}
	
	- \\[ 公式 \\]
	
	- $$ 公式 $$

## 指数和下标

```
指数符号：^		a^{asb}

下标符号：_		x_{234}
```

测试：

$a^{asb}, x_{234}$

## 上下划线

```
上划线：\overline{}	\overline{abcd}

下划线：\underline{}	\underline{def}
```

测试：

$\overline{abcd}, \underline{def}$

## 上下括号

```
上括号：\overbrace{}	\overbrace{1,2,3...n}^{n}

下括号：\underbrace{}	\underbrace{1,2,3...n}_{n}
```

测试：

$\overbrace{1,2,3...n}^{n},\underbrace{1,2,3...n}_{n}$

## 向量

```
\vec{}		\vec{abc}

\overrightarrow{}	\overrightarrow{efg}

\overleftarrow{}	\overleftarrow{hi}
```

测试：

$\vec{abc}, \overrightarrow{efg}, \overleftarrow{hi}$

## 分数

```
\frac{分子}{分母}
```

测试：

$\frac{分子}{分母}$

## 积分 乘积

```
- \int_{下限}^{上限}

- \sum_{下限}^{上限}

- \prod_{下限}^{上限}
```

测试：

- $\int_{下限}^{上限}$

- $\sum_{下限}^{上限}$

- $\prod_{下限}^{上限}$

## 附录

Latex 中的数学符号还有很多，具体可以参考[一份不太简短的 LATEX2e 介绍](/assets/files/lshort-cn.pdf "跳转")中的 `3.10` 节

> ## 参考文献
>
> [常用数学符号的 LaTeX 表示方法](http://www.mohu.org/info/symbols/symbols.htm "跳转")
>
> [一份不太简短的 LATEX2e 介绍](/assets/files/lshort-cn.pdf "跳转")








