# new handler

可以看一下《Effective C++》第8章

## 目的

当`operator new`失败的时候会先抛出一个`std::bad_alloc exception`异常。

但是有时候我们希望能进行一些操作让内存申请可行而不是直接抛出异常（抛出异常就覆水难收了）

这时候我们可以用`new handler`指定抛出异常前尝试的操作，有可能是下面两种

* 想办法让更多内存可用
* 找不到解决方案直接摆烂`abort()`或者`exit()`

![image-20220304091533709](https://s2.loli.net/2022/03/04/sCoXtQKd74GEUHh.png)



## 示例

![image-20220304092104238](https://s2.loli.net/2022/03/04/DtIhkvrYMOgi2Kq.png)