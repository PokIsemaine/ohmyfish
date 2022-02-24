# Reference

![image-20220223162044074](https://s2.loli.net/2022/02/23/1957XtmkeVPnzxA.png)

* `object`和`reference`大小相同地址也相同。
* `reference`代表`object`，`pointer`指向`object`，做区分。
* `reference`一定要有初值
* `reference`确定代表谁之后不能变了，代表谁是初始化的时候确定的，后面再用`=`就变成赋值了

![image-20220223162356186](https://s2.loli.net/2022/02/23/pZd7UoQFibfDk9M.png)

## reference 常见用途

![image-20220223162815213](https://s2.loli.net/2022/02/23/UgZIM2GVYo7CzLA.png)

* `reference` 一般不用与声明变量，常用于传参数或者返回类型的描述
* 函数签名：不含`return type`，`const`是函数签名一部分，传引用和传值区分不了

```cpp
double imag(const double& im){...}//imag1
double imag(const double im){...}//imag2
double imag（const double&im)const{...}//imag3 const也是函数签名的一部分，可以和imag1q
double im;
imag(im);//Ambiguity 二义性，不知道调用哪个了imag1还是imag2
```

