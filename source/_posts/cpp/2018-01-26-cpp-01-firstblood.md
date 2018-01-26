---
title: C++  01 初体验
date: 2018-01-26 14:25:46
tags: cpp
---

## 神奇的输入输出流

```c
#include <iostream>

int main()
{
	std::cout << "Hello C艹" << std::endl;

	std::cout << "Enter tow number:" << std::endl;
	int v1, v2;
	std::cin >> v1 >> v2;	// 输入
	std::cout << v1 << " + " << v2 << " = " << v1 + v2 << std::endl; // 输出结果

	return 0;
}
```

## 面对对象的类

对象操作  
`Sales_item.h`内容见 [Sales_item.h](https://github.com/moneys/IT-diary/blob/master/code/C%2B%2B/Sales_item.h)

```c
#include <iostream>
#include "Sales_item.h"

int main()
{
	// 读写 Sales_item对象
	Sales_item book;
	std::cin >> book;	// 通过重载 >> 符号实现
	std::cout << book << std::endl; // 通过重载 << 符号实现

	Sales_item item1, item2;
	std::cin >> item1 >> item2;
	if (item1.same_isbn(item2))		// 成员函数调用
	{
		std::cout << item1 + item2 << std::endl; // 项目相加，重载 + 符号
		return 0;
	}
	else {
		std::cerr << "Date must refer to same ISBN" << std::endl;
	}

	return 0;
}
```
