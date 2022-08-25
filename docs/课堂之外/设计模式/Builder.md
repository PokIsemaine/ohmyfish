# Builder 模式

## 动机（Motivation）

- 在软件系统中，有时候面临着“一个复杂对象”的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这 个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。
- 如何应对这种变化？如何提供一种“封装机制”来隔离出“复杂对象的各个部分”的变化，从而保持系统中的“稳定构建算法”不随着需求改变而改变？

## 模式定义

将一个复杂对象的**构建与其表示相分离**，使得同样的构建过程(稳定)可以创建不同的表示(变化)。 ——《设计模式》GoF



## SHOW ME THE CODE

以建房子为例，这不是一件容易的事情。

房子的各个部分可能有着不一样的需求，但总体流程还是差不多的。

```cpp
class House{
public:
	void Init(){//整体流程固定
        this->BuildPart1();
        
        for(int i=0;i<4;i++){
            this->BuildPart2();
        }
        
        bool flag=this->BuildPart3();
        
        if(flag){
            this->BuildPart4();
        }
        this->BuildPart5();
    }
    
protected://各部分具体步骤不一样
	virtual void BuildPart1()=0;
    virtual void BuildPart2()=0;
    virtual void BuildPart3()=0;
    virtual void BuildPart4()=0;
    virtual void BuildPart5()=0;
	
};
```

这样子是不是有点像`Template Method`设计模式，不过`Builder`设计模式更侧重解决创建的问题

### 直接写成构造函数可行吗 

思考一下我们直接把`Init`写成下面这样的构造函数可以吗？

不可以，会报错

```cpp
class House{
public:
	void House(){
        this->BuildPart1();//静态绑定，不会调用子类
        
        for(int i=0;i<4;i++){
            this->BuildPart2();
        }
        
        bool flag=this->BuildPart3();
        
        if(flag){
            this->BuildPart4();
        }
        this->BuildPart5();
    }
    
protected://各部分具体步骤不一样
	virtual void BuildPart1()=0;//调用了这里的实现，但这个是纯虚函数没有实现
    virtual void BuildPart2()=0;
    virtual void BuildPart3()=0;
    virtual void BuildPart4()=0;
    virtual void BuildPart5()=0;
	
};
```

在 C++ 对象模型里，子类的构造函数先调用父类的构造函数

不建议在构造函数里调用虚函数。如果构造函数调用虚函数的化，基类的构造函数会调用基类的虚函数，但是派生类的构造函数还没完成。这相当于对象还没完成就要用对象里的东西，不合逻辑（JAVA 和 C# 不一样）



### Builder 模式初步

```cpp
class House{
public:
	void Init(){//整体流程固定
        this->BuildPart1();
        
        for(int i=0;i<4;i++){
            this->BuildPart2();
        }
        
        bool flag=this->BuildPart3();
        
        if(flag){
            this->BuildPart4();
        }
        this->BuildPart5();
    }
    
protected://各部分具体步骤不一样
	virtual void BuildPart1()=0;
    virtual void BuildPart2()=0;
    virtual void BuildPart3()=0;
    virtual void BuildPart4()=0;
    virtual void BuildPart5()=0;
	
};


class StoneHouse: public House{
    
};

class StoneHouseBuilder: public HouseBuilder{
protected:
    
    virtual void BuildPart1(){
        //pHouse->Part1 = ...;
    }
    virtual void BuildPart2(){
        
    }
    virtual void BuildPart3(){
        
    }
    virtual void BuildPart4(){
        
    }
    virtual void BuildPart5(){
        
    }
    
};

int main(){
    House* pHouse = new StoneHouse();
    pHouse->Init();
}
```

### 优化

`Builder`模式在某些场景下可能过于复杂，可能有些字段混合在一起。

> 不要有太“肥”的类---《重构》Martin Fowler

如果类的代码行为太多，构建过程很复杂，那么我们可以把构建过程单独提出来。

为了解决这个问题我们进一步拆分，把`Init()`拆分出去即`House`与`HouseBuilder`分离

`StoneHouse`继承`House`

`StoneHouseBuilder`继承`HouseBuilder`

再进一步，既然`HouseBuilder`已经稳定，我们还可以在对其进行拆分分离出一个`HouseDirector`

`HouseDirector`给什么样的`HouseBuilder`就会创建出什么样的`House`

```cpp

class House{
    //....
};

class HouseBuilder {
public:
    House* GetResult(){
        return pHouse;
    }
    virtual ~HouseBuilder(){}
protected:
    
    House* pHouse;
	virtual void BuildPart1()=0;
    virtual void BuildPart2()=0;
    virtual void BuildPart3()=0;
    virtual void BuildPart4()=0;
    virtual void BuildPart5()=0;
	
};

class StoneHouse: public House{
    
};

class StoneHouseBuilder: public HouseBuilder{
protected:
    
    virtual void BuildPart1(){
        //pHouse->Part1 = ...;
    }
    virtual void BuildPart2(){
        
    }
    virtual void BuildPart3(){
        
    }
    virtual void BuildPart4(){
        
    }
    virtual void BuildPart5(){
        
    }
    
};


class HouseDirector{
    
public:
    HouseBuilder* pHouseBuilder;
    
    HouseDirector(HouseBuilder* pHouseBuilder){
        this->pHouseBuilder=pHouseBuilder;
    }
    
    House* Construct(){
        
        pHouseBuilder->BuildPart1();
        
        for (int i = 0; i < 4; i++){
            pHouseBuilder->BuildPart2();
        }
        
        bool flag=pHouseBuilder->BuildPart3();
        
        if(flag){
            pHouseBuilder->BuildPart4();
        }
        
        pHouseBuilder->BuildPart5();
        
        return pHouseBuilder->GetResult();
    }
};


```



## 结构设计

红色稳定，蓝色变化

如果不是特别复杂`Director`和`Builder`可以合并

![image-20220210011740319](https://s2.loli.net/2022/02/10/zsK3tgR6dhNV8IH.png)

## 要点总结

- Builder 模式主要用于“分步骤构建一个复杂的对象”。在这其中“分步骤”是一个稳定的算法，而复杂对象的各个部分则经常变化。
- 变化点在哪里，封装哪里—— Builder 模式主要在于应对“复杂对象各个部分”的频繁需求变动。其缺点在于难以应对“分步骤构建算法”的需求变动。
- 在 Builder 模式中，要注意不同语言中构造器内调用虚函数的差别（C++(构造函数中不可以调用虚函数) vs. C#)。
