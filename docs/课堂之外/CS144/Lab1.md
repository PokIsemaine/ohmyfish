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

在本实验中，您将编写负责此重组的数据结构：`StreamReassembler`。 它将接收由字节串组成的子字符串，以及该字符串在较大流中的**第一个字节的索引**。 流的每个字节都有自己的唯一索引，从零开始向上计数。 `StreamReassembler` 将拥有一个用于输出的 `ByteStream`：**一旦重组器知道流的下一个字节，它就会将其写入** `ByteStream`。 所有者可以随时访问和读取 `ByteStream`。

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

![CS144LAB1.png](https://s2.loli.net/2022/09/04/K6k9DuTOsAr5plw.png)





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

* 主要采用 `map` 来实现，因为增删改查都是$O(NlogN)$ 复杂度，而且自带排序 + 去重功能

* 基本思路是按字符遍历输入，然后根据当前状态来分类讨论
	* 分类讨论两个区域的状态：第一个区域是已经排序重组的但还没被上层应用接受的区域（红色），第二个区域是到达但还没被排序重组的区域（绿色）。然后按照还有没有空间就会有四种状态（你可以简化合并），特别注意 `capacity` 的定义
		* 红满，绿没满（不存在）
		* 红满，绿满
		* 红没满，绿满
		* 红没满，绿没满
	* 然后根据两个区域不同状态你可以进行不同决策
	* 考虑什么是真正满了，什么是好像满了但还可以塞
	* 字符是有优先级的，如果当前字符就是你重组需要的下一个字符，考虑删除等待区域低优先级的字符来给高优先级字符腾出位置
* 尝试将模块用函数划分更加清晰，堆在一起难调试
* 获取一些标记可以尝试写成函数每次动态获取，因为你可能忘记改
* C++ map 如果要实现遍历 + 删除，注意迭代器失效问题
* 题面谜语人可能需要面向用例编程，卡壳了可以适当停几天回来重新调





`stream_reassembler.hh`

```c++
#ifndef SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
#define SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH

#include "byte_stream.hh"

#include <cstdint>
#include <string>
#include <map>

//! \brief A class that assembles a series of excerpts from a byte stream (possibly out of order,
//! possibly overlapping) into an in-order byte stream.
class StreamReassembler {
  private:
    // Your code here -- add private members as necessary.
    std::map<size_t, char> _storage = {};
    size_t _byteStreamWant = 0;
    int _eofIndex = -1;
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

    bool isFull() const;

    void helpAssembled(const char data, const bool eof);

    void helpStorage(const char data, const size_t index);

    int getFirstUnassembled() const;
};

#endif  // SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH

```

`stream_reassembler.cc`

```c++
#include "stream_reassembler.hh"
#include <string>
// Dummy implementation of a stream reassembler.

// For Lab 1, please replace with a real implementation that passes the
// automated checks run by `make check_lab1`.

// You will need to add private members to the class declaration in `stream_reassembler.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

StreamReassembler::StreamReassembler(const size_t capacity) : _output(capacity), _capacity(capacity) {}

void StreamReassembler::helpAssembled(const char data, const bool eof) {
    // 去除辅助等待重组区域中的和当前 data 重复的字符
    while(!_storage.empty() && getFirstUnassembled() <= static_cast<int>(_byteStreamWant)) {
        _storage.erase(_storage.begin());
    }

    if(isFull() && _output.remaining_capacity() == 0) {
        return ;
    } else if(_output.remaining_capacity() > 0) {
        if(isFull()) _storage.erase(std::prev(_storage.end())); // 腾出一个位置给要重组排序的字符
        ++_byteStreamWant;
        std::string writeData = string(1, data);
        _output.write(writeData);
        if(eof) _output.end_input();
    }
}

void StreamReassembler::helpStorage(const char data, const size_t index) {
    if(isFull() && _output.remaining_capacity() == 0) return ;
    _storage[index] = data;
}

//! \details This function accepts a substring (aka a segment) of bytes,
//! possibly out-of-order, from the logical stream, and assembles any newly
//! contiguous substrings and writes them into the output stream in order.
void StreamReassembler::push_substring(const string &data, const size_t index, const bool eof) {
    if(data.empty()) {
        if(eof) _output.end_input();
        return;
    }
    if(isFull() && _output.remaining_capacity() == 0) return ;

    if(eof)_eofIndex = index + data.size() - 1;
    for(size_t i = 0; i < data.size(); ++i) {
        size_t curIndex = index + i;
        if(curIndex < _byteStreamWant) continue;
        else if(curIndex == _byteStreamWant) {
            helpAssembled(data[i], static_cast<int>(curIndex) == _eofIndex);
        } else {
            helpStorage(data[i], curIndex);
        }
    }
	// 看看等待区域有没有可以进入 _output 的
    for(auto iter = _storage.begin(); iter != _storage.end();) {
        if(iter->first == _byteStreamWant && _output.remaining_capacity() > 0) {
            ++_byteStreamWant;
            std::string writeData = string(1, iter->second);
            _output.write(writeData);
            if(static_cast<int>(iter->first) == _eofIndex) _output.end_input();
            _storage.erase(iter++);
        } else {    // 首个不行就直接返回
            return ;
        }
    }
}

size_t StreamReassembler::unassembled_bytes() const { return _storage.size(); }

bool StreamReassembler::empty() const { return _storage.empty(); }

bool StreamReassembler::isFull() const { return _output.buffer_size() +_storage.size() >= _capacity; }

int StreamReassembler::getFirstUnassembled() const {
    if(_storage.empty()) return  -1;
    return (*_storage.begin()).first;
}

```



### 测试

`win`  和 `many` 的测试可以接受但有些慢，后期尝试进行一些优化。

```bash
Test project /home/zsl/CLionProjects/sponge/build
      Start 18: t_strm_reassem_single
 1/16 Test #18: t_strm_reassem_single ............   Passed    0.00 sec
      Start 19: t_strm_reassem_seq
 2/16 Test #19: t_strm_reassem_seq ...............   Passed    0.00 sec
      Start 20: t_strm_reassem_dup
 3/16 Test #20: t_strm_reassem_dup ...............   Passed    0.01 sec
      Start 21: t_strm_reassem_holes
 4/16 Test #21: t_strm_reassem_holes .............   Passed    0.00 sec
      Start 22: t_strm_reassem_many
 5/16 Test #22: t_strm_reassem_many ..............   Passed    0.63 sec
      Start 23: t_strm_reassem_overlapping
 6/16 Test #23: t_strm_reassem_overlapping .......   Passed    0.00 sec
      Start 24: t_strm_reassem_win
 7/16 Test #24: t_strm_reassem_win ...............   Passed    0.78 sec
      Start 25: t_strm_reassem_cap
 8/16 Test #25: t_strm_reassem_cap ...............   Passed    0.08 sec
      Start 26: t_byte_stream_construction
 9/16 Test #26: t_byte_stream_construction .......   Passed    0.00 sec
      Start 27: t_byte_stream_one_write
10/16 Test #27: t_byte_stream_one_write ..........   Passed    0.00 sec
      Start 28: t_byte_stream_two_writes
11/16 Test #28: t_byte_stream_two_writes .........   Passed    0.00 sec
      Start 29: t_byte_stream_capacity
12/16 Test #29: t_byte_stream_capacity ...........   Passed    0.27 sec
      Start 30: t_byte_stream_many_writes
13/16 Test #30: t_byte_stream_many_writes ........   Passed    0.01 sec
      Start 53: t_address_dt
14/16 Test #53: t_address_dt .....................   Passed    0.01 sec
      Start 54: t_parser_dt
15/16 Test #54: t_parser_dt ......................   Passed    0.00 sec
      Start 55: t_socket_dt
16/16 Test #55: t_socket_dt ......................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 16

Total Test time (real) =   1.81 sec
[100%] Built target check_lab1

```

