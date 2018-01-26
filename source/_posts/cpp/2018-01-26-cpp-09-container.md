---
title: C++  09 顺序容器
date: 2018-01-26 14:31:06
tags: cpp
---

## 顺序容器初始化


```cpp
#include <iostream>
#include <vector>
#include <list>
#include <deque>
#include <string>
using std::vector; using std::list; using std::deque;
using std::string; using std::cin; using std::cout; using std::endl;

int main()
{
	// 顺序容器初始化，默认构造函数
	vector<string> svec;
	list<int> ilist;
	deque<string> items;
	// 容器元素初始化
	vector<int> ivec;
	vector<int> ivec2(ivec); // 将一个容器初始化为另一个容器副本
	list<string> slist(svec.begin(), svec.end()); // 初始化为一段元素的副本
	vector<string>::iterator mid = svec.begin() + svec.size() / 2;
	deque<string> front(svec.begin(), mid); // 初始化为一半
	deque<string> back(mid, svec.end());	// 初始化为后一半

	char *words[] = { "red","pink","green" };
	size_t words_size = sizeof(words) / sizeof(char *);
	list<string> words2(words, words + words_size); // string 数据 转list

	const list<int>::size_type list_size = 42; // 分配并初始化指定数目元素
	list<string> slist(list_size, "oh"); // 42 string each is oh

	list<int> ilist2(list_size);  // 指定大小

	return 0;
}

```

## 容器的增删改查


```cpp
#include <iostream>
#include <vector>
#include <list>
#include <deque>
#include <string>
using std::vector; using std::list; using std::deque;
using std::string; using std::cin; using std::cout; using std::endl;

int main()
{

	vector<int> vec(2, 5);
	// 顺序容器迭代器都支持自增自减、相等判断、解引用操作
	// vector 和 deque 支持额外操作 加减大小判断
	vector<int>::iterator iter = vec.begin() + vec.size() / 2;
	list<int> ilist;
	ilist.push_back(1);
	ilist.push_back(2);		// 尾插入
	ilist.push_front(3);  // 前插入 3,1,2
	list<int>::const_iterator it = ilist.begin();
	it = ilist.insert(it, 4); // 指定迭代器前插入一个 4,3,1,2
	ilist.insert(it, 3, 3); // 指定迭代器前插入n个value 3,3,3,4,3,1,2
	ilist.insert(it, vec.begin(), vec.end()); //指定迭代器前插入一段 3,3,3,5,5,4,3,1,2

	for (list<int>::const_iterator iter = ilist.begin(); iter != ilist.end(); iter++) // 正序输出list
	{
		cout << *iter << ",";
	}
	cout << endl << "逆序：" << endl;
	for (list<int>::const_reverse_iterator iter = ilist.rbegin(); iter != ilist.rend(); iter++) // 逆序输出list
	{
		cout << *iter << ",";
	}
	cout << endl;
	// 容器大小操作
	cout << "list 元素个数：" << ilist.size() << endl;
	cout << "list maxsize :" << ilist.max_size() << endl;
	cout << "list is empty:" << ilist.empty() << endl;

	list<int> ilist2(10, 42);
	ilist2.resize(15);	// add 5 elemet of value 0 back of list
	ilist2.resize(25, -1); // add 10 elemets of value -1 back of list
	ilist2.resize(5); // erases 20 elements from the back of list

	// 访问元素
	vector<string> svec;
	svec.push_back("a");
	svec.push_back("b");
	vector<string>::reference val1 = svec.front();
	vector<string>::reference val2 = svec.back();
	cout << "first element : " << val1 << endl;
	cout << "last  elememt : " << val2 << endl;
	cout << "second element : " << svec[1] << endl;

	// 删除元素
	deque<int> id(5,2);
	while (id.empty())
	{
		cout << "id's element : " << id.front() << endl;
		id.pop_front();
	}

	list<string> slist(10, "a");
	slist.back() = "b";
	string serachValue("b");
	list<string>::iterator it2 = std::find(slist.begin(), slist.end(), serachValue);
	if (it2 != slist.end()) slist.erase(it2); // 找到并删除
	for (list<string>::const_iterator iter = slist.begin(); iter != slist.end(); iter++) // 正序输出list
	{
		cout << *iter << ",";
	}
	slist.clear(); // clear list

	list<string> slist2(2, "a");
	list<string> slist3(2, "b");
	slist2 = slist3;	// 清空slist2 , 将slist3元素赋值给slist2
	slist2.swap(slist3); // 交换内容
	slist2.assign(slist3.begin(), slist3.end()); // 使用一段内容将slist2重新设置 
	slist2.assign(2, "c");				// 使用2个c重新设置slist2
	return 0;
}

```

