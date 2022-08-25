# Singleton 模式

## 动机

- 在软件系统中，经常有这样一些特殊的类，必须保证它们在系统中只存在一个实例，才能确保它们的**逻辑正确性、以及良好的效率**。
- 如何绕过常规的构造器，提供一种机制来保证一个类只有一个实例？
- 这应该是类设计者的责任，而不是使用者的责任。



## 模式定义

保证一个类仅有一个实例，并提供一个该实例的全局访问点。 ——《设计模式》GoF



## SHOW ME THE CODE

```cpp
class Singleton{
private:
    Singleton();//构造函数设为私有
    Singleton(const Singleton& other);
public:
    static Singleton* getInstance();
    static Singleton* m_instance;//静态变量
};

Singleton* Singleton::m_instance=nullptr;//静态变量初始化为nullptr,我们确定其为堆对象

//线程非安全版本
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}
```

上述代码并非线程安全，假设两个线程 A、B

A 先抢到时间片执行了`if (m_instance == nullptr)`，B 又抢到了时间片。那么接下来有可能`A`、`B`两个线程都会执行`m_instance = new Singleton();`

### 优化 1：加锁实现线程安全

```cpp
//线程安全版本，但锁的代价过高
Singleton* Singleton::getInstance() {
    Lock lock;//获取锁
    if (m_instance == nullptr) {//创建
        m_instance = new Singleton();
    }
    return m_instance;//读
}
```

`thread A`获取锁然后往下执行，此时`thread B`如果也调用这个方法，那么就要等`thread A`释放锁才可以

但这么做有个缺点：锁的代价太高

如果`m_instance`已经不是`nullptr`了，实际上没必要加锁了。因为现在我们只是需要读一个变量，而非创建一个变量



## 优化 2：双检查锁（不能用，出错概率很高）

锁前检查，锁后不检查，会出问题

```cpp
//双检查锁，但由于内存读写reorder不安全
Singleton* Singleton::getInstance() {
    
    if(m_instance==nullptr){//只要两个线程通过了这个if语句，那么下面的锁就没啥意义了，还是会创建多个对象
        Lock lock;
        m_instance = new Singleton();
    }
    return m_instance;
}
```



锁后检查，锁前不检查，虽然正确但是代价过高

```cpp
//线程安全版本，但锁的代价过高
Singleton* Singleton::getInstance() {
    Lock lock;//获取锁
    if (m_instance == nullptr) {//锁后检查
        m_instance = new Singleton();
    }
    return m_instance;//读
}
```



我们可以选择双检查锁

```cpp
//双检查锁，但由于内存读写reorder不安全
Singleton* Singleton::getInstance() {
    
    if(m_instance==nullptr){//锁前检查，避免都是读取操作的时候代价过高
        Lock lock;
        if (m_instance == nullptr) {//锁后检查，避免通过锁前检查后创建多个对象
            m_instance = new Singleton();
        }
    }
    return m_instance;
}
```



看起来没问题了？并不，内存读写`reorder`会造成双检查锁的失效

什么是内存读写`reorder`？

我们通常会认为指令序列会按照我们想的方式实现，但是到了 CPU 的指令层次的时候，实际顺序有可能和我们想的不一样

并且要注意的是，线程正是在指令层次去争抢时间片的

```cpp
//双检查锁，但由于内存读写reorder不安全
Singleton* Singleton::getInstance() {
    
    if(m_instance==nullptr){//锁前检查，避免都是读取操作的时候代价过高
        Lock lock;
        if (m_instance == nullptr) {//锁后检查，避免通过锁前检查后创建多个对象
            //m_instance = new Singleton();
            //我们假想的顺序
            //1.分配内存
            //2.调用构造器
            //3.返回内存地址 m_instance != nullptr
            //CPU指令层可能的顺序 reorder（编译器为了优化的时候可能就会这么干，你要看汇编）
            //1.分配内存
            //2.返回内存地址 m_instance != nullptr,但构造器还没调用呢，此时如果别的线程可能拿到的是一个没有构造的对象
            //3.调用构造器
        }
    }
    return m_instance;
}
```

那么如何解决这个问题呢？实际上从我们自己写代码的层面来解决它是很困难的，那么就让编译器来解决问题

解铃还需系铃人，编译器为了优化进行指令的`reorder`造成的锅，当然要让编译器来解决啦！

### 优化 3：volatile/atomic

`volatile/atomic`阻止了编译器优化

```cpp
//C++ 11版本之后的跨平台实现 (volatile)
std::atomic<Singleton*> Singleton::m_instance;
std::mutex Singleton::m_mutex;

Singleton* Singleton::getInstance() {
    Singleton* tmp = m_instance.load(std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire);//获取内存fence
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(m_mutex);
        tmp = m_instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton;
            std::atomic_thread_fence(std::memory_order_release);//释放内存fence
            m_instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}
```



## 结构设计

单看结构不看实现`Singleton`模式实际上非常简单

但是在多线程的情况下`Singleton`的实现需要注意线程安全、内存读写`reorder`等问题

![image-20220210154859602](https://s2.loli.net/2022/02/10/38vrL6AwhfRWEbG.png)

## 要点总结

- Singleton 模式中的实例构造器可以设置为 protected 以允许子类派生。
- Singleton 模式一般不要支持拷贝构造函数和 Clone 接口，因为这有可能导致多个对象实例，与 Singleton 模式的初中违背。
- 如何实现多线程环境下安全的 Singleton？注意对双检查锁的正确实现。