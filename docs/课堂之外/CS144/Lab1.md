# Lab1 将子字符串拼接成一个字节流

## 概述

![image-20220726103422931](https://s2.loli.net/2022/07/26/e5I4HkRsF7calqN.png)

1. 在实验 1 中，你将实现一个流重组器——一个将字节流的小块（称为子串，或段）按正确顺序组装成连续的字节流的模块。
2. 在实验 2 中，你将实现 TCP 中处理入站字节流的部分：`TCPReceiver`。这需要考虑 TCP 如何表示每个字节在数据流中的位置，即所谓的序列号。`TCPReceiver`负责告诉发送方(a)它已经成功地组装了多少入站字节流（这被称为 “确认”）和(b)发送方现在允许再发送多少字节（”流量控制”）。
3. 在实验 3 中，你将实现 TCP 中处理出站字节流的部分：`TCPSender`。当发送方怀疑它所发送的一个段在途中丢失了，并没有到达接收方时，它应该如何反应？它应该在什么时候再次尝试并重新传输一个丢失的段？
4. 在实验 4 中，你将结合你在前面几个实验中的工作，创建一个工作的 TCP 实现：一个包含`TCPSender`和`TCPReceiver`的`TCPConnection`。你将用它来与世界各地的真实服务器对话。

## 目标

在本实验和下一个实验中，你将实现一个 TCP 接收器：接收数据报并将其转化为可靠的字节流供用户读取的模块（就像你的`webget`程序从实验 0 中的`webserver`读取字节流一样）。

TCP 发送方将其字节流分为短段（每个子串不超过 1,460 字节），以便它们各自在一个数据报中。但**网络可能会对这些数据报进行重新排序，或丢弃它们，或多次传送它们**。接收方必须将这些片段重新组合成它们开始时的连续的字节流。

在本实验中，您将编写负责此重组的数据结构：`StreamReassembler`。 它将接收由字节串组成的子字符串，以及该字符串在较大流中的**第一个字节的索引**。 流的每个字节都有自己的唯一索引，从零开始向上计数。 `StreamReassembler` 将拥有一个用于输出的 `ByteStream`：**一旦重组器知道流的下一个字节，它就会将其写入 `ByteStream`**。 所有者可以随时访问和读取 `ByteStream`。

## 解决方案

### 提供的接口

```c++
#ifndef SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
#define SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH

#include "byte_stream.hh"

#include <cstdint>
#include <string>

//! \brief A class that assembles a series of excerpts from a byte stream (possibly out of order,
//! possibly overlapping) into an in-order byte stream.
class StreamReassembler {
  private:
    // Your code here -- add private members as necessary.

    ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes

  public:
    //! \brief Construct a `StreamReassembler` that will store up to `capacity` bytes.
    //! \note This capacity limits both the bytes that have been reassembled,
    //! and those that have not yet been reassembled.
    StreamReassembler(const size_t capacity);

    //! \brief Receive a substring and write any newly contiguous bytes into the stream.
    //!
    //! The StreamReassembler will stay within the memory limits of the `capacity`.
    //! Bytes that would exceed the capacity are silently discarded.
    //!
    //! \param data the substring
    //! \param index indicates the index (place in sequence) of the first byte in `data`
    //! \param eof the last byte of `data` will be the last byte in the entire stream
    void push_substring(const std::string &data, const uint64_t index, const bool eof);

    //! \name Access the reassembled byte stream
    //!@{
    const ByteStream &stream_out() const { return _output; }
    ByteStream &stream_out() { return _output; }
    //!@}

    //! The number of bytes in the substrings stored but not yet reassembled
    //!
    //! \note If the byte at a particular index has been pushed more than once, it
    //! should only be counted once for the purpose of this function.
    size_t unassembled_bytes() const;

    //! \brief Is the internal state empty (other than the output stream)?
    //! \returns `true` if no substrings are waiting to be assembled
    bool empty() const;
};

#endif  // SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH

```



### 设计

您的 `push substring` 方法将忽略字符串中任何会导致 `StreamReassembler` 超出其“容量”的部分：内存使用限制，即它允许存储的最大字节数。 这可以防止重组器使用无限量的内存，无论 TCP 发送方决定做什么。 我们在下图中说明了这一点。 “容量”是两者的上限：

1. 重新组装后的 `ByteStream` 中的字节数（下图绿色）
2. “未组装”子串可以使用的最大字节数（红色）

![image-20220726105716989](https://s2.loli.net/2022/07/26/sSVz1DmnZQorRYO.png)

* 蓝色：接收到的字节并且已经被读取的部分
* 绿色：接受到的字节已经被重组放在 `ByteStream`，但还没被读取的部分
* 红色：接受到的字节放在辅助存储空间里，等待重组的部分



个人对 `capacity` 的理解其实就是把 `Lab0` 里实现的 `ByteStream`  中空闲容量的部分拿来作为上图的红色区域（`auxiliary storage`）

![CS144LAB1](../../../../CS144LAB1.png)



FAQ

* **整个流中第一个字节的索引是多少？** 零

* **我的实施效率应该如何？** 请不要将此视为构建一个空间或时间效率极低的数据结构的挑战——这个数据结构将是您的 TCP 实现的基础。 一个大致的期望是每个新的 Lab 1 测试都可以在不到半秒的时间内完成。

* **应该如何处理不一致的子串？** 你可以假设它们不存在。 也就是说，您可以假设存在唯一的底层字节流，并且所有子字符串都是它的（准确）切片

  > 前面提到：TCP 送方将其字节流分为短段（每个子串不超过 1,460 字节），以便它们各自在一个数据报中。但**网络可能会对这些数据报进行重新排序，或丢弃它们，或多次传送它们**。接收方必须将这些片段重新组合成它们开始时的连续的字节流。


  但好像没提到数据被意外修改的情况，比如整个字节流是`abcdefghi`然后其中一个子串是`abcd` 结果在传输的过程中变成了`abcz`，对于这个情况我们的`StreamRessembler`可以认为不存在（或许会在其他 Lab 里进行处理吧）

* **我可以使用什么？** 您可以使用您认为有帮助的标准库的任何部分。 特别是，我们希望您至少使用一种数据结构。 

* **何时应将字节写入流？** 尽早。 字节不应该在流中的唯一情况是，当它之前有一个字节尚未被`push`时。

* **提供给 `push substring()` 函数的子字符串可以重叠吗？** 是的。

* **我是否需要将私有成员添加到 `StreamReassembler`？** 是的。 子字符串可能以任何顺序到达，因此您的数据结构必须“记住”子字符串，直到它们准备好放入流中——也就是说，直到它们之前的所有索引都被写入。 

* **我们的重组数据结构可以存储重叠子串吗？** 不，可以实现一个“接口正确”的重组器来存储重叠的子串。 但是允许重新组装器这样做会破坏“容量”作为内存限制的概念。 在评分时，我们会认为重叠子串的存储是一种风格违规。





### 代码实现



### 测试
