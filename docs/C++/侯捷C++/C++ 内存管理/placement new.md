# placement new

![image-20220302145549882](https://s2.loli.net/2022/03/02/h6GENMQBriTHuVj.png)

* `placement new`可以认为是定点的`new`，指定位置调用构造函数
* `placement new`实际上没有分配内存，所以也不存在`placement delete`。实际上如果用了`placement new`，我们也要使用对应的`delete`，有人会把这个`delete`称为`placement delete`

```cpp
Complex* pc = new(buf)Complex(1,2);//placment new
//编译器转为
Complex *pc;
try{
    void* mem = operator new(sizeof(Complex),buf);//传入位置又直接返回位置，没有分配内存，相当于啥都没做
    pc = static_cast<Complex*>(mem);//cast类型转换，无关紧要
    pc->Complex::Complex(1,2);//调用构造函数
}
//前两步操作基本没什么用，所以可以认为 placement new 等同于 调用构造函数了
```



