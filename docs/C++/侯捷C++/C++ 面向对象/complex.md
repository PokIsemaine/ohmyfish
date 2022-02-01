# class without pointer members (Complex)

## 概述

* 以`complex`类为例编写无需指针的类
* 复习的时候可以直接看，代码注释、要点、以及视频《复习 Complex类的实现过程》
* 要点基本在代码里有对应位置的注释，可以一起看便于回顾理解

## 代码

`complex.hpp`

```cpp
#ifndef __MYCOMPLEX__//guard 头文件防卫式声明
#define __MYCOMPLEX__

// orward declarations 前置声明
class complex; 
complex&
  __doapl (complex* ths, const complex& r);
complex&
  __doami (complex* ths, const complex& r);
complex&
  __doaml (complex* ths, const complex& r);

// 类声明
class complex//class head
{
// class body
public:// 函数如果要被外部调用放public，如果只是内部处理那么可以放private

// 构造函数应该使用initialization list(初值列, 初始列)来初始化
// 不建议在大括号里通过赋值的方式实现初始化
  complex (double r = 0, double i = 0): re (r), im (i) { }

  complex& operator += (const complex&);// 函数若在 class body 內定義完成，便自动成为 inline 候选人
  complex& operator -= (const complex&);// 可以把函数都声明为 inline ，但要记住这只是对编译器的建议
  complex& operator *= (const complex&);// 尽量用 pass by reference(to const) 代替 pass by value
  complex& operator /= (const complex&);// 如果可以，尽量使用 return by reference(to const) 代替 return by value
  double real () const { return re; }// const member functions 常量成员函数，不会改变数据的函数加 const
  double imag () const { return im; }// 如果不加const,那么 const 对象就不能调用非常量函数成员

private:// 数据应当放在private里，不应该直接拿
  double re, im;

  friend complex& __doapl (complex *, const complex&);// 友元可以自由取得 private 成员，但是一定程度上会破坏封装
  friend complex& __doami (complex *, const complex&);
  friend complex& __doaml (complex *, const complex&);
};


// class definition 类定义

inline complex&
__doapl (complex* ths, const complex& r)//没带 class 名称是非成员函数，没有 this
{
  ths->re += r.re;
  ths->im += r.im;
  return *ths;//传递者无需知道接收者是以 reference 形式接收，返回的是 object ,但是是不是 reference 接收无所谓
}
 
inline complex&
complex::operator += (const complex& r)//成员函数实际上隐含了一个参数 this 来表示调用者，但不能写出来
{
  return __doapl (this, r);
}

inline complex&
__doami (complex* ths, const complex& r)
{
  ths->re -= r.re;
  ths->im -= r.im;
  return *ths;
}
 
inline complex&
complex::operator -= (const complex& r)
{
  return __doami (this, r);
}
 
inline complex&
__doaml (complex* ths, const complex& r)
{
  double f = ths->re * r.re - ths->im * r.im;
  ths->im = ths->re * r.im + ths->im * r.re;
  ths->re = f;
  return *ths;
}

inline complex&
complex::operator *= (const complex& r)
{
  return __doaml (this, r);
}
 
inline double
imag (const complex& x)
{
  return x.imag ();
}

inline double
real (const complex& x)
{
  return x.real ();
}

inline complex//如果必须要在函数里创建一个地方来存放结果（local 对象），那么不能返回引用
operator + (const complex& x, const complex& y)
{
  return complex (real (x) + real (y), imag (x) + imag (y));
}

inline complex
operator + (const complex& x, double y)
{
  return complex (real (x) + y, imag (x));
}

inline complex
operator + (double x, const complex& y)
{
  return complex (x + real (y), imag (y));
}

inline complex
operator - (const complex& x, const complex& y)
{
  return complex (real (x) - real (y), imag (x) - imag (y));
}

inline complex
operator - (const complex& x, double y)
{
  return complex (real (x) - y, imag (x));
}

inline complex
operator - (double x, const complex& y)
{
  return complex (x - real (y), - imag (y));
}

inline complex
operator * (const complex& x, const complex& y)
{
  return complex (real (x) * real (y) - imag (x) * imag (y),
			   real (x) * imag (y) + imag (x) * real (y));
}

inline complex
operator * (const complex& x, double y)
{
  return complex (real (x) * y, imag (x) * y);
}

inline complex
operator * (double x, const complex& y)
{
  return complex (x * real (y), x * imag (y));
}

inline complex
operator / (const complex& x, double y)
{
  return complex (real (x) / y, imag (x) / y);
}

inline complex
operator + (const complex& x)
{
  return x;
}

inline complex
operator - (const complex& x)
{
  return complex (-real (x), -imag (x));
}

inline bool
operator == (const complex& x, const complex& y)
{
  return real (x) == real (y) && imag (x) == imag (y);
}

inline bool
operator == (const complex& x, double y)
{
  return real (x) == y && imag (x) == 0;
}

inline bool
operator == (double x, const complex& y)
{
  return x == real (y) && imag (y) == 0;
}

inline bool
operator != (const complex& x, const complex& y)
{
  return real (x) != real (y) || imag (x) != imag (y);
}

inline bool
operator != (const complex& x, double y)
{
  return real (x) != y || imag (x) != 0;
}

inline bool
operator != (double x, const complex& y)
{
  return x != real (y) || imag (y) != 0;
}

#include <cmath>

inline complex
polar (double r, double t)
{
  return complex (r * cos (t), r * sin (t));
}

inline complex
conj (const complex& x) 
{
  return complex (real (x), -imag (x));
}

inline double
norm (const complex& x)
{
  return real (x) * real (x) + imag (x) * imag (x);
}

#endif   //__MYCOMPLEX__





```

