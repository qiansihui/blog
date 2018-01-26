---
title: C++  14 重载操作符与转换
date: 2018-01-26 14:33:22
tags: cpp
---

# 重载操作符

Cat.h  
```cpp
#pragma once
#include <string>
#include <iostream>
using std::string; using std::ostream; using std::istream;
class Cat {
public:
	Cat() {}
	Cat(string name) :name(name) {};
	// 重载输入、输出操作符
	// 若供外部使用，写非成员函数的样子 添加友元
	// 成员函数的写法只要一个左操作符参数
	friend	ostream& operator<<(ostream& out, Cat& cat);
	friend	istream& operator>>(istream& in , Cat& cat);

private:
	string name;
};
```

Cat.cpp


```cpp
#include "Cat.h"

ostream & operator<<(ostream & out, Cat & cat)
{
	out << cat.name;
	return out;
}

istream & operator >> (istream & in, Cat & cat)
{
	in >> cat.name;
	return in;
}

```

重载函数的使用 main.cpp


```cpp
#include <iostream>
#include <sstream>
#include "Cat.h"
using namespace std;
int main()
{
	Cat cat ;
	cin >> cat; // 输入

	cout << cat << endl; // 输出
	return 0;
}

```

# 算术操作符重载

Book.h 

```cpp
#pragma once
#include <string>
#include <iostream>
using std::string; using std::ostream; using std::istream;

class Book
{
public:
	Book(const Book& b) { name = b.name; count = b.count; }
	Book(string name, int count) :name(name), count(count) {}
	friend Book operator+(const Book& lb, const Book& rb);
	friend bool operator==(const Book& lb, const Book& rb);
	friend	ostream& operator<<(ostream& out, Book& b);
	Book operator+=(const Book&rb);
private:
	string name;
	int count; // 数量
};

Book operator+(const Book & lb, const Book & rb)
{
	Book b(lb);
	b += rb;
	return b;
}

Book Book::operator+=(const Book & rb)
{
	count += rb.count;
	return *this;
}

inline bool operator==(const Book & lb, const Book & rb)
{
	return lb.name == rb.name && lb.count == rb.count;
}

ostream & operator<<(ostream & out, Book & b)
{
	out << "name:" << b.name << "，count:" << b.count ;
	return out;
}

```

算术符号重载后的使用


```cpp
#include <iostream>
#include <sstream>
#include "Book.h"
using namespace std;
int main()
{
	Book b1("piao", 1);
	Book b2("piao", 2);
	Book b3 = b1 + b2;
	Book b4("piao", 3);
	bool is = b3 == b4;
	cout << "重载加号，数量想加后为：" << b3 << endl;
	cout << "重载相等符号，b3 == b4：" << is << endl;
	return 0;
}

```

# 下标操作费、成员访问操作符

下标操作符重载示例


```cpp
#pragma once
#include<vector>
using namespace std;
class Foo {
public:
	int &operator[](const size_t);
	const int &operator[](const size_t) const;
private:
	vector<int> data;
};

int & Foo::operator[](const size_t index)
{
	return data[index];
}
inline const int & Foo::operator[](const size_t index) const
{
	return data[index];
}
```

使用解引用操作符与箭头符号重载，构建智能指针

指针计数类 ScrPtr.h

```cpp
#pragma once
#include "ScreenPtr.h"
#include "Screen.h"
class ScrPtr
{
	// 指针计数类
	friend class ScreenPtr;
	Screen *sp;
	size_t use;
	ScrPtr(Screen *p) :sp(p), use(1) {};
	~ScrPtr();
};


ScrPtr::~ScrPtr()
{
	delete sp;
}
```

指针封装对象 ScreenPtr.h

```cpp
#pragma once
#include "Screen.h"
#include "ScrPtr.h"
class ScreenPtr
{
public:
	ScreenPtr(Screen *p) :ptr(new ScrPtr(p)) {}
	ScreenPtr(const ScreenPtr& o) :ptr(o.ptr) { ++ptr->use; }
	~ScreenPtr() {
		// 若指针计数为0 则删除
		if (--ptr->use == 0)
			delete ptr;
	};
	// 重载解引用操作符 返回安全指针
	Screen &operator*() { return *ptr->sp; }
	// 重载箭头操作符 ->右操作数不是表达式，是对应类成员的标识符
	Screen *operator->() { return ptr->sp; }
private:
	ScrPtr *ptr;
};

```