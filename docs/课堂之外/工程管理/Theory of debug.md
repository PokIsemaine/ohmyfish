# Theory of debug 调试理论

[TOC]

# Summary 摘要

适合人群：有一定编程经验，被 bug 按在地上摩擦过的同学

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

bug **触发**到**被观测**到错误有可能经历了**很长**一段过程

* 定位难：不知道是哪里触发的
	* 如何去定位？
* 观测难：状态规模巨大？
* 检查难：难以判断一个状态是否是正确的
	* 需求理解不正确
	* 观测的状态量不足以真正反应一个状态的正确性

![image-20220223142516772](https://s2.loli.net/2022/02/23/8W7Cqn6ij3vkOml.png)



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



可观测性



## Theory of debug 调试理论

> 调试理论：**如果我们能判定任意程序状态的正确性**，那么给定一个failure，我们可以通过二分查找定位到
>
> **第一个**error 的状态，此时的代码就是fault (bug)。
>
> <p align="right">———[NJU 2021秋季学期ICS课程]王慧妍老师的课件</p>



## Printf 与 GDB

单步调试，二八定理时间黑洞

printf 手动二分01二分，trace日志怎么写？

gdb 

状态是否足够

## Recommended practice 推荐实践

### Before 调试之前

摆正心态：机器永远是对的，未测代码永远是错的

利用编译选项

### Doing 调试时



### After 调试后



# More than just programs 不止程序

## 调试的三个对象

### 人



### 调试目标



### 调试过程

## 调试的本质



## 调试一切



# Reference 参考资料

[NJU 2021秋季学期ICS课程](http://jyywiki.cn/ICS/2021/)

[Debugging-Wikipedia](https://en.wikipedia.org/wiki/Debugging)

[调试理论](https://zhuanlan.zhihu.com/p/141514396)