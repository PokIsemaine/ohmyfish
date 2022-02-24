# Object Model

先去“三大基本关系”那个文档复习一下类之间的三种关系以及相关的构造和析构函数

## 关于 vptr 和 vtbl

![image-20220223220332562](https://s2.loli.net/2022/02/23/5Qglf4ATHNROqMm.png)

* 只要类里面有虚函数，类里就会有一个指针（无论有多少个虚函数），这个指针就是虚指针
* 父类有虚函数，子类也一定有。继承会把数据和函数的调用权都继承下来
* 当我们用指针调用的时候会发生动态绑定，首先通过指针找到`vptr`，然后找到`vtbl`，最后调用要求的函数

我们可以用C来模拟动态绑定的路线

```cpp
//n是虚函数在虚函数表中的第几个，编译器按代码顺序放
(*(p->vptr)[n])(p);
(*p->vptr[n])(p);
```



以一个画板程序为例子，我们可以在容器里放指针。然后利用继承+虚函数实现一个多态，调用各自的`draw`

这比`if-else`更好一些，具体好在哪里可以学一下设计模式

![image-20220223225922768](https://s2.loli.net/2022/02/23/5bGdEn2VNUChXym.png)



C++ 编译器看到一个函数调用有两种套路

* 静态绑定：call xxx，一定调用到某个地址
* 动态绑定：如果是通过指针调用虚函数并且该指针向上转型（upcast，比如指针是动物，然后new一只猪），那么编译器就会把调用动作编译成类似C语言版本来模拟调用路线。调用哪个地址要看指针指向什么

## 关于 this

![image-20220223231429238](https://s2.loli.net/2022/02/23/M6oD7tR1LNcqUHF.png)

可以先去看一下设计模式的`Template Method`

框架里把一些固定的、确定的步骤写好了，但是有一些操作还不确定要看应用具体怎么做

这时候我们就可以利用虚函数实现一个延后，把具体操作的实现延后到调用的时候，谁调用谁负责实现

然后再来看看`this`，我们可以认为`this`是调用者的地址，是一个指针

```cpp
CMyDoc myDoc;
myDoc.OnFileOpen();//成员函数隐藏了一个this，注意啊这里还是对象调用而且OnFileOpen自己不是虚函数，所以这里是静态调用
myDoc.OnFileOpen(this);
myDoc.OnFileOpen(&myDoc);
myDoc.CDocument::OnFileOpen(&myDoc);//子类可以用父类的函数
//接下来就会对Serialize()进行动态绑定
this->Serialize();//this是子类对象
(*(this->vptr)[n])(this);//虚函数b
```



## 关于 Dynamic Binding

来看看汇编视角下的静态绑定：`call xxx`

![image-20220223233511050](https://s2.loli.net/2022/02/23/4ALMk5KSxYyI3F8.png)



汇编视角下的动态绑定

![image-20220223234506350](https://s2.loli.net/2022/02/23/DKgIbJ4w3iFtGRV.png)
