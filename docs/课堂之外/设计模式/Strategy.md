# Strategy

## 动机

- 在软件构建过程中，某些对象使用的算法可能多种多样，经常改变，如果将这些算法都编码到对象中，将会使对象变得异常复杂；而且有时候支持不使用的算法也是一个性能负担。
- 如何在运行时根据需要透明地更改对象的算法？将算法与对象本身解耦，从而避免上述问题？

## 模式定义

> **定义一系列算法，把它们一个个封装起来**，并且使它们可互相替换（变化）。该模式使得算法可独立于使用它的客户程序(稳定)而变化（扩展，子类化）。 ——《设计模式》 GoF



## SHOW ME THE CODE

`strategy1.cpp`

```cpp
enum TaxBase {
	CN_Tax,
	US_Tax,
	DE_Tax,
	FR_Tax      
};

class SalesOrder{
    TaxBase tax;
public:
    double CalculateTax(){
        //...
        
        if (tax == CN_Tax){
            //CN***********
        }
        else if (tax == US_Tax){
            //US***********
        }
        else if (tax == DE_Tax){
            //DE***********
        }
		else if (tax == FR_Tax){//这样更改违反开闭原则
			//...
		}
        //....
     }
    
};
```

### 用时间轴的概念发现问题

上述代码在静态视角下没啥问题，但是当我们用时间轴的概念动态的看待时，问题就暴露出来了

如果增加一个货币类型，就要修改`if-else`，这样实际上违背了开闭原则

开放封闭原则（OCP）：用扩展来适应需求的变化，用增加代替修改

* 对扩展开放，对更改封闭
* 类模块应该是可扩展的，但是不可修改

### 优化：使用抽象基类和多态指针



`strategy2.cpp`

```cpp
class TaxStrategy{
public:
    virtual double Calculate(const Context& context)=0;
    virtual ~TaxStrategy(){}
};


class CNTax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};

class USTax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};

class DETax : public TaxStrategy{
public:
    virtual double Calculate(const Context& context){
        //***********
    }
};



//扩展
//*********************************
//变化：新增一个法国的货币，但SalesOrder不用改（得到了复用性）
class FRTax : public TaxStrategy{
public:
	virtual double Calculate(const Context& context){
		//.........
	}
};


class SalesOrder{
private:
    TaxStrategy* strategy;//多态指针

public:
    
    SalesOrder(StrategyFactory* strategyFactory){
        this->strategy = strategyFactory->NewStrategy();
    }
    ~SalesOrder(){
        delete this->strategy;
    }

    public double CalculateTax(){
        //...
        Context context();
        
        double val = 
            strategy->Calculate(context); //śŕĚŹľ÷ÓĂ
        //...
    }
    
};
```



## 结构设计

![image-20220215165554359](https://s2.loli.net/2022/02/15/OUcjkdMWE8eKB41.png)



## 要点总结

- Strategy及其子类为组件提供了一系列可重用的算法，从而可以使得类型在**运行时（多态调用）**方便地根据需要在各个算法之间进行切换。
- Strategy模式提供了用条件判断语句以外的另一种选择，消除条件判断语句，就是在解耦合。**含有许多条件判断语句的代码通常都需要Strategy模式。**（当然这个不是绝对的）
- 如果Strategy对象没有实例变量，那么各个上下文可以共享同一个Strategy对象，从而节省对象开销。