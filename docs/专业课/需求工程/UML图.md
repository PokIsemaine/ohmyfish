# UML图

## 一些区别

**类图和对象图**

对象图用来描述特定时刻各个对象之间的关系，是类图在某个特定时刻的实例

| 类图                                           | 对象图                                               |
| ---------------------------------------------- | ---------------------------------------------------- |
| 三个分栏：名称、属性、操作                     | 两个分栏：名称、属性                                 |
| 名称部分只有类名                               | 名称部分"对象：类名"，匿名对象"：类名"，有下划线     |
| 类中有操作                                     | 对象图中没操作，因为对于同一个类的对象操作都是相同的 |
| 类使用关联连接，有名称、角色、多重性以及约束等 | 对象图用链连接，有名称、角色，没有多重性             |
| 类的属性定义了所有属性的特征                   | 对象只定义了属性的当前值                             |

**活动图与状态图区别**

相同点:

* 图符基本一致
* 描述一个系统或对象在生存期间的状态或行为
* 可以描述系统或对象在多进程操作中的同步与异步操作行为
* 可以用条件分支描述一个系统或对象的行为控制流

区别

| 活动图                               | 状态图                                   |
| ------------------------------------ | ---------------------------------------- |
| 不需要事件触发                       | 需要触发                                 |
| 用泳道来描述多个对象共同完成一个操作 | 用状态嵌套来描述多个对象共同完成一个操作 |



## 用例图

![用例图](https://s2.loli.net/2022/05/23/PO8zfMwRKuk3W7l.png)

* 注意有没有肩头
* 注意是否虚线
* 注意箭头方向

![用例建模](https://s2.loli.net/2022/05/23/Gbu6vFSdTpBcKEa.png)



## 活动图

![活动图](https://s2.loli.net/2022/05/22/JhaiWsd5F76eukt.png)

### 借阅图书

![借阅图书](https://s2.loli.net/2022/05/22/sBIDml5qHXP2JzL.png)

### 图书管理员

![图书管理员](https://s2.loli.net/2022/05/22/Pn7zgXTcdbMUutS.png)

### 系统管理员

![系统管理员](https://s2.loli.net/2022/05/22/f2spXUq1TeLz7JV.png)

## 状态图

![状态图](https://s2.loli.net/2022/05/22/HaJVkXIYs2pg3KZ.png)



### 设备机床嵌套状态图

![设备机床](https://s2.loli.net/2022/05/22/MlZ7u2HhnfTVONE.png)

### POS 机

![image-20220522202551802](https://s2.loli.net/2022/05/22/IUCPKzOuS2DR9rt.png)

## 类图

### 类之间的关系

![类关系](https://s2.loli.net/2022/05/23/xZC7nPDAUioa2bd.png)





### 借书

![书](https://s2.loli.net/2022/05/22/bwSxMZB15e4VUJL.png)

### 动物

![动物](https://s2.loli.net/2022/05/22/4xpg5IDrcMTNmdJ.png)

### 人类

![人类](https://s2.loli.net/2022/05/22/xrKvw8UpkC5iuRF.png)