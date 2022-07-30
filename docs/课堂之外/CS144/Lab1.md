# Lab1 将子字符串拼接成一个字节流

## 概述

![image-20220726103422931](https://s2.loli.net/2022/07/26/e5I4HkRsF7calqN.png)

1. 在实验1中，你将实现一个流重组器——一个将字节流的小块（称为子串，或段）按正确顺序组装成连续的字节流的模块。
2. 在实验2中，你将实现TCP中处理入站字节流的部分：`TCPReceiver`。这需要考虑TCP如何表示每个字节在数据流中的位置，即所谓的序列号。`TCPReceiver`负责告诉发送方(a)它已经成功地组装了多少入站字节流（这被称为 “确认”）和(b)发送方现在允许再发送多少字节（”流量控制”）。
3. 在实验3中，你将实现TCP中处理出站字节流的部分：`TCPSender`。当发送方怀疑它所发送的一个段在途中丢失了，并没有到达接收方时，它应该如何反应？它应该在什么时候再次尝试并重新传输一个丢失的段？
4. 在实验4中，你将结合你在前面几个实验中的工作，创建一个工作的TCP实现：一个包含`TCPSender`和`TCPReceiver`的`TCPConnection`。你将用它来与世界各地的真实服务器对话。

## 目标

在本实验和下一个实验中，你将实现一个TCP接收器：接收数据报并将其转化为可靠的字节流供用户读取的模块（就像你的`webget`程序从实验0中的`webserver`读取字节流一样）。

TCP发送方将其字节流分为短段（每个子串不超过1,460字节），以便它们各自在一个数据报中。但网络可能会对这些数据报进行重新排序，或丢弃它们，或多次传送它们。接收方必须将这些片段重新组合成它们开始时的连续的字节流。

在本实验中，你将编写负责重新组装的数据结构：一个`StreamReassembler`。它将接收子串（由一串字节和大数据流中该串的第一个字节的索引组成），并提供一个`ByteStream`，其中所有的数据都被正确排序。

## 解决方案

### 提供的接口

```cpp
 #ifndef SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
 #define SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
  
 #include "byte_stream.hh"
  
 #include <cstdint>
 #include <string>
  
 class StreamReassembler {
   private:
     // Your code here -- add private members as necessary.
  
     ByteStream _output;  
     size_t _capacity;    
  
   public:
     StreamReassembler(const size_t capacity);
  
     void push_substring(const std::string &data, const uint64_t index, const bool eof);
  
     const ByteStream &stream_out() const { return _output; }
     ByteStream &stream_out() { return _output; }
  
     size_t unassembled_bytes() const;
  
     bool empty() const;
 };
  
 #endif  // SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
```

* `push_substring`将接受到的字串写入连续字节流中

	* `data` 表示子串
	* `index` 指示 `data`中**第一个字节**的索引（按顺序排列）
	* `eof`表示数据的最后一个字节将是否是整个流中的最后一个字节

* `stream_out` 访问重组好的字节流`_output`

* `unassembled_bytes`已存储但尚未重组的子字符串中的字节数

	* 如果特定索引处的字节已被多次推送，则出于此功能的目的，它应该只计算一次

* `empty`内部状态是否为空（输出流除外）？

	* 如果没有子串等待重组就返回`true`

* `_output`重组好的顺序字节流

* `_capacity` 最大字节数

	* `push_substring` 方法将忽略字符串中任何会导致 `StreamReassembler` 超出其“容量”的部分
	* 是已经被重组字节流的（绿色部分）和`unassembled_bytes`为重组子串可使用的最大字节数（红色部分）
	* 不仅用于限制高内存占用，而且它还会起到流量控制的作用

	![image-20220726105716989](https://s2.loli.net/2022/07/26/sSVz1DmnZQorRYO.png)

* 蓝色：接收到的字节并且已经被读取的部分
* 绿色：接受到的字节已经被重组放在 `ByteStream`，但还没被读取的部分
* 红色：接受到的字节放在辅助存储空间里，等待重组的部分

### 实现

底层网络只提供”尽力而为”的数据报。可以丢失、重新排序、改变或重复的短数据报。

* 数据报可能在传输过程中丢失，假设丢失的序号为 3，那么序号在 3 后面的数据报就必须等 3 来了才能重组后发给`ByteStream`
* 数据报在传输过程中可能会乱序，那么需要在等待重组区域将乱序的数据报重新排序，然后重组发送给 `ByteStream`
* 数据报在传输过程中可能会改变，应该去做一个校验、纠错不过这个不是`StreamReassembler`该干的活
* 数据在传输过程中可能重复
	* 重复的数据可能出现在
		* 蓝色部分：已经从 `ByteStream`读取了
		* 绿色部分：已经被重组放在 `ByteStream`，但还没被读取的部分
		* 红色部分：放在辅助存储空间里，等待重组
	
	* 如果是完全重复（重叠），则直接去重
	* 如果是部分重复（重叠），则需要合并
	



上述情况可以通过 C++ 的`set`加上序号来处理，`set`自带排序和去重

增加`_reassembler` 和`_nextnum` 成员

* `_nextnum` 表示下一个推入 `ByteStream` 的字节序号

* `_reassembler`对应红色部分，在执行`push_substring`前后都应该保证是下面这样一个状态：
	* `_reassembler` 为空
	* `_reassembler`不为空
		* 内部有序、无重复 （这两个 `set` 自动保证）
		* 在`_reassembler`内部，`ByteStream`部分，已读部分均无重叠（如果有重叠，就需要去除重叠部分）
		* 如果`_reassembler`内最小序号必须  `> _nextnum`，等于意味着应当推入`ByteStream`，小于意味着存在重叠

```cpp
private:
	// Your code here -- add private members as necessary.

    ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes
    using substring = std::pair<uint64_t , std::string >;
    std::set<substring> _reassembler = {};  // 重组缓冲区，等待重组的数据
    size_t _nextnum = 0;    // 下一个推入 ByteStream 的字节序号
```

