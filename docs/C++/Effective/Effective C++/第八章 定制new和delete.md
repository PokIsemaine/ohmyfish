# 第八章 定制new和delete

## 条款49：了解 new-handler 的行为

* 当`operator new`抛出异常以反映一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，一个所谓的`new-handler`
* 为了指定`new-handler`这个用以处理内存不足的函数，客户必须调用`set_new_handler`（参数是一个指针，指向`operator new`无法分配足够内存时该被调用的函数。其返回值也是个指针，指向`set_new_handler`被调用前正在执行（但马上要被替换）的那个`new-handler`函数
* 当`operator new`无法满足内存申请时，它会不断调用`new-handler`函数，直到找到足够内存
* 一个设计良好的`new-handler`函数必须做以下事情
	* 让更多内存可被使用
	* 安装另一个`new-handler`
	* 卸除`new-handler`
	* 抛出`bad_alloc`的异常
	* 不返回



* `set_new_handler`允许客户指定一个函数，在内存分配无法满足时被调用
* `Nothrow new`是个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常



## 条款50：了解 new 和 delete 的合理替换时机

* 替换编译器提供的`operator new`或`operator delete`的常见理由
	* 用来检测运用上的错误
	* 为了强化效能
	* 为了收集使用上的数据
* 本条款的主题是，了解何时可在全局性的或`class`专属的基础上合理替换缺省的`new`和`delete`
	* 为了检测运用错误
	* 为了收集动态分配内存之使用统计信息
	* 为了增加分配和归还速度
	* 为了降低缺省内存管理器带来的空间额外开销
	* 为了弥补缺省分配器中的非最佳齐位
	* 为了将相关对象成簇集中
	* 为了获得非传统的行为
* 有许多理由需要写一个自定的`new`和`delete`，包括改善效能、对`heap`运用错误进行调试、收集`heap`使用信息



## 条款51：编写 new 和 delete 时需固守常规

* `operator new`应该包含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用`new-handler`。它也应该有能力处理`0 bytes`申请。`Class`专属版本则还应该处理比正确大小更大的错误申请
* `operator delete`应该在受到`null`指针时不做任何事。`Class`专属版本则还应该处理比正确大小更大的错误申请



## 条款52：写了 placement new 也要写 placement delete

* 当你写一个 `placement operator new`， 请确定也写出了对应的` placement operator delete`。如果没有这样做，你的程序可能会发生隐蔽而时断时续的内存泄漏
* 当你声明`placement new`和`placement delete`，请确定不要无意识地遮掩了它们的正常版本