`complex.cpp`

```cpp
#include <iostream>
#include "complex.hpp"

using namespace std;

ostream&//考虑cout << c1 << conj(c1);这种情况 cout << c1得到的结果还要能够接收 conj(c1),所以不能返回 value
operator << (ostream& os, const complex& x)//os 会被改变所以不用 const
{
  return os << '(' << real (x) << ',' << imag (x) << ')';
}

int main()
{
  complex c1(2, 1);
  complex c2(4, 0);

  cout << c1 << endl;
  cout << c2 << endl;
  
  cout << c1+c2 << endl;
  cout << c1-c2 << endl;
  cout << c1*c2 << endl;
  cout << c1 / 2 << endl;
  
  cout << conj(c1) << endl;
  cout << norm(c1) << endl;
  cout << polar(10,4) << endl;
  
  cout << (c1 += c2) << endl;
  
  cout << (c1 == c2) << endl;
  cout << (c1 != c2) << endl;
  cout << +c2 << endl;
  cout << -c2 << endl;
  
  cout << (c2 - 2) << endl;
  cout << (5 + c2) << endl;
  
  return 0;
}
```



## 要点

* 头文件采用防卫式声明

* 函数若在 `class body`內定义完成，便自动成为`inline`候选人

* 可以把函数都声明为`inline`，但要记住这只是对编译器的建议

* 数据应当放在`private`里，不应该直接拿；函数如果要被外部调用放`public`，如果只是内部处理那么可以放`private`

* 构造函数应该使用`initialization list`(初值列, 初始列)来初始化，不建议在大括号里通过赋值的方式实现初始化

* `C++`允许同名函数重载，在编译器视角下函数名称实际上不同，但要注意默认实参的问题

* 在`Singleton`设计模式（单例模式）下可能会把构造函数放在`private`里，外界只允许用一份对象并且由类自己来创建

* 不会改变数据的成员函数加`const`，如果不加那么常量对象就不能调用非常量函数成员

* 尽量用`pass by reference(to const)`代替`pass by value`（当然这是个大方向，如果深究的话需要看具体情况）

* **如果可以**，尽量使用 `return by reference(to const)` 代替 `return by value`

	* 如果必须要在函数里创建一个地方来存放结果（local 对象、临时对象），那么不能返回引用
	* 传递者无需知道接收者是以`reference`形式接收，返回的是 `object` ,但是是不是 `reference` 接收无所谓
	* 要考虑连续使用操作符（特别是赋值和输入输出相关的）的情况

* 友元可以自由取得 `private` 成员，但是一定程度上会破坏封装

* 相同的`class`的各个`objects`互为友元

* 成员函数实际上隐含了一个参数 `this`来表示调用者，可以直接用，但不能在参数列表里写出来

* 没带 `class` 名称是非成员函数，没有`this`

	