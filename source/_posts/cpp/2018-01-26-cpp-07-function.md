---
title: C++  07 函数
date: 2018-01-26 14:30:13
tags: cpp
---

## 定义

函数者，黑盒是也。

## 参数传递

形参初始化：若形参为非引用类型，复制实参值，若未引用类型，为实参的别名。

指针形参与const形参


```cpp
#include <iostream>

using std::cin;
using std::cout;
using std::endl;

void swap(int * a, int * b); // 指针作为形参
void print(const int * a, const int * b); // const使参数不能被改变

int main()
{
	int a = 1, b = 2;
	print(&a, &b);
	swap(&a, &b);
	print(&a, &b);
}

void swap(int * a, int * b)
{
	int temp = *a;
	*a = *b;
	*b = temp;
}

void print(const int * a, const int * b)
{
	cout << "a :" << *a << ", b : " << *b << endl;
}

```


## 容器遍历查找


```cpp
#include <iostream>
#include <vector>

using std::cin;
using std::cout;
using std::endl;
using std::vector;

vector<int>::const_iterator find_val(
	vector<int>::const_iterator beg,
	vector<int>::const_iterator end,
	int value,
	vector<int>::size_type &occurs);

int main()
{
	vector<int> vi(10);
	vi[3] = 3;

	for (vector<int>::iterator iter = vi.begin(); iter != vi.end(); ++iter)
		cout << *iter << ",";
	cout << endl;
	vector<int>::size_type occurs;
	vector<int>::const_iterator citer = find_val(vi.begin(), vi.end(), 3, occurs);
	cout << "find them : " << *citer << " , occurs time : " << occurs << endl;
}

// 从迭代器范围内找到一个值，返回第一个位置，找不到则返回末尾。使用参数occurs表明
vector<int>::const_iterator find_val(vector<int>::const_iterator beg, vector<int>::const_iterator end, int value, vector<int>::size_type & occurs)
{
	vector<int>::const_iterator res_iter = end;
	occurs = 0;
	for (; beg != end; ++beg)
	{
		if (*beg == value)
		{
			if (res_iter == end) // 若找到值，将返回的迭代器指向第一个位置
				res_iter = beg;
			++occurs;
		}
	}
	return res_iter;
}

```


## 引用返回左值


```cpp
#include <iostream>
#include <string>

using std::cin;
using std::cout;
using std::endl;
using std::string;

// 返回引用
char &get_val(string &str, string::size_type ix)
{
	return str[ix];
}

int main()
{
	string s("hello");
	cout << s << endl;
	get_val(s, 1) = 'E'; // 对引用赋值

	cout << s << endl;
	return 0;
}

```

## 参数默认值


```cpp
#include <iostream>
#include <string>
#include <sstream>

using std::cin;
using std::cout;
using std::endl;
using std::string;
using std::ostringstream;

string screenInit(string::size_type hight = 20,
	string::size_type width = 30); // 参数默认值

int main()
{
	cout << "scree is " << screenInit(50);

	return 0;
}

string screenInit(string::size_type hight, string::size_type width)
{
	ostringstream strm;
	strm << "hight:" << hight << " and width: " << width << endl; // 类型转换
	return strm.str();
}

```

## 内联函数避免函数调用开销，放入头文件


```cpp
#include <iostream>
#include <string>

using std::cin;
using std::cout;
using std::endl;
using std::string;

// 内联函数避免函数调用开销，放入头文件
inline const string & shorterString(const string &s1, const string &s2)
{
	return s1.size() < s2.size() ? s1 : s2;
}

int main()
{
	cout << shorterString("aa", "bbb") << endl;
	return 0;
}


```

## 类的成员函数

`Dog.h`

```cpp
#pragma once
#include <iostream>
#include <string>

using std::string;

class Dog {
public: // 构造方法
	Dog(const string name)
	{
		Dog::name = name;
	}
	Dog(): name("ss"),age(1) {} //构造函数初始化列表
public: // 公共成员函数
	string get_name() const; // 返回值类型为const
	int get_age() {		return this->age;	}
	void set_age(int age) { this->age = age; }
	void to_string() const;
private: // 私有成员变量
	string name;
	int age;
};

inline  // 共有成员函数的内联实现
string Dog::get_name() const
{
	return Dog::name;
}
```

`main.c`


```cpp
#include <iostream>
#include "Dog.h"

using std::cin;
using std::cout;
using std::endl;
void Dog::to_string() const
{
	cout << "dog name : " << this->get_name() << ",age :" << this->age << endl;
}
int main()
{
	Dog dog; // 默认构造方法初始化
	dog.to_string();
	Dog tom("tom");
	tom.set_age(18);
	tom.to_string();
	return 0;
}

```

## 指向函数的指针


```cpp
#include <iostream>
#include <string>

using std::string;
using std::cout;
using std::endl;

int lengthCompare(const string &, const string &);

int(*ff(int))(int*, int); // 返回指向函数的指针 的函数调用

int f(int *, int); 

int main()
{
	// 使用typedef 简化函数指针使用
	typedef int(*compFcn)(const string &, const string &);
	compFcn pf = lengthCompare;
	int flag = lengthCompare("hi", "hello");
	cout << "compare result : " << flag << endl;
	flag = pf("hello", "hello");
	cout << "compare result : " << flag << endl;
	flag = (*pf)("hello", "hi");
	cout << "compare result : " << flag << endl;
	int i = 1;
	int(*fp)(int*, int) = ff(i); // 返回指向函数的指针 的函数调用
	int r = fp(&i, i); // 函数指针调用
	cout << "最终结果：" << r << endl;
	cout << "最终结果,一步到位：" << ff(i)(&i,i) << endl;
	return 0;
}

int lengthCompare(const string &s1, const string &s2)
{
	return s1.size() - s2.size();
}

int(*ff(int a))(int * p, int i)
{
	cout << "用参数a做点什么:" << a << endl;
	int(*fp)(int*, int) = f; // 返回函数指针
	return fp;
}

int f(int * p, int i)
{
	return (*p) + i;
}

```
