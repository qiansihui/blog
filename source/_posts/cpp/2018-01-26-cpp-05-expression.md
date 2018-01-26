---
title: cpp-05-expression
date: 2018-01-26 14:29:23
tags: cpp
---
## 算术表达式

加减乘除、取商取余数

## 关系操作符和逻辑表达啥

比较操作、逻辑判断

## 位操作符

取反，左右移，与或非

## bitset对象

设置位，重置位，取位值： 

```cpp
#include <iostream>
#include <bitset>

using std::cout;
using std::endl;
using std::bitset;

int main()
{
	bitset<4> bitset_quizl;
	unsigned long int_quizl = 0;
	cout << "bitset:" << bitset_quizl << "int_quizl:" << int_quizl << endl;

	bitset_quizl.set(2); // 将第n位置为1
	int_quizl |= 1UL << 2;	// 两个操作等价，移位后位或操作，将该位置为1
	cout << "bitset:" << bitset_quizl << "int_quizl:" << int_quizl << endl;

	bitset_quizl.reset(2);	// 将第n位置为0
	int_quizl &= ~(1UL << 2);	// 两个操作等价，移位后位取反再或操作，将该位置为0
	cout << "bitset:" << bitset_quizl << "int_quizl:" << int_quizl << endl;

	bool status1= bitset_quizl[2];	// 获取第n位的状态
	bool status2 = int_quizl & (1UL << 2); // 使用掩码取值
	cout << "bitset status: " << status1 << "int_quizl status :" << status2;
	return 0;
}
```

## 移位操作符之于IO

重载之后，输入输出。

## 赋值操作符

从右向左，优先级低，赋值表达式等于左值。

## 自增自减操作符

前置先操作，后置后操作

## 箭头操作符

通过指针调用。

## 条件操作符

三目运算

## sizeof操作符

返回对象或类型名长度，返回类型`size_t`

## 逗号操作符

隔开

## 复合表达式求值

优先级、结合性、求值顺序

## new和delete表达式

`new`出来的，迟早都要`delete`的

## 类型转换

