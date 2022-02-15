# Visitor

## 动机

* 在软件构建过程中，由于需求的改变，某些类层次结构中常常需 要增加新的行为（方法），**如果直接在基类中做这样的更改，将会 给子类带来很繁重的变更负担，甚至破坏原有设计。**
* 如何在不更改类层次结构的前提下，在运行时根据需要透明地为类层次结构上的各个类动态添加新的操作，从而避免上述问题？

## 模式定义

> 表示一个作用于某对象结构中的各元素的操作。使得可以在**不改变（稳定）各元素的类的前提**下定义（扩展）作用于这些元素的新操作（变化）。 
>
> <p align="right">——《设计模式》GoF</p>



## SHOW ME THE CODE

```cpp

#include <iostream>
using namespace std;

class Visitor;


class Element
{
public:
    virtual void Func1() = 0;
    
    virtual void Func2(int data)=0;
    virtual void Func3(int data)=0;
    //...
    
    virtual ~Element(){}
};

class ElementA : public Element
{
public:
    void Func1() override{
        //...
    }
    
    void Func2(int data) override{
        //...
    }
    
};

class ElementB : public Element
{
public:
    void Func1() override{
        //***
    }
    
    void Func2(int data) override {
        //***
    }
    
};

```

因为需求变化，我们需要改变基类的实现，那么我们往往还要在子类做相应修改，这么做似乎很常见。

但是如果我们现在已经完成了基本的部署分发，那么代价很高，我们要在每个子类里做修改。

这实际上违反了 开闭原则

开放封闭原则（OCP）：用扩展来适应需求的变化，用增加代替修改

* 对扩展开放，对更改封闭
* 类模块应该是可扩展的，但是不可修改

### 优化：双重分发

Vistor 模式的前提是能够预料到未来要加新的操作，但是具体加多少、加什么操作是不知道的。

Vistor 通过双重分发，使得在不改变各个元素类的前提下，扩展新的操作

```cpp
#include <iostream>
using namespace std;

class Visitor;


class Element
{
public:
    virtual void accept(Visitor& visitor) = 0; //第一次多态辨析

    virtual ~Element(){}
};

class ElementA : public Element
{
public:
    void accept(Visitor &visitor) override {
        visitor.visitElementA(*this);
    }
    

};

class ElementB : public Element
{
public:
    void accept(Visitor &visitor) override {
        visitor.visitElementB(*this); //第二次多态辨析
    }

};


class Visitor{
public:
    virtual void visitElementA(ElementA& element) = 0;
    virtual void visitElementB(ElementB& element) = 0;
    
    virtual ~Visitor(){}
};

//上面是预先设计，不会改变
//==================================
//未来可能情况

//扩展1
class Visitor1 : public Visitor{
public:
    void visitElementA(ElementA& element) override{
        cout << "Visitor1 is processing ElementA" << endl;
    }
        
    void visitElementB(ElementB& element) override{
        cout << "Visitor1 is processing ElementB" << endl;
    }
};
     
//扩展2
class Visitor2 : public Visitor{
public:
    void visitElementA(ElementA& element) override{
        cout << "Visitor2 is processing ElementA" << endl;
    }
    
    void visitElementB(ElementB& element) override{
        cout << "Visitor2 is processing ElementB" << endl;
    }
};
        
    

        
int main()
{
    Visitor2 visitor;
    ElementB elementB;
    elementB.accept(visitor);// double dispatch
    
    ElementA elementA;
    elementA.accept(visitor);

    
    return 0;
}
```



## 结构设计

一个设计模式的缺陷往往是它稳定的部分

![image-20220215155519727](https://s2.loli.net/2022/02/15/7Ef5VNZ4wKlYST2.png)

## 要点总结

- Visitor 模式通过所谓的双重分发（double dispatch）来实现在不更改(不添加新的操作-编译时) Element 类层次结构的前提下，在运行时 明地为类层次结构上的各个类动态添加新的操作(支持变化)。
- 所谓双重分发Visitor模式中间包括了两个多态分发：第一个为accept方法的多态辨析；第二个为visitElement方法的多态辨析。
- Visitor 模式的最大缺点在于扩展类层次结构(添加新的Element子类)，会导致Visitor类的改变， **因此Visitor模式适用于“ Element类层次结构稳定，而其中的操作却经常面临频繁改动”。**
	- 子类(数量等）确定这个前提很重要，这也是`Visitor`模式的一个很大缺陷，条件很严格
	- 注意稳定的是类的层次结构，类的操作是不稳定的，注意区分。如果层次结构和操作都稳定那就不用设计模式了，如果层次结构和操作都变化那就没有设计模式可以解决