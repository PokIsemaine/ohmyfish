# 参与开源 C++ 实战

## 一、文档

### 绘图工具





## 二、测试

### 单元测试



### 基准测试

google benchmark

绘图



## 三、部署

### 支持 Docker 部署



## 四、编译

GCC编译优化应用预编译头 https://blog.csdn.net/zhuaizi888/article/details/123598645?spm=1001.2014.3001.5501

GCC编译优化应用预编译头(二) https://blog.csdn.net/zhuaizi888/article/details/125248520?spm=1001.2014.3001.5501

如何加快C++代码的编译速度 https://zhuanlan.zhihu.com/p/29346995

iwyu:https://github.com/include-what-you-use/include-what-you-use

### 预编译头文件

考虑预编译头文件，特别是 headonly 项目

1. 对所有单元测试使用 cmake 的预编译功能
2. 将所有pch文件保存在github中（大概10-15M）
3. 每次在 CI 中我们都可以下载它并编译。

### 加速 CI with ccache

https://cristianadam.eu/20200113/speeding-up-c-plus-plus-github-actions-using-ccache/
https://github.com/cristianadam/HelloWorld
https://github.com/Chocobo1/setup-ccache-action
https://github.com/marketplace/actions/ccache-for-gh-actions

## 五、代码

### 