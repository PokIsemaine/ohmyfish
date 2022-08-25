# Prototype

## 动机

- 在软件系统中，经常面临这**“某些结构复杂的对象”**的创建工作；由于需求的变化，这些对象经常面临着剧烈的变化，但是它们却拥有比较稳定一致的接口。
- 如何应对这种变化？如何向“客户程序(使用这些对象的程序)”隔离出“这些易变对象”，从而使得依赖这些”易变对象“的客户程序不随着需求改变而改变。

## 模式定义

使用原型实例指定创建对象的种类，然后通过拷贝这些原型来创建新的对象。 ——《设计模式》GoF

## SHOW ME THE CODE

```cpp
//抽象类
class ISplitter{
public:
    virtual void split()=0;
    virtual ISplitter* clone()=0; //通过克隆自己来创建对象
    
    virtual ~ISplitter(){}

};
```

和`Factory Method`等设计模式不同，`Prototype`原型设计模式采用了另外一种创建对象的方法`clone`

提到`clone`我们可以想到`C++`当中的拷贝构造函数，原型模式利用拷贝构造函数来进行对自己的克隆，从而创建对象

```cpp
//具体类
class BinarySplitter : public ISplitter{
public:
    virtual ISplitter* clone(){
        return new BinarySplitter(*this);//拷贝构造函数
    }
};

class TxtSplitter: public ISplitter{
public:
    virtual ISplitter* clone(){
        return new TxtSplitter(*this);
    }
};

class PictureSplitter: public ISplitter{
public:
    virtual ISplitter* clone(){
        return new PictureSplitter(*this);
    }
};

class VideoSplitter: public ISplitter{
public:
    virtual ISplitter* clone(){
        return new VideoSplitter(*this);
    }
};

```

`Client.cpp`

```cpp
class MainForm : public Form
{
    ISplitter*  prototype;//原型对象

public:
    
    MainForm(ISplitter*  prototype){
        this->prototype=prototype;
    }
    
	void Button1_Click(){

		ISplitter * splitter=
            prototype->clone(); //克隆原型
        
        splitter->split();
        
        

	}
};

```

需要注意的是，原型对象并不用于使用。我们需要通过克隆来的新对象进行操作，原型对象只能拿来克隆。

## 结构设计

![image-20220208214219082](https://s2.loli.net/2022/02/08/iw4TaC2LsIdM8lV.png)

## 要点总结

- Prototype 模式同样用于隔离对象的使用者和具体类型(易变类)之间的耦合关系，**它同样要求这些“易变类”拥有稳定的接口**。
- Prototype 模式对于“如何创建易变类的实体对象“采用”原型克隆“的方法来做， 它使得我们可以非常灵活地动态创建”拥有某些稳定接口“的新对象——所需工作仅仅是注册一个新类的对象(即原型)， 然后在任何需要的地方 Clone。
- Prototype 模式中的 Clone 方法可以利用某些框架中的序列化来实现深拷贝。（有的语言没有拷贝构造函数）
- `Prototype`适合那种刚开始状态不太好，然后等对象状态相对比较好的时候传入作为原型，以后使用的时候直接克隆这个原型即可，这适合那些结构复杂的对象。这里的复杂其实是比较主观的感受，一般来说你如果使用工厂方法创建的时候感觉很麻烦，那么可以考虑使用原型模式。