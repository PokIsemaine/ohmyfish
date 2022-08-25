# Flyweight

## 动机

- 在软件系统采用纯粹对象方案的问题在于**大量细粒度的对象**会很快充斥在系统中，从而带来很高的运行时代价——**主要指内存需求方面的代价。**
- 如何在避免大量细粒度对象问题的同时，让外部客户程序仍然能够透明地使用面向对象的方式来进行操作？

## 模式定义

**运行共享技术**有效地支持大量细粒度的对象。 ——《设计模式》GoF



## SHOW ME THE CODE

过去有一种思潮：一切皆对象

但是后来发现如果什么都当作对象也会有缺点，其中一个就是如果有大量细粒度的对象充斥在系统中会带来很大的运行时代价（主要内存需求方面）

为了解决这个问题，我们可以运用共享的思想来支持大量细粒度的对象

例子：实现字处理系统并假设字体也是对象

字体对象有可能量很大，每个字符都有一个字体，但实际上一篇文章中并不会有很多种字体类型。也就是说，绝大部分字符的字体对象都是相同的。

我们可以弄一个对象池来实现，Flyweight 实现方式很多样，**总体来说要注重设计思想：运行共享技术有效地支持大量细粒度的对象**

另外像`JAVA`、`C#`等等会在编译器层面就会实现和`Flyweight`类似的设计

```cpp
class Font {
private:

    //unique object key
    string key;
    
    //object state
    //....
    
public:
    Font(const string& key){
        //...
    }
};

class FontFactory{
private:
    map<string,Font* > fontPool;
    
public:
    Font* GetFont(const string& key){

        map<string,Font*>::iterator item=fontPool.find(key);
        
        if(item!=footPool.end()){
            return fontPool[key];
        }
        else{
            Font* font = new Font(key);
            fontPool[key]= font;
            return font;
        }

    }
    
    void clear(){
        //...
    }
};
```



## 结构设计

![image-20220210161615417](https://s2.loli.net/2022/02/10/8Qd9YItzcbSexHg.png)

## 要点总结

- 面向对象很好地解决了抽象性的问题，但是作为一个运行机器中的程序实体，我们需要考虑对象的代价问题， Flyweight 主要解决面向对象的代价问题，一般不触及面向对象的抽象性问题。
- Flyweight 采用对象共享的做法来降低系统中对象的个数，从而降低细粒度对象给系统带来的压力。在具体实现方面，要注意对象状态的处理。
- 对象的数量太大从而导致对象内存开销加大——什么样的数量才算大？这需要我们仔细的根据**具体应用情况进行评估，而不能凭空臆断。**（sizeof 去看）