# class with pointer members (String)

## 概述

* 以`String`类为例编写需要指针的类
* 复习的时候可以直接看，代码注释、要点、以及视频《复习 String类的实现过程》
* 要点基本在代码里有对应位置的注释，可以一起看便于回顾理解

## 代码

`string.hpp`

```cpp
#ifndef __MYSTRING__
#define __MYSTRING__

class String
{
public:                                 
   String(const char* cstr=0);	//构造函数 cotr                     
   String(const String& str);	//拷贝构造函数 copy ctor                   
   String& operator=(const String& str);	//拷贝赋值函数 copy op=
   ~String();	//析构函数 dtor                                    
   char* get_c_str() const { return m_data; }
private:
   char* m_data;
};

#include <cstring>

inline
String::String(const char* cstr)
{
   if (cstr) {
      m_data = new char[strlen(cstr)+1];//记住申请大小要 len+1
      strcpy(m_data, cstr);
   }
   else {   
      m_data = new char[1];//这样写的话可以统一为 new[]方便析构
      *m_data = '\0';
   }
}

inline
String::~String()
{
   delete[] m_data;//array new 一定要搭配 array delete
}

inline
String& String::operator=(const String& str)
{
   if (this == &str)//检测自我赋值
      return *this;

   delete[] m_data;//清空原来的
   m_data = new char[ strlen(str.m_data) + 1 ];//直接取另一个 object 的 private data.(兄弟之间互为 friend)
   strcpy(m_data, str.m_data);//右边的内容传给左边
   return *this;
}

inline
String::String(const String& str)
{
   m_data = new char[ strlen(str.m_data) + 1 ];
   strcpy(m_data, str.m_data);
}

#include <iostream>
using namespace std;

ostream& operator<<(ostream& os, const String& str)
{
   os << str.get_c_str();
   return os;
}

#endif

```

`string_test.cpp`

```cpp
#include "string.h"
#include <iostream>

using namespace std;

int main()
{
  String s1("hello"); 
  String s2("world");
    
  String s3(s2);
  cout << s3 << endl;
  
  s3 = s1;
  cout << s3 << endl;     
  cout << s2 << endl;  
  cout << s1 << endl;      
}

```



## 要点

* `class with pointer members `必須有 `copy ctor` 和 `copy op=`
	* 使用 `default copy ctor` 或 `default op=` 就会造成`memory leak`和多个指针指向同一块内存区域

![image-20220202235927934](https://s2.loli.net/2022/02/03/komLUZMfQGwuFcT.png)

* 拷贝赋值函数要检测自我赋值

* 为了和标准库实现保持一致，用于输出的`operator<<`重载一般写成非成员函数

* `new`：`先分配 memory`, 再調用` ctor`

	* `operator new`分配内存（内部调用`malloc`）
	* 转型
	* 调用构造函数

	![image-20220203001043798](https://s2.loli.net/2022/02/03/CaqOBv65YMHkmgS.png)

* `delete`：先調用` dtor`, 再釋放 `memory`

	* 调用析构函数(处理自己动态申请的部分)
	* `operator delete`释放内存（内部调用`free`）

	![image-20220203001016562](https://s2.loli.net/2022/02/03/WdAhVtmb8UMsfY4.png)

* `array new` 一定要搭配 `array delete`