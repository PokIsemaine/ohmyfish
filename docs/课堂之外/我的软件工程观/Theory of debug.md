# Theory of debug 调试理论

[TOC]

# Summary 摘要

在大二下，我才意识到很多人（包括我自己），对于调试的理解和实践存在一些问题。于是去看了一些资料（见Reference）并写下这篇文章作为总结。

这篇总结是否写得有些晚了，换句话说是不是学调试学的有些晚。对我来说或许是的，



适合人群：有一定编程经验，被 bug 按在地上摩擦过的同学。

建议了解的前置知识：二分、状态机、GDB



文章将包含以下几点

* Bug 的基本概念、由来
* Theory of debug 调试理论要点

# Bug 漏洞

![image-20220222213700785](https://s2.loli.net/2022/02/22/2XuLThP5GfORHoZ.png)



使用翻译软件查询`bug`的含义我们的到的解释有“漏洞”、“窃听”、“飞虫”、“恼人”等

其中“飞虫”显得有些特殊，那么我们就从它开始讲起

## First bug 第一个漏洞

![](https://s2.loli.net/2022/02/22/QZaDfmikpOFtWsq.jpg)

> From Debugging-Wikipedia
>
> The terms "bug" and "debugging" are popularly attributed to [Admiral Grace Hopper](https://en.wikipedia.org/wiki/Admiral_Grace_Hopper) in the 1940s.[[1\]](https://en.wikipedia.org/wiki/Debugging#cite_note-1) While she was working on a [Mark II](https://en.wikipedia.org/wiki/Harvard_Mark_II) computer at Harvard University, her associates discovered a moth stuck  in a relay and thereby impeding operation, whereupon she remarked that  they were "debugging" the system. 

1940s 海军上将  Grace Hopper 在哈佛大学的 `Mark II` 电脑上工作的时候发现一只飞蛾卡在继电器之间，阻碍了正常操作。然后她就记录下来并说他们正在 Debug(除虫)。从这以后 Bug 便经常用来代指程序中的错误或漏洞

## Software bug 软件中的漏洞

软件中的 bug 主要由两种情况导致：需求理解出错或代码实现出错

程序的两大功能：人类世界需求的载体，计算过程的精确描述

* 如果理解错需求，那么会导致 bug

* 如果实现错误，那么会导致 bug

![image-20220222222132019](https://s2.loli.net/2022/02/22/jxznihyCR8XsEpo.png)



# Debugging 调试

## Difficulty in debugging 调试的难点

调试的难点在哪里？我认为是找 bug 的过程。大多时候我们并不是找到了 bug 不会改，而是根本没找到 bug。

是什么增加了寻找 bug 源头的难度呢？

* 定位难：bug **触发**到**被观测**到错误有可能经历了**很长**一段过程
* 检查难：难以判断一个状态是否是正确的
  * 需求理解不正确
  * 观测的状态量不足以真正反应一个状态的正确性
* 观测难：



### 定位难

![image-20220227180341599](https://s2.loli.net/2022/02/27/Q4VGxmzfYj8HNLo.png)



举一个简单的例子

| 需求                                  | 设计/算法（假象代码）        | 实际代码                     | Fault             | Error                         | Failure |
| ------------------------------------- | ---------------------------- | ---------------------------- | ----------------- | ----------------------------- | ------- |
| $\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}i*j$ | `for(int j = 0; j < n; ++j)` | `for(int j = 0; j < n; ++i)` | `++i`,应该为`++j` | `i,j`状态<br />`(0,0)->(1,0)` | TLE超时 |

设计/算法（假象代码）

```cpp
#include <stdio.h>

#define n 3

int main(){
    int sum = 0;
    for(int i = 0; i < n; ++i)
        for(int j = 0; j < n; ++j)
            sum += i * j;
    printf("sum = %d\n",sum);
    return 0;
}
```

实际代码

```cpp
#include <stdio.h>

#define n 3

int main(){
    int sum = 0;
    for(int i = 0; i < n; ++i)
        for(int j = 0; j < n; ++i)//Fault(Bug) ++i,应该为++j
            sum += i * j;
    printf("sum = %d\n",sum);
    return 0;
}
```



> 程序就是状态机！！！

上面两个程序代码对应的状态机如下图所示

![image-20220227180942349](https://s2.loli.net/2022/02/27/31CVADzE2ad7pbi.png)

可以看到实际程序状态机的状态从第二个状态就开始错误，经过了很长一段时间才被观察到TLE。

那么首先的一个问题就是**如何去定位**到第一个Error的状态，在定位过程中我们暂时先不考虑用于需求理解错误导致的 bug，这需要一些软件工程领域的方法论并且每个人的理解与抽象能力各不相同我们很难保证从自然语言到程序语言的传递过程中的一致性。（如果解决不了问题可以考虑一下解决一下产生问题的那个人）



### 检查难

对于得到过基本代码训练的同学来说，上面例子中的错误应该能够很快就能发现并解决。

对于我短暂的写代码经历而言，最令我头疼的是动态规划类的算法题目。一方面，由于我较弱的抽象能力与模型转化能力，我难以将一个自然语言描述的题目准确地转化为正确的数学模型，这是需求到设计/算法（假象代码）过程的能力确实。另一方面，由于我较弱的代码能力，我难以将我的想法用代码正确且高效的描述出来，这是设计/算法（假象代码）到实际代码的能力缺失。（没错，我就是这么弱）

在很长一段时间内，我因为上面两个归因结果自我怀疑过（虽然确实如此），最近我意识到对于某些程序来说，调试它本身就是一件很难的事情，甚至不可调试。

前面我们提到了 “程序就是状态机” 这一观点，那么我们检查一个程序的状态正确与否本质上就是检查实际上代码的状态机是否与假想代码的状态机匹配。



所有程序都能进行有意义的检查吗？

人脑做检查？

用于判断状态正确与否的状态数是否正确且足够？





机器检查？

人类是有极限的！所以我不做人了啦！

![](https://tse1-mm.cn.bing.net/th/id/OIP-C.cF4t6XlQMW2XOIK1T4zMrAHaEL?w=332&h=187&c=7&r=0&o=5&pid=1.7)



谁来编写规则呢？还是人啊

用于检查的状态数太多啦！

调试输出的东西如果完全没有可读性，那和没调试有什么区别？抱着试试看的心态去调试追踪一些东西， 动手按按键盘、点点鼠标或许可以给你心理上的安慰，但往往是在做无用功。



### 观测难

是否可观测？

观测是否会有影响？薛定谔的猫



回顾一下我们要解决问题：

1. 因为 bug 触发到被观测到错误有可能经历了很长一段过程，我们要找一个高效的方法去定位



## 如何使定位更容易

### 时间黑洞恐惧下的单步调试

> 二八定律（帕累托原则）：一小部分原因导致大部分结果。

单步调试



### 调试理论

> 调试理论：**如果我们能判定任意程序状态的正确性**，那么给定一个failure，我们可以通过二分查找定位到
>
> **第一个**error 的状态，此时的代码就是fault (bug)。
>
> <p align="right">———[NJU 2021秋季学期ICS课程]王慧妍老师的课件</p>





## 如何使检查更容易

### 更好的输出格式



## 如何使观测更容易



## Printf 与 GDB



单步调试，二八定理时间黑洞

printf 手动二分01二分，trace日志怎么写？

gdb 

状态是否足够









## Recommended practice 推荐实践

### Before 调试之前

与其说是预先准备，不如说是习惯养成

摆正心态：机器永远是对的，未测代码永远是错的

利用编译选项来避免/监控可能犯下的错误

```shell
-Wall
-fsanitize=undefined,address
```

代码习惯

利用断言

### Doing 调试时



### After 调试后



# More than just programs 不止程序

## 调试的三个对象

### 人



### 调试目标



### 调试过程

## 调试的本质

调试的本质是控制手段



## 调试一切



# Reference 参考资料

[NJU 2021秋季学期ICS课程](http://jyywiki.cn/ICS/2021/)

[Debugging-Wikipedia](https://en.wikipedia.org/wiki/Debugging)

[调试理论](https://zhuanlan.zhihu.com/p/141514396)

[二八定律](https://chinese.freecodecamp.org/news/the-80-20-rule-pareto-principle/)