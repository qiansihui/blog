---
title: C++  11 泛型算法
date: 2018-01-26 14:31:50
tags: cpp
---
# 插入迭代器


```cpp
#include <iostream>
#include <sstream>
#include <fstream>
#include <set>
#include <vector>
#include <list>
#include <map>
#include <string>
#include <iterator>     // std::front_inserter

using namespace std;

int main()
{
	// 插入迭代器
	list<int> ilist;
	vector<int> ivec;
	ivec.push_back(1);
	ivec.push_back(42);
	ivec.push_back(3);
	// 利用插入迭代器拷贝 , 将ivec元素依次插入到ilist前面
	copy(ivec.begin(), ivec.end(), front_inserter(ilist)); 
	// ilist输出为 3， 42 ，1
	for (list<int>::iterator i = ilist.begin(); i != ilist.end(); ++i)
		cout << "index i : " << *i << endl;
	
	// 从ivec找到元素42的位置，拷贝42与之后的元素 42,3
	ilist.clear();
	vector<int>::iterator it = find(ivec.begin(), ivec.end(), 42);
	copy(it, ivec.end(), front_inserter(ilist)); // 拷贝指定范围
	// ilist输出为42 ，3
	for (list<int>::iterator i = ilist.begin(); i != ilist.end(); ++i)
		cout << "index i : " << *i << endl;

	return 0;
}

```


# 流迭代器


```cpp
#include <iostream>
#include <sstream>
#include <fstream>
#include <set>
#include <vector>
#include <list>
#include <map>
#include <string>
#include <iterator>     // std::front_inserter
#include "Sales_item.h"

using namespace std;

int main()
{
	// 流迭代器定义
	cout << "int类型流迭代器，输入数字：" << endl;

	istream_iterator<int> cin_it(cin); // 从输入流中读取int值
	istream_iterator<int> eof; // 输入流终止符

	while (cin_it != eof)
	{
		int input = *cin_it++;
		if (input == 0)break;

		cout << "input:" << input << endl; // 会显示输入的上一个？
	}
	cin.clear();
	cout << "对象类型流迭代器，输入字符串 整数 浮点数：" << endl;
	istream_iterator<Sales_item> item_iter(cin), item_eof;
	Sales_item sum;
	sum = *item_iter++;
	while (item_iter != item_eof)
	{
		if (item_iter -> same_isbn(sum))
		{
			sum = sum + *item_iter;
		}
		else
		{
			cout << sum << endl;
			sum = *item_iter;
		}
		++item_iter;
	}
	cout << sum << endl;
	return 0;
}

```

流迭代器 与算法


```cpp
#include <iostream>
#include <sstream>
#include <fstream>
#include <set>
#include <vector>
#include <list>
#include <map>
#include <string>
#include <iterator>     // std::front_inserter
#include <algorithm>

using namespace std;

int main()
{
	// 流迭代器 与算法
	cout << "int类型流迭代器，输入数字：" << endl;

	istream_iterator<int> cin_it(cin); // 从输入流中读取int值
	istream_iterator<int> eof; // 输入流终止符

	vector<int> vec(cin_it, eof); //将输入数字转入容器
	sort(vec.begin(), vec.end());
	ostream_iterator<int> output(cout, " "); // 以空格为输出元素分隔符
	unique_copy(vec.begin(), vec.end(), output); // 容器去重拷贝到输出流

	return 0;
}

```


# 反向迭代器


```cpp
#include <iostream>
#include <sstream>
#include <fstream>
#include <set>
#include <vector>
#include <list>
#include <map>
#include <string>
#include <iterator>     // std::front_inserter
#include <algorithm>

using namespace std;

int main()
{
	cout << "反向迭代器：" << endl;
	vector<int> ivec;
	ivec.push_back(1);
	ivec.push_back(2);
	ivec.push_back(3);

	ostream_iterator<int> output(cout,",");
	copy(ivec.begin(), ivec.end(), output); // 打印数组不错哎

	cout << "用反向迭代器逆序输出: " << endl;
	for (vector<int>::reverse_iterator it = ivec.rbegin();
		it != ivec.rend(); ++it)
	{
		cout << *it << ",";
	}
	cout << "用反向迭代器拷贝到输出流：" << endl;
	copy(ivec.rbegin(), ivec.rend(), output);

	return 0;
}

```
