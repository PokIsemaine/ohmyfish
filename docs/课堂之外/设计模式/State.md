# State

## 动机

* 在软件构建过程中，**某些对象的状态如果改变，其行为也会随之而发生变化**，比如文档处于只读状态，其支持的行为和读写状态支持的行为就可能完全不同。 
* 如何在运行时根据对象的状态来透明地更改对象的行为？而不会 为对象操作和状态转化之间引入紧耦合？

## 模式定义

> 允许一个对象在其内部状态改变时改变它的行为。从而使对象看起来似乎修改了其行为。 ——《设计模式》GoF

## SHOW ME THE CODE

以一个网络应用为例

`state1.cpp`

```cpp
enum NetworkState
{
    Network_Open,
    Network_Close,
    Network_Connect,
};

class NetworkProcessor{
    
    NetworkState state;

public:
    
    void Operation1(){
        if (state == Network_Open){

            //**********
            state = Network_Close;
        }
        else if (state == Network_Close){

            //..........
            state = Network_Connect;
        }
        else if (state == Network_Connect){

            //$$$$$$$$$$
            state = Network_Open;
        }
    }

    public void Operation2(){

        if (state == Network_Open){
            
            //**********
            state = Network_Connect;
        }
        else if (state == Network_Close){

            //.....
            state = Network_Open;
        }
        else if (state == Network_Connect){

            //$$$$$$$$$$
            state = Network_Close;
        }
    
    }

    public void Operation3(){

    }
};
```

上面代码有什么问题呢？回顾一下`Strategy`策略模式，我们可以想到`if-else`有可能是一种`Bad Smell`

用时间轴的思维考虑状态未来的变化，`NetworkState`未来是否有可能有扩展

```cpp
enum NetworkState
{
    Network_Open,
    Network_Close,
    Network_Connect,
};
```

那么如果还是通过`if-else`的方式来处理的话，实际上违背了开闭原则，这会导致你在很多地方不断去改

开放封闭原则（OCP）：用扩展来适应需求的变化，用增加代替修改

* 对扩展开放，对更改封闭
* 类模块应该是可扩展的，但是不可修改

我们可以参考`Strategy`模式的经验进行优化

### 优化1：提取抽象基类

把枚举类型转化为抽象基类，操作采用虚函数多态，然后塞一个指针

```cpp
class NetworkState{

public:
    NetworkState* pNext;
    virtual void Operation1()=0;
    virtual void Operation2()=0;
    virtual void Operation3()=0;

    virtual ~NetworkState(){}
};
```

### 优化2：编写OpenState并使用单例模式

因为是网络状态，一般只要一个对象，所以我们可以使用单例模式

并且状态相关的操作得到的不再是枚举类型而是一个对象

```cpp
class OpenState :public NetworkState{
    
    static NetworkState* m_instance;//这里实际上用了单例模式
public:
    static NetworkState* getInstance(){
        if (m_instance == nullptr) {
            m_instance = new OpenState();
        }
        return m_instance;
    }

    void Operation1(){
        
        //**********
        pNext = CloseState::getInstance();//改变为对应状态
    }
    
    void Operation2(){
        
        //..........
        pNext = ConnectState::getInstance();
    }
    
    void Operation3(){
        
        //$$$$$$$$$$
        pNext = OpenState::getInstance();
    }
    
    
};


class CloseState:public NetworkState{ }
//...
```



`NetworkProcessor`其实并不关心是什么状态，只要是正确的状态的即可,状态的具体行为由状态类自己去扩展。这样当我们扩展状态的时候，并不需要改变`NetworkProcessor`只需要多谢一个状态类即可

```cpp
class NetworkProcessor{//其实并不关心是什么状态，只要是正确的状态的即可
    
    NetworkState* pState;
    
public:
    
    NetworkProcessor(NetworkState* pState){
        
        this->pState = pState;
    }
    
    void Operation1(){
        //...
        pState->Operation1();//状态具体行为由状态自己去管理
        pState = pState->pNext;//也不关心下一个状态是什么
        //...
    }
    
    void Operation2(){
        //...
        pState->Operation2();
        pState = pState->pNext;
        //...
    }
    
    void Operation3(){
        //...
        pState->Operation3();
        pState = pState->pNext;
        //...
    }

};
```

`state2.cpp`

```cpp
class NetworkState{

public:
    NetworkState* pNext;
    virtual void Operation1()=0;
    virtual void Operation2()=0;
    virtual void Operation3()=0;

    virtual ~NetworkState(){}
};


class OpenState :public NetworkState{
    
    static NetworkState* m_instance;//这里实际上用了单例模式
public:
    static NetworkState* getInstance(){
        if (m_instance == nullptr) {
            m_instance = new OpenState();
        }
        return m_instance;
    }

    void Operation1(){
        
        //**********
        pNext = CloseState::getInstance();
    }
    
    void Operation2(){
        
        //..........
        pNext = ConnectState::getInstance();
    }
    
    void Operation3(){
        
        //$$$$$$$$$$
        pNext = OpenState::getInstance();
    }
    
    
};

class CloseState:public NetworkState{ }
//...


class NetworkProcessor{
    
    NetworkState* pState;
    
public:
    
    NetworkProcessor(NetworkState* pState){
        
        this->pState = pState;
    }
    
    void Operation1(){
        //...
        pState->Operation1();
        pState = pState->pNext;
        //...
    }
    
    void Operation2(){
        //...
        pState->Operation2();
        pState = pState->pNext;
        //...
    }
    
    void Operation3(){
        //...
        pState->Operation3();
        pState = pState->pNext;
        //...
    }

};

```

## 结构设计

![image-20220212233648859](https://s2.loli.net/2022/02/12/wTrPM9RDug6ovcj.png)

## 要点总结

* State模式将所有与一个特定状态相关的行为都放入一个State的子类对象中 在对象状态切换时，切换相应的对象，但同时维持State的接口，这样实现了具体操作与状态转换之间的解耦。 
* 为不同的状态引入不同的对象使得状态转换变得更加明确，而且可以保证不会出现状态不一致的情况，因为转换是原子性的-----即 . 要么彻底转换过来，要么不转换。
* 如果State对象没有实例变量，那么各个上下文可以共享同个State对象，从而节省对象开销。(单例模式)