# Chain of Resposibility

## 动机

- 在软件构建过程中，一个请求可能被多个对象处理，但是每个请求在运行时只能有一个接收者，如果显式指定，将必不可少地带来请求发送者与接收者的紧耦合。
- 如何使请求的发送者不需要指定具体的接收者？让请求的接收者自己在运行时决定来处理请求，从而使两者解耦。

## 模式定义

> 使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。 
>
> <p align="right">——《设计模式》GoF</p>





## SHOW ME THE CODE

```cpp
#include <iostream>
#include <string>

using namespace std;

enum class RequestType
{
    REQ_HANDLER1,
    REQ_HANDLER2,
    REQ_HANDLER3
};

class Reqest
{
    string description;
    RequestType reqType;
public:
    Reqest(const string & desc, RequestType type) : description(desc), reqType(type) {}
    RequestType getReqType() const { return reqType; }
    const string& getDescription() const { return description; }
};

class ChainHandler{
    
    ChainHandler *nextChain;//其实就是一个链表
    void sendReqestToNextHandler(const Reqest & req)
    {
        if (nextChain != nullptr)
            nextChain->handle(req);
    }
protected:
    virtual bool canHandleRequest(const Reqest & req) = 0;//运行时判断这个请求能不能处理
    virtual void processRequest(const Reqest & req) = 0;//具体处理请求
public:
    ChainHandler() { nextChain = nullptr; }
    void setNextChain(ChainHandler *next) { nextChain = next; }
    
   
    void handle(const Reqest & req)
    {
        if (canHandleRequest(req))//如果当前对象能处理这个请求就处理
            processRequest(req);
        else//如果处理不了就往后传
            sendReqestToNextHandler(req);
    }
};


class Handler1 : public ChainHandler{
protected:
    bool canHandleRequest(const Reqest & req) override
    {
        return req.getReqType() == RequestType::REQ_HANDLER1;
    }
    void processRequest(const Reqest & req) override
    {
        cout << "Handler1 is handle reqest: " << req.getDescription() << endl;
    }
};
        
class Handler2 : public ChainHandler{
protected:
    bool canHandleRequest(const Reqest & req) override
    {
        return req.getReqType() == RequestType::REQ_HANDLER2;
    }
    void processRequest(const Reqest & req) override
    {
        cout << "Handler2 is handle reqest: " << req.getDescription() << endl;
    }
};

class Handler3 : public ChainHandler{
protected:
    bool canHandleRequest(const Reqest & req) override
    {
        return req.getReqType() == RequestType::REQ_HANDLER3;
    }
    void processRequest(const Reqest & req) override
    {
        cout << "Handler3 is handle reqest: " << req.getDescription() << endl;
    }
};

int main(){
    Handler1 h1;
    Handler2 h2;
    Handler3 h3;
    h1.setNextChain(&h2);
    h2.setNextChain(&h3);
    
    Reqest req("process task ... ", RequestType::REQ_HANDLER3);
    h1.handle(req);
    return 0;
}
```



## 结构设计

![image-20220214230232888](https://s2.loli.net/2022/02/14/3MwRZyQXLpaYI8S.png)

## 要点总结

* Chain of Responsibility 模式的应用场合在于 一个请求能有多个接受者，但是最后真正的接受者只有一个。这时候请求发送者与接受者的耦合有可能出现 “变化脆弱” 的症状，职责链的目的就是将二者解耦，从而更好地应对变化
* 应用了 Chain of Responsibility 模式 后，对象的职责分派更具灵活性。我们可以在运行时动态添加/修改请求的处理职责
* 如果请求传递到职责链的末尾仍得不到处理 ，一个合理缺省机制。这也是每 一个接受对象的责任，而不是发出请求对象的责任
* 有些过时了，今天看来职责链其实就是一个链表，不过要注意 GOF 写的设计模式这本书很早啊，当时一些概念、技术等等还不是特别成熟