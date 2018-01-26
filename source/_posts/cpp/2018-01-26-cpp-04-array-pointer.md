---
title: cpp-04-array-pointer
date: 2018-01-26 14:29:04
tags: cpp
---

## 数组、指针、动态数组


```cpp
#include <iostream>
#include <string>
#include <bitset>

using std::cin;
using std::cout;
using std::endl;
using std::string;

int main()
{
	// 显示初始化 ， 数字长度为固定值
	const unsigned size = 3;
	int ia[size] = { 0,1,2 };
	// 使用下标遍历
	for (size_t ix = 0; ix != size; ++ix)
		cout << ix << ":" << ia[ix] << endl;
	// 使用指针遍历
	for (int *p = ia; p <  ia + size; p++)
		cout << p << ":" << *p << endl;
	// 创建动态数组
	int *pia = new int[10]; // array of 10 uninitialized ints
	int *pia2 = new int[10](); //数组初始化 0
	delete[] pia;	// 动态空间释放
	delete[] pia2;
	return 0;
}
```

还有const、C字符串、二维数组。没啥好记的，用到时再来翻看。