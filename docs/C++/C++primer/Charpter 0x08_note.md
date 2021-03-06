<h1 align="center">📔 C++ Primer 0x08 学习笔记</h1>

## 8.1 IO 类

* 标准库通过继承机制，使我们能忽略不同类型的流之间的差异
* 继承机制使我们可以声明一个特定的类继承自另一个类，我们通常可以将一个派生类（继承类）对象当作其基类（所继承的类）对象来使用

### 8.1.1 IO 对象无拷贝或赋值

* 我们不能拷贝或对 IO 对象赋值
* 不能将形参或返回类型设为刘类型
* 进行 IO 操作的函数通常以引用方式传递和返回流，因为读写一个 IO 对象会改变状态，因此传递和返回的引用不能是 `const`的

### 8.1.2 条件状态

* IO 类定义了一些函数和标志可以帮助我们访问和操纵流的条件状态
* 一个流一旦发生错误，其上后续的 IO 操作都会失败，只有当一个流处于无错状态是才可以从它读取数据
* `iostate`表示流状态，当作一个位集合使用。`rdstate`成员返回一个`iostate`，`setstate`操作将给定条件置位
* 如果`badbit`、`failbit`和`eofbit`任一被置位，检测流状态的条件就会失败

### 8.1.3 管理输出缓冲

* 每个输出流都管理一个缓冲区，用来保存程序的读写数据，这可以提高性能
* 导致缓冲刷新（数据写到输出设备或文件）的原因有很多
	* 程序正常结束，作为`main`函数的`return`操作的一部分，缓冲刷新被执行
	* 缓冲区满时，需要刷新缓存，而后新的数据才能继续写入缓冲区
	* 使用`endl`显示刷新缓冲区
	* 可以通过`unitbuf`来设置流的内部状态清空缓冲区，默认清空下，对`cerr`设置`unitbuf`因此写到`cerr`内容都是立即刷新的。
	* 一个输出流可能被关联到另一个输出流，读写被关联的流是，关联到的流的缓冲区会被刷新。`cin`和`cerr`都关联到`cout`，读`cin`或写`cerr`都会导致`cout`的缓冲区被刷新
* `flush`刷新缓冲区，但不输出任何额外字符
* `ends`向缓冲区插入一个空字符，然后刷新缓冲区
* 如果程序异常终止，输出缓冲区是不会被刷新的。当调试已经崩溃的程序时，需要确认那些你认为已经输出的数据已经刷新了，否则可能将大量时间浪费在追踪代码为什么没有执行上，实际上已经执行了，只是崩溃后缓冲区没有被刷新，输出数据被挂起没有打印而已
* 当输入流关联到输出流是，任何试图从输入流读数据的操作都会先刷新关联的输出流，标准库将`cout`和`cin`关联在一起
* `x.tie(&o)`将流x关联到输出流o
* 每个流同时最多关联一个流，但是多个流可以同时关联同一个`ostream`

## 8.2 文件输入输出

### 8.2.1 使用文件流对象

* 想要读写一个文件，可以定义一个文件流对象，并将对象与文件关联起来
* 文件流类有个`open`的成员函数，创建流对象时如果提供了文件名，则`open`会自动被调用
* 用`fstream`代替`iostream&`
* 使用`open`将一个空文件流对象和文件关联起来，进行`open`是否成功的检测通常是一个好习惯
* 当一个`fstream`对象离开其作用域时，与之关联的文件会自动关闭
* 当一个`fstream`对象被销毁时，`close`会自动被调用

### 8.2.2 文件模式

* 每个流都有一个关联的文件模式，用来指出如何使用文件
* 只可以对 `ofstream`或`fstream`对象设定`out`模式、`in`模式
* 只有当`out`也被设定是才可以设定`trunc`模式
* 只要`trunc`没被设定，就可以设定`app`模式。`app`模式下文件总是以输出方式打开
* 默认情况下即使没设定`trunc`，以`out`模式打开的文件也会被截断
* `ate`和`binary`模式可用于任何文件流对象
* `ifstream`关联的文件默认以`in`模式打开
* `ofstream`关联的文件默认以`out`模式打开
* `fstream`关联的文件默认以`in`和`out`模式打开
* 以`out`模式打开文件会丢弃已有数据，阻止方法是指定`app`或`in`模式
* 每次调用`open`都会确定文件模式，可能显示设置，有可能隐式地设置。如果没指定就使用默认值

## 8.3 string 流

* 当我们某些工作是对整行文本进行处理，而其他一些工作是处理行内的单个单词，通常可以使用`istringstream`，`.str()`可以关联一个`string`，`.clear()`清空
* 当我们逐步构造输出，希望最后一起打印时，`ostringstream`很有用
* `stringstream`既可以读也可以写