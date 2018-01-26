---
title: cpp-05-statement
date: 2018-01-26 14:29:45
tags: cpp
---
## 简单语句

分号结束

## 复合语句

代码块，判断、循环等待

## 语句作用域

语句内部

## if、switch、default、while、for、do-while、break、continue、goto


```cpp
#include <iostream>

using std::cin;
using std::cout;
using std::endl;

int main()
{
	int num;
	while (cin >> num)
	{
		if (num == 886) break;
		if (num == 520) continue;
		if (num == 69) goto ok;
		if (num % 2 == 0)
			cout << num << " is 偶数 " << endl;
		else
			cout << num << " is 奇数 " << endl;

	}
ok:
	cout << num << " is ok " << endl;
	return 0;
}
```

## 异常处理 throw、try、catch


```cpp
#include <iostream>

using std::cin;
using std::cout;
using std::endl;

double divide(int n1, int n2);

int main()
{
	try
	{
		double n3 = divide(1, 0);
		cout << "相除得：" << n3 << endl;
	}
	catch (std::runtime_error err)
	{
		cout << "相除异常：" << err.what() << endl;
	}
}

double divide(int n1, int n2)
{
	if (n2 == 0) throw std::runtime_error("被除数不能为0！");
	return (double)n1 / n2;
}

```
