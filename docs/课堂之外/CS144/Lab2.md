# Lab2 实现 TCP 接收方

## 概述

在 Lab 0 中，您实现了流控制字节流 (ByteStream) 的抽象。在实验 1 中，您创建了一个 StreamReassembler，它接受一系列子字符串，所有子字符串都从同一字节流中摘录，并将它们重新组合回原始流。

这些模块将证明在您的 TCP 实现中很有用，但它们中的任何内容都不是特定于传输控制协议的细节。现在情况发生了变化。在实验 2 中，您将实现 TCPReceiver，它是处理传入字节流的 TCP 实现的一部分。

 **TCP Receiver 在传入的 TCP 段（通过 Internet 传输的数据报的有效负载）和传入的字节流之间进行转换**。这是上一个实验室的图表。 

**TCP Receiver 从 Internet 接收段（通过 `segment_received()` 方法）并将它们转换为对 StreamReassembler 的调用**，最终写入传入的 ByteStream。应用程序从这个 ByteStream 中读取，就像您在 Lab 0 中通过从 TCPSocket 读取一样。

![image-20220726103422931](https://s2.loli.net/2022/07/26/e5I4HkRsF7calqN.png)

除了写入传入流之外，TCPReceiver 还负责告诉发送者两件事

1. “第一个未组装”(`first unassembled`) 字节的索引，称为“确认号”(`ackno`) 。 这是接收方需要从发送方那得到的第一个字节。

2. “第一个未组装”(`first unassembled`)指标与“第一个不可接受”(`first unacceptable`)指标之间的距离。 这称为“窗口大小”(`window size`)。

![image-20220726105716989](https://s2.loli.net/2022/07/26/sSVz1DmnZQorRYO.png)







ackno 和窗口大小一起描述了接收方的窗口：允许 TCP 发送方发送的一系列索引。 使用该窗口，接收者可以控制传入数据的流向，使发送者限制它发送的数量，直到接收者准备好接收更多数据。 **我们有时将 ackno 称为窗口的“左边缘”（TCPReceiver 感兴趣的最小索引），将 ackno 窗口大小称为“右边缘”（刚好超出 TCPReceiver 感兴趣的最大索引）。** 

当您编写 StreamReassembler 和 ByteStream 时，您已经完成了实现 TCPReceiver 所涉及的大部分算法工作； 这个实验是关于将这些通用类连接到 TCP 的细节。 最困难的部分将涉及思考 TCP 将如何表示每个字节在流中的位置——称为“序列号”。



## 设计

TCP 是一种通过不可靠的数据报可靠地传送一对流控制字节流（每个方向一个）的协议。 两方参与 TCP 连接，每一方同时充当（其自己的传出字节流的）“发送者”和（传入字节流的）“接收者”。 两方被称为连接的“端点”或“对等点”。 

本周，您将实现 TCP 的“接收器”部分，负责接收 TCP 段（实际的数据报有效负载），重新组装字节流（包括其结束，何时发生），并确定应该发回的信号 发送方进行确认和流量控制。



> 为什么要这样做？ 这些信号对于 TCP 在不可靠的数据报网络上提供流控制的可靠字节流服务的能力至关重要。 在 TCP 中，确认意味着“接收方需要的下一个字节的索引是什么，以便它可以重组更多的 ByteStream？” 这告诉发送者它需要发送或重新发送哪些字节。 流量控制的意思是，“接收方有兴趣并愿意接收什么范围的索引？” （通常作为其剩余容量的函数）。 这告诉发送方允许发送多少。

### 在 64 位索引和 32 位序列号之间转换

作为热身，我们需要实现 TCP 表示索引的方式。上周您创建了一个 StreamReassembler，它重新组合子字符串，其中每个单独的字节都有一个 64 位的流索引，流中的第一个字节总是索引为零 (`stream index`)。 64 位索引足够大，我们可以将其视为永不溢出。**然而，在 TCP 标头中，空间是宝贵的，流中每个字节的索引不是用 64 位索引表示，而是用 32-位“序列号”或“seqno”。**这增加了三个复杂性： 

1. **您的实现需要规划 32 位整数来回绕**。 TCP 中的流可以任意长——可以通过 TCP 发送的 ByteStream 的长度没有限制。但是 $2^{32}$ 字节只有 4 GiB，并没有那么大。一旦 32 位序列号计数到 $2^{32} - 1$，流中的下一个字节的序列号将为零。 
2. **TCP 序列号从一个随机值开始**：为了提高安全性并避免被属于同一端点之间较早连接的旧段混淆，TCP 尝试确保序列号不能被猜测并且不太可能重复。所以流的序列号不是从零开始的。流中的第一个序列号是一个随机的 32 位数字，称为初始序列号 (ISN)。这是表示 SYN（流的开始）的序列号。其余序列号在此之后正常运行：数据的第一个字节将具有 $ISN +1$（模 $2^{32}$）的序列号，第二个字节将具有 $ISN + 2$（模 $2^{32}$）等。 
3. **逻辑开始和结尾各占一个序号**：TCP 除了保证接收到所有字节的数据外，还要确保可靠地接收到流的开头和结尾。因此，在 TCP 中，SYN (beginning-of-stream) 和 FIN (end-of-stream) 控制标志被分配了序列号。这些中的每一个都占用一个序列号。 （SYN 标志占用的序号就是 ISN。）流中的每个数据字节也占用一个序号。**请记住，SYN 和 FIN 不是流本身的一部分，也不是“字节”——它们代表字节流本身的开始和结束**。

这些序列号 (seqnos) 在每个 TCP 段的标头中传输。 （同样，有两个流——每个方向一个。每个流都有单独的序列号和不同的随机 ISN。）有时谈论“绝对序列号”的概念也很有帮助（它总是从零开始） 并且不换行），以及关于“流索引”（您已经在 StreamReassembler 中使用的内容：流中每个字节的索引，从零开始）。

为了使这些区别具体化，请考虑仅包含三个字母字符串“cat”的字节流。 如果 SYN 恰好有 seqno $2^{32} - 2$，那么每个字节的 seqnos、绝对 seqnos 和流索引是：

![image.png](https://s2.loli.net/2022/09/05/xUoPjOdlKYWLIny.png)

该图显示了 TCP 中涉及的三种不同类型的索引：

![image.png](https://s2.loli.net/2022/09/05/efgjiLqSVIaNH7p.png)

绝对序列号和流索引之间的转换很容易——只需加或减一个。 不幸的是，序列号和绝对序列号之间的转换有点困难，混淆两者会产生棘手的错误。 为了系统地防止这些错误，我们将使用自定义类型表示序列号：WrappingInt32，并编写它与绝对序列号（用 uint64 t 表示）之间的转换。 WrappingInt32 是包装器类型的一个示例：一种包含内部类型（在本例中为 uint32 t）但提供一组不同函数的类型

我们已经为您定义了类型并提供了一些辅助函数（请参阅 `wrapping integers.hh`），但是您将在 `wrapping_integers.cc` 中实现转换：
1. `WrappingInt32 wrap(uint64 t n, WrappingInt32 isn) `
    转换绝对 seqno → seqno。给定一个绝对序列号 (n) 和一个初始序列号 (isn)，生成 n 的（相对）序列号。
2. `uint64 t unwrap(WrappingInt32 n, WrappingInt32 is, uint64 t checkpoint)`
    转换 seqno → 绝对 seqno。给定一个序列号 (n)，初始序列号 (isn) 和绝对检查点序列号，计算对应于最接近检查点的 n 的绝对序列号。
    注意：需要一个检查点，因为任何给定的 seqno 对应于许多绝对序列号。例如。如果 ISN 为零，则 seqno “17” 对应于绝对 seqno 17，还有 $2^{32} + 17$，或 $2^{33} + 17$，或 $2^{34} + 17$ 等。检查点有助于解决歧义：这是一个绝对的 seqno，这个类的用户知道是“在球场上”的正确答案。**在您的 TCP 实现中，您将使用最后一个的索引重新组装的字节作为检查点（`the index of the last reassembled byte as checkpoint`)。**
    提示：最干净/最简单的实现将使用提供的辅助函数 `wrapping_integers.hh`。 wrap/unwrap 操作应该保留偏移量，两个相差 17 的 seqno 将对应于两个也相差 17 的绝对 seqno。

您可以通过运行 WrappingInt32 测试来测试您的实现。从构建目录，运行 `ctest -R wrap`

#### 提供的接口

```c++
#ifndef SPONGE_LIBSPONGE_WRAPPING_INTEGERS_HH
#define SPONGE_LIBSPONGE_WRAPPING_INTEGERS_HH

#include <cstdint>
#include <ostream>

//! \brief A 32-bit integer, expressed relative to an arbitrary initial sequence number (ISN)
//! \note This is used to express TCP sequence numbers (seqno) and acknowledgment numbers (ackno)
class WrappingInt32 {
  private:
    uint32_t _raw_value;  //!< The raw 32-bit stored integer

  public:
    //! Construct from a raw 32-bit unsigned integer
    explicit WrappingInt32(uint32_t raw_value) : _raw_value(raw_value) {}

    uint32_t raw_value() const { return _raw_value; }  //!< Access raw stored value
};

//! Transform a 64-bit absolute sequence number (zero-indexed) into a 32-bit relative sequence number
//! \param n the absolute sequence number
//! \param isn the initial sequence number
//! \returns the relative sequence number
WrappingInt32 wrap(uint64_t n, WrappingInt32 isn);

//! Transform a 32-bit relative sequence number into a 64-bit absolute sequence number (zero-indexed)
//! \param n The relative sequence number
//! \param isn The initial sequence number
//! \param checkpoint A recent absolute sequence number
//! \returns the absolute sequence number that wraps to `n` and is closest to `checkpoint`
//!
//! \note Each of the two streams of the TCP connection has its own ISN. One stream
//! runs from the local TCPSender to the remote TCPReceiver and has one ISN,
//! and the other stream runs from the remote TCPSender to the local TCPReceiver and
//! has a different ISN.
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint);

//! \name Helper functions
//!@{

//! \brief The offset of `a` relative to `b`
//! \param b the starting point
//! \param a the ending point
//! \returns the number of increments needed to get from `b` to `a`,
//! negative if the number of decrements needed is less than or equal to
//! the number of increments
inline int32_t operator-(WrappingInt32 a, WrappingInt32 b) { return a.raw_value() - b.raw_value(); }

//! \brief Whether the two integers are equal.
inline bool operator==(WrappingInt32 a, WrappingInt32 b) { return a.raw_value() == b.raw_value(); }

//! \brief Whether the two integers are not equal.
inline bool operator!=(WrappingInt32 a, WrappingInt32 b) { return !(a == b); }

//! \brief Serializes the wrapping integer, `a`.
inline std::ostream &operator<<(std::ostream &os, WrappingInt32 a) { return os << a.raw_value(); }

//! \brief The point `b` steps past `a`.
inline WrappingInt32 operator+(WrappingInt32 a, uint32_t b) { return WrappingInt32{a.raw_value() + b}; }

//! \brief The point `b` steps before `a`.
inline WrappingInt32 operator-(WrappingInt32 a, uint32_t b) { return a + -b; }
//!@}

#endif  // SPONGE_LIBSPONGE_WRAPPING_INTEGERS_HH

```



### 实现 TCP 接收方

恭喜您获得了正确的包装和展开逻辑！ 如果可以的话，我们会和你握手。 在本实验的其余部分，您将实现 TCPReceiver。 它将 (1) 从其对等方接收段，(2) 使用您的 StreamReassembler 重新组装 ByteStream，以及 (3) 计算确认号 (ackno) 和窗口大小。 确认和窗口大小最终将在传出段中传输回对等方。 

首先，请查看 TCP 段的格式。 这是两个端点相互发送的消息； 它是较低级别数据报的有效负载。 非灰色字段表示本实验中感兴趣的信息：序列号、有效负载以及 SYN 和 FIN 标志。 这些是发送者写入的字段，接收者读取和操作的字段。

![image.png](https://s2.loli.net/2022/09/07/BDPEA1Upct9KMoz.png)

关于 TCPSegment class 可以看下面这两个文档

https://cs144.github.io/doc/lab2/class_t_c_p_segment.html

https://cs144.github.io/doc/lab2/struct_t_c_p_header.html

 可以关注一下[length_in_sequence_space()](https://cs144.github.io/doc/lab2/class_t_c_p_segment.html#a41eb3ff25fee485849fd38eb31c990d6)方法，该方法计算一个段占用多少个序列号（包括 SYN 和 FIN 标志每个占用一个序列号的事实，以及有效负载的每个字节）



#### 提供的接口

```c++
#ifndef SPONGE_LIBSPONGE_TCP_RECEIVER_HH
#define SPONGE_LIBSPONGE_TCP_RECEIVER_HH

#include "byte_stream.hh"
#include "stream_reassembler.hh"
#include "tcp_segment.hh"
#include "wrapping_integers.hh"

#include <optional>

//! \brief The "receiver" part of a TCP implementation.

//! Receives and reassembles segments into a ByteStream, and computes
//! the acknowledgment number and window size to advertise back to the
//! remote TCPSender.
class TCPReceiver {
    //! Our data structure for re-assembling bytes.
    StreamReassembler _reassembler;

    //! The maximum number of bytes we'll store.
    size_t _capacity;

  public:
    //! \brief Construct a TCP receiver
    //!
    //! \param capacity the maximum number of bytes that the receiver will
    //!                 store in its buffers at any give time.
    TCPReceiver(const size_t capacity) : _reassembler(capacity), _capacity(capacity) {}

    //! \name Accessors to provide feedback to the remote TCPSender
    //!@{

    //! \brief The ackno that should be sent to the peer
    //! \returns empty if no SYN has been received
    //!
    //! This is the beginning of the receiver's window, or in other words, the sequence number
    //! of the first byte in the stream that the receiver hasn't received.
    std::optional<WrappingInt32> ackno() const;

    //! \brief The window size that should be sent to the peer
    //!
    //! Operationally: the capacity minus the number of bytes that the
    //! TCPReceiver is holding in its byte stream (those that have been
    //! reassembled, but not consumed).
    //!
    //! Formally: the difference between (a) the sequence number of
    //! the first byte that falls after the window (and will not be
    //! accepted by the receiver) and (b) the sequence number of the
    //! beginning of the window (the ackno).
    size_t window_size() const;
    //!@}

    //! \brief number of bytes stored but not yet reassembled
    size_t unassembled_bytes() const { return _reassembler.unassembled_bytes(); }

    //! \brief handle an inbound segment
    void segment_received(const TCPSegment &seg);

    //! \name "Output" interface for the reader
    //!@{
    ByteStream &stream_out() { return _reassembler.stream_out(); }
    const ByteStream &stream_out() const { return _reassembler.stream_out(); }
    //!@}
};

#endif  // SPONGE_LIBSPONGE_TCP_RECEIVER_HH

```



**`segment_received() `**

这是主要的主力方法。 TCPReceiver::segment_received() 将在每次从对等端接收到新段时调用。 

此方法需要： 

* 如有必要，设置初始序列号。 设置了 SYN 标志的第一个到达段的序号是初始序号。 您需要跟踪它以便继续在 32 位包装的 seqnos 之间进行转换

* 将任何数据或流结束标记推送到 StreamReassembler。 如果在 TCPSegment 的标头中设置了 FIN 标志，则意味着负载的最后一个字节是整个流的最后一个字节。 请记住，StreamReassembler 期望流索引从零开始； 你将不得不解开 seqnos 来产生这些。

**`ackno()`**

返回一个`optional<WrappingInt32>`，其中包含接收者尚不知道的第一个字节的序列号(`seqno`)。 这是窗口的左边缘：接收者有兴趣接收的第一个字节。 如果尚未设置 ISN，则返回一个空的`optional`。

**`window_size()`**

返回“first unassembled”索引（对应于 ackno 的索引）和“first unacceptable”索引之间的距离。



### TCPReceiver 在连接生命周期内的演变

在 TCP 连接过程中，您的 TCPReceiver 将经历一系列状态：从等待 SYN（带有空 ackno）到正在进行的流，再到已完成的流，这意味着输入已在 ByteStream 上结束。 测试套件将检查您的 TCPReceiver 是否正确处理传入的 TCPSegments 并通过这些状态演变，如下所示。 （在实验 4 之前，您不必担心错误状态或 RST 标志。）

![image.png](https://s2.loli.net/2022/09/08/8w7qfEXTa2xyZhm.png)

## 代码实现

### 实现 WrappingInt32

![image.png](https://s2.loli.net/2022/09/08/Do1k9PR2sSTriI4.png)



#### 注意

* 想清楚无符号整数作差可能溢出的时候，你是要自然溢出取模还是要像有符号数那样取绝对值
* 先找到 checkpoint 所在段上下边界（可以模仿操作系统里的页对齐位运算），可能的位置
	* 当前段前一个位置
	* 当前段
	* 当前段后一个

* 步骤
	1. 绿色 checkpoint 找边界
	2. 找 三个可能的蓝色的位置 
	3. 去除蓝色位置的偏移得到三个红色位置
	4. 计算红色位置到绿色 checkpoint 的距离取最小的

#### 代码

`wrapping_integers.cc`

```c++
#include "wrapping_integers.hh"
#include <cmath>
// Dummy implementation of a 32-bit wrapping integer

// For Lab 2, please replace with a real implementation that passes the
// automated checks run by `make check_lab2`.

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

//! Transform an "absolute" 64-bit sequence number (zero-indexed) into a WrappingInt32
//! \param n The input absolute 64-bit sequence number
//! \param isn The initial sequence number
WrappingInt32 wrap(uint64_t n, WrappingInt32 isn) {
    // 无符号整数自然溢出
    return WrappingInt32{static_cast<uint32_t >(n) + isn.raw_value()};
}

//! Transform a WrappingInt32 into an "absolute" 64-bit sequence number (zero-indexed)
//! \param n The relative sequence number
//! \param isn The initial sequence number
//! \param checkpoint A recent absolute 64-bit sequence number
//! \returns the 64-bit sequence number that wraps to `n` and is closest to `checkpoint`
//!
//! \note Each of the two streams of the TCP connection has its own ISN. One stream
//! runs from the local TCPSender to the remote TCPReceiver and has one ISN,
//! and the other stream runs from the remote TCPSender to the local TCPReceiver and
//! has a different ISN.
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint) {
    int offset = -n.raw_value() + isn.raw_value();
    constexpr uint64_t RANGE = 1L << 32;
    
    uint64_t lower = (checkpoint) & ~(RANGE - 1);
    uint64_t upper = ((checkpoint + RANGE - 1) & ~(RANGE -1));
    
    uint64_t preSeg = lower - offset ;
    uint64_t nextSeg = upper - offset;
    uint64_t next2Seg = (upper + RANGE) - offset;
    
    uint64_t dis1 = checkpoint >= preSeg ? checkpoint - preSeg : preSeg - checkpoint;
    uint64_t dis2 = checkpoint >= nextSeg ? checkpoint - nextSeg : nextSeg - checkpoint;
    uint64_t dis3 = checkpoint >= next2Seg ? checkpoint - next2Seg : next2Seg - checkpoint;
    uint64_t minDis = min(min(dis1, dis2), dis3);
    if(minDis == dis1) return preSeg;
    else if(minDis == dis2) return nextSeg;
    else return next2Seg;
}

```



### 实现 TCP Receiver

#### segment_received

注意以下几点

* TCP Receiver 的 `segment_reveived` 要调用 StreamReassembler 的 `push_substring`，这其中的参数涉及到从 `32bit seqno` 到 `64bit stream index` 的转换，其中还隐含了 `32bit seqno` 到 `64bit absolute seqno` 的转换。在调用之前要完成两步转换
* 在 TCP Receiver 接收到带有 syn 标志的 TCP Segement 之前（LISTEN 状态），收到的数据应该丢弃
* checkpoint 要求 
	* a recent absolute 64 bit sequence number
	* the index of the last reassembled byte as checkpoint


![image.png](https://s2.loli.net/2022/09/08/6AYfwOUNKu4m5qJ.png)

#### ackno

接收方期望的发送方发送的字节

1. `stream index` -->`64bit absolute seqno` 
2. `64bit absolute seqno` --> `32bit seqno`

#### window size

容量减去 StringReassembler 中已经重组好的部分

![image.png](https://s2.loli.net/2022/09/08/HW9cKfezyJtadgM.png)



#### 代码

`tcp_receiver.hh`

```c++
#ifndef SPONGE_LIBSPONGE_TCP_RECEIVER_HH
#define SPONGE_LIBSPONGE_TCP_RECEIVER_HH

#include "byte_stream.hh"
#include "stream_reassembler.hh"
#include "tcp_segment.hh"
#include "wrapping_integers.hh"

#include <optional>

//! \brief The "receiver" part of a TCP implementation.

//! Receives and reassembles segments into a ByteStream, and computes
//! the acknowledgment number and window size to advertise back to the
//! remote TCPSender.
class TCPReceiver {
    WrappingInt32 _isn {0};
    bool _hasSyn = false;

    //! Our data structure for re-assembling bytes.
    StreamReassembler _reassembler;

    //! The maximum number of bytes we'll store.
    size_t _capacity;

  public:
    //! \brief Construct a TCP receiver
    //!
    //! \param capacity the maximum number of bytes that the receiver will
    //!                 store in its buffers at any give time.
    TCPReceiver(const size_t capacity) : _reassembler(capacity), _capacity(capacity) {}

    //! \name Accessors to provide feedback to the remote TCPSender
    //!@{

    //! \brief The ackno that should be sent to the peer
    //! \returns empty if no SYN has been received
    //!
    //! This is the beginning of the receiver's window, or in other words, the sequence number
    //! of the first byte in the stream that the receiver hasn't received.
    std::optional<WrappingInt32> ackno() const;

    //! \brief The window size that should be sent to the peer
    //!
    //! Operationally: the capacity minus the number of bytes that the
    //! TCPReceiver is holding in its byte stream (those that have been
    //! reassembled, but not consumed).
    //!
    //! Formally: the difference between (a) the sequence number of
    //! the first byte that falls after the window (and will not be
    //! accepted by the receiver) and (b) the sequence number of the
    //! beginning of the window (the ackno).
    size_t window_size() const;
    //!@}

    //! \brief number of bytes stored but not yet reassembled
    size_t unassembled_bytes() const { return _reassembler.unassembled_bytes(); }

    //! \brief handle an inbound segment
    void segment_received(const TCPSegment &seg);

    //! \name "Output" interface for the reader
    //!@{
    ByteStream &stream_out() { return _reassembler.stream_out(); }
    const ByteStream &stream_out() const { return _reassembler.stream_out(); }
    //!@}
};

#endif  // SPONGE_LIBSPONGE_TCP_RECEIVER_HH

```



`tcp_receiver.cc`

```c++
#include "tcp_receiver.hh"

// Dummy implementation of a TCP receiver

// For Lab 2, please replace with a real implementation that passes the
// automated checks run by `make check_lab2`.

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

void TCPReceiver::segment_received(const TCPSegment &seg) {
    const TCPHeader &head = seg.header();
    // CHECK LISTEN
    if(!_hasSyn) {
        if(head.syn) {
            _hasSyn = true;
            _isn = head.seqno;
        } else {
            return ;
        }
    }

    // 32bit seqno --> 64bit absolute seqno

    // the index of the last reassembled byte (stream index)
    // A recent absolute sequence number (absolute seqno)
    // absolute seqno = stream index + 1
    // bytes_written 得到下一个期望的 stream index, 转为 64bit absolute = stream index + 1
    // the last reassembled byte 对应 bytes_written - 1
    uint64_t checkPoint = _reassembler.stream_out().bytes_written();

    uint64_t absSeqno = unwrap(head.seqno, _isn, checkPoint);

    // 64bit absolute seqno --> 64bit stream index

    uint64_t streamIndex = absSeqno - 1 + head.syn;

    // call push_substring
    _reassembler.push_substring(seg.payload().copy(), streamIndex, head.fin);
}

optional<WrappingInt32> TCPReceiver::ackno() const {
    if(!_hasSyn) return nullopt;        // LISTEN
    // bytes_written 得到下一个期望的 stream index, 转为 64bit absolute = stream index + 1
    uint64_t ackNo = _reassembler.stream_out().bytes_written() + 1;
    // 如果包含 FIN 那还需要再  + 1
    if(_reassembler.stream_out().input_ended()) ++ackNo;
    // 64bit absolute seqno --> 32bit seqno
    return wrap(ackNo, _isn);
}

size_t TCPReceiver::window_size() const { return _capacity - _reassembler.stream_out().buffer_size(); }

```



## 测试

### 测试 WrappingInt32

```bash
Test project /home/zsl/CLionProjects/sponge/build
    Start 1: t_wrapping_ints_cmp
1/4 Test #1: t_wrapping_ints_cmp ..............   Passed    0.01 sec
    Start 2: t_wrapping_ints_unwrap
2/4 Test #2: t_wrapping_ints_unwrap ...........   Passed    0.00 sec
    Start 3: t_wrapping_ints_wrap
3/4 Test #3: t_wrapping_ints_wrap .............   Passed    0.00 sec
    Start 4: t_wrapping_ints_roundtrip
4/4 Test #4: t_wrapping_ints_roundtrip ........   Passed    0.16 sec

100% tests passed, 0 tests failed out of 4

Total Test time (real) =   0.37 sec
```



### 测试 TCPReceiver

```c++
Test project /home/zsl/CLionProjects/sponge/build
      Start  1: t_wrapping_ints_cmp
 1/26 Test  #1: t_wrapping_ints_cmp ..............   Passed    0.01 sec
      Start  2: t_wrapping_ints_unwrap
 2/26 Test  #2: t_wrapping_ints_unwrap ...........   Passed    0.00 sec
      Start  3: t_wrapping_ints_wrap
 3/26 Test  #3: t_wrapping_ints_wrap .............   Passed    0.00 sec
      Start  4: t_wrapping_ints_roundtrip
 4/26 Test  #4: t_wrapping_ints_roundtrip ........   Passed    0.12 sec
      Start  5: t_recv_connect
 5/26 Test  #5: t_recv_connect ...................   Passed    0.00 sec
      Start  6: t_recv_transmit
 6/26 Test  #6: t_recv_transmit ..................   Passed    0.03 sec
      Start  7: t_recv_window
 7/26 Test  #7: t_recv_window ....................   Passed    0.00 sec
      Start  8: t_recv_reorder
 8/26 Test  #8: t_recv_reorder ...................   Passed    0.00 sec
      Start  9: t_recv_close
 9/26 Test  #9: t_recv_close .....................   Passed    0.00 sec
      Start 10: t_recv_special
10/26 Test #10: t_recv_special ...................   Passed    0.00 sec
      Start 18: t_strm_reassem_single
11/26 Test #18: t_strm_reassem_single ............   Passed    0.00 sec
      Start 19: t_strm_reassem_seq
12/26 Test #19: t_strm_reassem_seq ...............   Passed    0.00 sec
      Start 20: t_strm_reassem_dup
13/26 Test #20: t_strm_reassem_dup ...............   Passed    0.01 sec
      Start 21: t_strm_reassem_holes
14/26 Test #21: t_strm_reassem_holes .............   Passed    0.00 sec
      Start 22: t_strm_reassem_many
15/26 Test #22: t_strm_reassem_many ..............   Passed    0.55 sec
      Start 23: t_strm_reassem_overlapping
16/26 Test #23: t_strm_reassem_overlapping .......   Passed    0.00 sec
      Start 24: t_strm_reassem_win
17/26 Test #24: t_strm_reassem_win ...............   Passed    0.68 sec
      Start 25: t_strm_reassem_cap
18/26 Test #25: t_strm_reassem_cap ...............   Passed    0.05 sec
      Start 26: t_byte_stream_construction
19/26 Test #26: t_byte_stream_construction .......   Passed    0.00 sec
      Start 27: t_byte_stream_one_write
20/26 Test #27: t_byte_stream_one_write ..........   Passed    0.00 sec
      Start 28: t_byte_stream_two_writes
21/26 Test #28: t_byte_stream_two_writes .........   Passed    0.00 sec
      Start 29: t_byte_stream_capacity
22/26 Test #29: t_byte_stream_capacity ...........   Passed    0.20 sec
      Start 30: t_byte_stream_many_writes
23/26 Test #30: t_byte_stream_many_writes ........   Passed    0.01 sec
      Start 53: t_address_dt
24/26 Test #53: t_address_dt .....................   Passed    0.03 sec
      Start 54: t_parser_dt
25/26 Test #54: t_parser_dt ......................   Passed    0.00 sec
      Start 55: t_socket_dt
26/26 Test #55: t_socket_dt ......................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 26

Total Test time (real) =   1.74 sec
[100%] Built target check_lab2
```

