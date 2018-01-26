---
title: C++ 13 复制控制
date: 2018-01-26 14:33:00
tags: cpp
---

## 复制构造函数

复制构造函数，传入同类型其他对象，初始化成员 。

```cpp
Message(const Message &m) :contents(m.contents), folders(m.folders){} ;
```

## 操作符重载

```cpp
	// 操作符重载，左操作数为this,又操作数为参数
	Message& operator=(const Message&);
```

## 析构函数

变量在超出作用域时自动撤销，动态分配对象被删除时调用。

```cpp
~Message();
```

## 消息处理示例
Message.h

```cpp
#pragma once
#include<string>
#include<set>
#include<vector>
#include<iostream>
#include "Folder.h"
using namespace std;

class Message
{
	friend class Folder;
public:
	Message(const string &str = "") :contents(str) {};
	// 复制构造函数，传入同类型其他对象，初始化成员
	Message(const Message &m) :contents(m.contents), folders(m.folders) {
		// 将消息 放入对应文件夹中
		put_msg_in_folders(folders);
	};
	// 操作符重载，左操作数为this,又操作数为参数
	Message& operator=(const Message&);
	// 析构函数
	~Message();

	// 将消息保存到文件夹中
	void save(Folder&);
	// 从文件夹中移除消息
	void remove(Folder&);

	vector<Folder*> get_folders();
	string print_message() { return contents; }
	void debug_print();

private:
	string contents; // 消息内容
	set<Folder*> folders; // 消息对应文件夹

	// 将消息放入文件夹
	void put_msg_in_folders(const set<Folder*>&);
	// 从文件夹移除消息
	void remove_msg_from_Folders();

	void addFolder(Folder *f) { folders.insert(f); };
	void remFolder(Folder *f) { folders.erase(f); };
};

```

Message.cpp

```cpp
#include "Message.h"
#include "Folder.h"

Message::~Message()
{
	remove_msg_from_Folders();
}

// 对原消息赋值，右操作数不改变，原消息改变成员和对应的文件夹关系
Message & Message::operator=(const Message &rhs)
{
	if (&rhs != this)
	{
		remove_msg_from_Folders(); // 将原消息从文件夹中删除
		contents = rhs.contents; // 复制成员
		folders = rhs.folders;
		put_msg_in_folders(rhs.folders); // 将原消息放入新文件夹中
	}
	return *this;
}


void Message::save(Folder &f)
{
	folders.insert(&f); // 添加消息与文件夹的关联
	f.addMsg(this);		// 添加文件夹与消息的关联
}

void Message::remove(Folder &f)
{
	folders.erase(&f); // 删除消息与文件夹的关联
	f.remMsg(this);		// 删除文件夹与消息的关联
}

vector<Folder*> Message::get_folders()
{
	return vector<Folder*>(folders.begin(), folders.end());
}

void Message::debug_print()
{
	cerr << "Message:\n\t" << contents << endl;
	cerr << "Appears in " << folders.size() << " Folders " << endl;
}

void Message::put_msg_in_folders(const set<Folder*> &rhs)
{
	for (set<Folder*>::const_iterator beg = rhs.begin(); beg != rhs.end(); ++beg)
		(*beg)->addMsg(this); // 遍历文件夹 添加消息
}

void Message::remove_msg_from_Folders()
{
	for (set<Folder*>::const_iterator beg = folders.begin(); beg != folders.end(); ++beg)
		(*beg)->remMsg(this); // 遍历文件夹 删除消息
}

```

Folder.h

