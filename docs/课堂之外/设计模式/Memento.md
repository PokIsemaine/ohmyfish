# Memento

## 动机

* 在软件构建过程中，某些对象的状态在转换过程中，可能由于某种需要，要求程序能够回溯到对象之前处于某个点时的状态。如果使用一些公有接口来让其他对象得到对象的状态，便会暴露对象的细节实现。 
* 如何实现对象状态的良好保存与恢复？但同时又不会因此而破坏对象本身的封装性。

## 模式定义

> **在不破坏封装性的前提下**（信息隐藏），捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可以将该对象恢复到原先保存的状态。 
>
> <p align="right">——《设计模式》GoF<p>



## SHOW ME THE CODE

```cpp

class Memento
{
    string state;
    //..
public:
    Memento(const string & s) : state(s) {}
    string getState() const { return state; }
    void setState(const string & s) { state = s; }
};



class Originator
{
    string state;
    //....
public:
    Originator() {}
    Memento createMomento() {
        Memento m(state);//实际实现可能相当复杂，往往采用一些编码技术处理（内存流、字符串、序列化）
        return m;
    }
    void setMomento(const Memento & m) {
        state = m.getState();
    }
};



int main()
{
    Originator orginator;
    
    //捕获对象状态，存储到备忘录
    Memento mem = orginator.createMomento();
    
    //... 改变orginator状态
    
    //从备忘录中恢复
    orginator.setMomento(memento);

   
    
}
```

理解起来比较简单，可以当作是对状态拍个照，然后照着照片恢复

但是具体实现很有难度：如何保存状态、保存几次、如何恢复、保存哪些状态？

## 结构设计

![image-20220214155201456](https://s2.loli.net/2022/02/14/CJX1riLp4B3dbgx.png)

## 要点总结

* 备忘录（Memento）存储原发器（Originator）对象的内部状态， 在需要时恢复原发器状态。 

* Memento模式的核心是信息隐藏，即 Originator 需要向外接隐藏信息，保持其封装性。但同时又需要将状态保持到外界 ( Memento )
*  由于现代语言运行时（如C#、Java等）都具有相当的对象序列化 持，因此往往采用效率较高、又较容易正确实现的序列化方案实现 Memento模式。