## string 对象与容器


```cpp
#include <iostream>
#include <vector>
#include <list>
#include <deque>
#include <string>
using std::vector; using std::list; using std::deque;
using std::string; using std::cin; using std::cout; using std::endl;

int main()
{
	// 构造string对象
	string s1;
	string s2(5, 'a');
	string s3(s2);
	string s4(s3.begin(), s3.begin() + s3.size() / 2);
	// 用紫川初始化
	s1 = "nioh";
	string s6(s1, 2); // oh
	string s7(s1, 0, 2); // ni
	string s8(s1, 0, 8); // nioh

	s2.erase(s2.size() - 5, 5); // 删除最后5个元素
	s2.insert(s2.size(), 5, '!');//最后添加5个!

	char *cp = "nihaoa! xiaohui";
	string s;
	s.assign(cp, 7);// nihaoa!
	s.insert(s.size(), cp + 7); // s尾添加 cp[7]之后的元素
	cout << "after insert s: " << s << endl;

	s = "hello world";
	string ss2 = s.substr(6, 5); // world
	cout << "ss2: " << ss2 << endl;
	cout << "ss2.append('!') = " << ss2.append("!") << endl;
	cout << "s.replace(6,5,'xiaohui') = " << s.replace(6, 5, "xiaohui") << endl;

	// 查找
	string name("Xiaohui");
	string::size_type pos1 = name.find("hui"); // 4

	string numerics("0123456789");
	string name2("r2d2");
	string::size_type pos = name2.find_first_of(numerics);
	cout << "fount at index : " << pos << " element is " << name2[pos] << endl; // 匹配点1,2

	string name3("49p123");
	pos = name3.find_first_not_of(numerics);
	cout << "find at index : " << pos << "element is " << name3[pos] << endl;  // 不匹配点 2,p

	string sp1("aab");
	string sp2("ab");
	cout << "sp1.compare(sp2) = " << sp1.compare(sp2) << endl;
	cout << "sp1.compare(1,2,sp2) = " << sp1.compare(1, 2, sp2) << endl;
	cout << " sp1.compare(1, 2, sp2, 0, 2) = " << sp1.compare(1, 2, sp2, 0, 2) << endl;
	return 0;
}

```

## 容器适配器 stack 、 quene


```cpp
#include <iostream>
#include <vector>
#include <list>
#include <stack>
#include <queue>
#include <deque>
#include <string>
using std::vector; using std::list; using std::deque; using std::stack;
using std::string; using std::cin; using std::cout; using std::endl;

int main()
{

	deque<int> deq;
	stack<int> stk(deq);
	stack<string, vector<string> > str_stk;  

	const stack<int>::size_type stk_size = 10;
	stack<int> intStack;
	int ix = 0;
	while (intStack.size() != stk_size)
		intStack.push(ix++);

	int error_cnt = 0;
	while (intStack.empty() == false) {
		int value = intStack.top();
		if (value != --ix) {
			std::cerr << "oops expected " << ix << " received " << value << endl;
			++error_cnt;
		}
		intStack.pop();
	}
	cout << "out program ran with " << error_cnt << " errors" << endl;
	return 0;
}

```
