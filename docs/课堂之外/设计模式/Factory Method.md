# Factory Method

## 动机

- 在软件系统中，经常面临着创建对象的工作；由于需求的变化，需要创建的对象的具体类型经常变化。
- 如何应对这种变化？如何绕过常规的对象创建方法(new)，提供一种“封装机制”来避免客户程序和这种“具体对象创建工作”的紧耦合？

## 模式定义

定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使得一个类的实例化延迟（目的：解耦，手段：虚函数）到子类。 ——《设计模式》GoF

## SHOW ME THE CODE

`MainForm1.cpp`

```cpp
class MainForm : public Form
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;

public:
	void Button1_Click(){
        
		string filePath = txtFilePath->getText();
        int number = atoi(txtFileNumber->getText().c_str());
        
        FileSplitter * splitter //声明成具体的类，没有考虑未来的变化
            = new FileSplitter(filePath,number);
        splitter->split();

	}
};
```



以静态的视角看过去其实没什么问题，但是以未来的视角来看实际上存在问题。

我们可以用抽象类/接口的方法来将问题暴露出来，回忆设计原则：面向接口编程

针对接口编程，而不是针对实现编程

* **不将变量类型声明为某个特定的具体类，而是声明为某个接口**（不要绝对化了，这里主要指业务类）
* 客户程序无需获知对象的具体类型，需要知道对象所具有的接口
* 减少系统中各部分的依赖关系，从而实现“高内聚、松耦合”的类型设计方案
* 产业强盛的标志是接口标准化，接口标准化的本质是分工协作

写一个抽象类，然后考虑未来可能的扩展

```cpp
//抽象类
class ISplitter{
public:
    virtual void split()=0;
    virtual ~ISplitter(){}
};

class BinarySplitter : public ISplitter{
    
};

class TxtSplitter: public ISplitter{
    
};

class PictureSplitter: public ISplitter{
    
};

class VideoSplitter: public ISplitter{
    
};
```

那么`MainForm`变为

```cpp
class MainForm : public Form
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;

public:
	void Button1_Click(){
		//去掉了一些没啥影响的代码
		ISplitter * splitter=//依赖抽象了
            new BinarySplitter();//依赖具体类
        
        splitter->split();

	}
};
```

在接口抽象之后还有个问题，对象创建(new)依赖了具体类

```cpp
ISplitter * splitter=//依赖抽象了
    new BinarySplitter();//依赖具体类，违反了依赖倒置原则
```

根据依赖倒置（DIP）：编译时依赖

* 高层模块（稳定）不应该依赖于低层模块（变化），二者都应该依赖于抽象（稳定）
* 抽象（稳定）不应该依赖于实现细节（变化），实现细节应该依赖于抽象（稳定）



目前我们的思路大致如下

面向接口编程原则发现问题->接口抽象-->依赖倒置原则发现问题-->解决创建对象依赖具体类

考虑创建对象的方法

* 直接创建一个抽象类？不可以，C++不允许这么做
* 在堆上`new`一个对象？那就是上面代码暴露出的问题
* **用一个方法来返回一个对象**（实际上可以认为`new`也是一个方法）



```cpp
class SplitterFactory{
public:
	ISplitter* CreateSpliter(){
		return new BinarySplitter();
	}
}
```



```cpp
class MainForm : public Form
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;

public:
	void Button1_Click(){

		SplitterFactory factory;
        ISplitter * splitter=
            factory.CreateSplitter(); 
        splitter->split();

	}
};
```

这样问题解决了吗？有改善但没完全解决

注意目前的依赖指的是编译时依赖

`splitter`依赖`SplitterFactory`，`SplitterFactory`依赖`BinarySplitter`，最终`splitter`还是依赖`BinarySplitter`

那么考虑一下有没有运行时依赖的玩意：`virtual`函数



```cpp
//工厂基类
class SplitterFactory{
public:
	virtual ISplitter* CreateSpliter()=0;
    virtual ~SplitterFactory(){}//任何一个纯虚基类都需要一个虚的析构函数
}
```



```cpp
class MainForm : public Form
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;

public:
	void Button1_Click(){

		SplitterFactory* factory;//依赖交给未来
        ISplitter * splitter=
            factory->CreateSplitter(); //多态new，用指针的方式调用纯虚基类
        splitter->split();

	}
};
```

`virtual`函数实际上就是一种延迟的手法，我们将依赖交给了未来

`ISplitterFactory.cpp`

```cpp
//抽象类
class ISplitter{
public:
    virtual void split()=0;
    virtual ~ISplitter(){}
};


//工厂基类
class SplitterFactory{
public:
    virtual ISplitter* CreateSplitter()=0;
    virtual ~SplitterFactory(){}
};
```

`FileSplitter2.cpp`

```cpp
//具体类
class BinarySplitter : public ISplitter{
    
};

class TxtSplitter: public ISplitter{
    
};

class PictureSplitter: public ISplitter{
    
};

class VideoSplitter: public ISplitter{
    
};

//具体工厂
class BinarySplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new BinarySplitter();
    }
};

class TxtSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new TxtSplitter();
    }
};

class PictureSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new PictureSplitter();
    }
};

class VideoSplitterFactory: public SplitterFactory{
public:
    virtual ISplitter* CreateSplitter(){
        return new VideoSplitter();
    }
};
```



`MainForm2.cpp`

```cpp
class MainForm : public Form
{
    SplitterFactory*  factory;//工厂
public:
    
    MainForm(SplitterFactory*  factory){//未来从外界传递某一个具体的SplitterFactory
        this->factory=factory;
    }
    
	void Button1_Click(){

        
		ISplitter * splitter=
            factory->CreateSplitter(); //多态new
        
        splitter->split();

	}
};
```

现在`MainForm`解决了依赖具体类的问题

但是依赖具体类的问题真的完全被消灭了吗？可能没有。实际上我们的面向对象设计模式很多时候并不能消灭变化，但是我们可以把变化赶到某一个局部的地方。

在这个例子当中，我们真正解决的问题是变化在整个类设计的各个地方穿插，而非完全消灭变化

## 结构设计

红色稳定，蓝色变化

`MainForm`的创建现在依赖红色稳定的部分

![image-20220207145959190](https://s2.loli.net/2022/02/07/eN9qirZIdm5DnQR.png)

## 要点总结

- Factory Method模式用于隔离类对象的使用者和具体类型之间的耦合关系。面对一个经常变化的具体类型，紧耦合关系(new)会导致软件的脆弱。
- Factory Method模式通过面向对象的手法（多态），将所要创建的具体对象工作延迟到子类，从而实现一种扩展（而非更改）的策略，较好地解决了这种紧耦合关系。
- Factory Method模式解决“单个对象”的需求变化。**缺点在于要求创建方法/参数相同**。