```cpp
#pragma once
#include <set>
#include "Message.h"

class Folder
{
	friend class Message; // 设为友元以便访问私有成员
public:
	~Folder();
	// 复制构造函数
	Folder(const Folder&);
	Folder& operator=(const Folder&);

	Folder() {};
	// 添加消息，保存消息与文件夹的关联
	void save(Message& msg); 
	// 删除消息，删除消息与文件夹的关联
	void remove(Message& msg);

	vector<Message*> messages();
	void debug_print();

private:
	typedef set<Message*>::const_iterator Msg_iter;
	set<Message*> msgs;
	// 建立与消息的关联
	void copy_msgs(const set<Message*>&);
	// 删除消息列表与文件夹的关联
	void empty_msgs(); 
	void addMsg(Message *m) { msgs.insert(m); };
	void remMsg(Message *m) { msgs.erase(m); };
};


```
Folder.cpp
```cpp
#include "Folder.h"
#include "Message.h"

Folder::~Folder()
{
	empty_msgs();
}

Folder::Folder(const Folder &f)
{
	copy_msgs(f.msgs);
}

Folder & Folder::operator=(const Folder &f)
{
	if (&f != this)
	{
		empty_msgs();
		copy_msgs(f.msgs);
	}
	return *this;
}

void Folder::save(Message & msg)
{
	msgs.insert(&msg);
	msg.addFolder(this);
}

void Folder::remove(Message & msg)
{
	msgs.erase(&msg);
	msg.remFolder(this);
}

vector<Message*> Folder::messages()
{
	return vector<Message*>(msgs.begin(), msgs.end());
}

void Folder::debug_print()
{
	cerr << "Folder contains " << msgs.size() << "messages " << endl;
	int ctr = 1;
	for (Msg_iter beg = msgs.begin(); beg != msgs.end(); ++beg)
		cerr << "Message " << ctr++ << ":\n\t" << (*beg)->print_message << endl;
}

void Folder::copy_msgs(const set<Message*> &m)
{
	for (Msg_iter beg = m.begin(); beg != m.end(); ++beg)
		(*beg)->save(*this); // 遍历消息保存到文件夹中
}

void Folder::empty_msgs()
{
	Msg_iter it = msgs.begin();
	while (it != msgs.end())
	{
		Msg_iter next = it;
		++next;
		(*it)->remove(*this); // 遍历消息，删除消息与文件夹的关联
		it = next;
	}
}

```

## 智能指针
通过记录指针被复制次数，避免产生悬垂指针。

普通含有指针成员的类  HasPtr.h
```cpp
#pragma once
class HasPtr
{
public:
	HasPtr(int *p, int i) :ptr(p), val(i) {}

	int *get_ptr()const { return ptr; }
	int get_val() const { return val; }

	void set_ptr(int *p){ptr = p; }
	void set_val(int v) { val = v; }

	int get_ptr_val() const { return *ptr; }
	void set_ptr_val(int val)const { *ptr = val; }
private:
	int *ptr;
	int val;
};

```

main.cpp

```cpp
#include <iostream>
#include <sstream>
#include "HasPtr.h"

using namespace std;
int main()
{
	int obj = 0;
	HasPtr ptr1(&obj, 42);
	HasPtr ptr2(ptr1); // 默认的复制构造函数，将复制两个成员，指针将指向同一个对象

	ptr1.set_ptr_val(22);
	cout << ptr2.get_ptr_val() << endl; // 22
	return 0;
}

```

引入指针计数器 U_Ptr.h

```cpp
#pragma once
#include "HasPtr.h"
// 指针计数类
class U_Ptr
{
	friend class HasPtr;
	int *ip;
	size_t use;
	U_Ptr(int *p) :ip(p), use(1) {};
	~U_Ptr() { delete ip; };
};

```

HasPtr.h

```cpp
#pragma once
#include "U_Ptr.h"

class HasPtr
{
public:
	HasPtr(int *p, int i) :ptr(new U_Ptr(p)), val(i) {}
	HasPtr(const HasPtr &orig) :ptr(orig.ptr), val(orig.val) {}

	HasPtr& operator=(const HasPtr&);

	~HasPtr() { if (--ptr->use == 0) delete ptr; }
private:
	U_Ptr *ptr;
	int val;
};

HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
	++rhs.ptr->use;
	if (--ptr->use == 0)
		delete ptr;
	ptr = rhs.ptr;
	val = rhs.val;
	return *this;
}
```
我好懒