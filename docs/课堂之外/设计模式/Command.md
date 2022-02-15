# Command

## 动机

- 在软件构建过程中，”行为请求者“与”行为实现者“通常呈现一种”紧耦合“。但在某些场合——比如需要对行为进行”记录、撤销/重（undo/redo）、事务“等处理，这种无法抵御变化的紧耦合是不合适的。
- 在这种情况下，如何将”行为请求者“与”行为实现者“解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

## 模式定义

> **将一个请求(行为)封装成一个对象（行为对象化）**，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。 
>
> <p align="right">——《设计模式》GoF</p>

## SHOW ME THE CODE

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;


class Command
{
public:
    virtual void execute() = 0;//接口规范，后面继承的要按照接口规范来实现，相比C++函数对象更严格
};

class ConcreteCommand1 : public Command
{
    string arg;
public:
    ConcreteCommand1(const string & a) : arg(a) {}
    void execute() override
    {
        cout<< "#1 process..."<<arg<<endl;
    }
};

class ConcreteCommand2 : public Command
{
    string arg;
public:
    ConcreteCommand2(const string & a) : arg(a) {}
    void execute() override
    {
        cout<< "#2 process..."<<arg<<endl;
    }
};
        
        
class MacroCommand : public Command
{
    vector<Command*> commands;
public:
    void addCommand(Command *c) { commands.push_back(c); }
    void execute() override
    {
        for (auto &c : commands)
        {
            c->execute();
        }
    }
};
        

        
int main()
{

    ConcreteCommand1 command1(receiver, "Arg ###");//构建命令
    ConcreteCommand2 command2(receiver, "Arg $$$");
    
    MacroCommand macro;//命令组合
    macro.addCommand(&command1);
    macro.addCommand(&command2);
    
    macro.execute();//命令执行

}
```

`command1`、`command2`、`macro`都是对象，但是它们表示行为。

作为对象就更加灵活了，可以作为参数传递，可以当作字段来序列化，可以存在某些数据结构里。

比如把表示行为的对象放到栈里，就可以实现行为的`redo/undo`

## 结构设计

![image-20220215150951271](https://s2.loli.net/2022/02/15/1Ynv6gBXbCFz3eK.png)



## 要点总结

- Command 模式的根本目的在于将”行为请求者“与”行为实现者“解耦，在面向对象语言中，常见的实现手段是”将行为抽象为对象“。
- 实现 Command 接口的具体命令对象 ConcreteCommand 有时候根据需要可能会保存一些额外的状态信息。通过使用 Composite 模式，可以将多个命令封装为一个复合命令
- Command 模式与 C++ 中的函数对象类似。但二者定义行为接口的规范有所区别。Command 以面向对象中的“接口-实现”来定义行为接口规范，更严格，但有性能损失。C++函数对象以函数签名来定义行为接口规范（隐式），更灵活，性能更高。
	- C++ 函数对象利用了重载函数调用符，还可以用模板编译时绑定。虽然是编译时绑定，但不意味着不灵活，因为我们可以利用泛型编程模板来实现编译时多态
	- Command 则是用虚函数实现的运行时绑定