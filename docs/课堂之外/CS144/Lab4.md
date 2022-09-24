# Lab4 实现 TCP Connection

## 概述

在 Lab 0 中，您实现了流控制字节流 (ByteStream) 的抽象。 在实验 1、2 和 3 中，您实现了在该抽象和 Internet 提供的抽象之间双向转换的工具：不可靠的数据报。

 现在，在实验 4 中，您将创建名为 TCPConnection 的总体模块，它结合了 TCPSender 和 TCPReceiver 并处理连接的全局管理。 连接的 TCP 段可以封装到用户（TCP-in-UDP）或 Internet (TCP/IP) datagrams，让您的代码与 Internet 上使用相同 TCP/IP  的数十亿台计算机通信。图片 1 再次展示了总体设计。



![image-20220726103422931](https://s2.loli.net/2022/07/26/e5I4HkRsF7calqN.png)

一个简短的注意事项：TCPConnection 大多只是结合了您在早期实验中实现的发送器和接收器模块——TCPConnection 本身可以用不到 100 行代码来实现。 如果您的发送者和接收者很健壮，这将是一个简短的实验。 如果没有，您可能需要在测试失败消息的帮助下花时间进行调试。 （我们不鼓励你尝试阅读测试源代码，除非是万不得已。）基于去年学生的双峰体验，我们强烈建议你早点开始，直到前一天晚上才离开这个实验室 最后期限。



## 设计

## TCP connection

本周，您将完成构建一个有效的 TCP 实现。 你已经完成了大部分工作：你已经实现了发送者和接收者。 你本周的工作是将它们“连接”在一起成为一个对象（一个 TCPConnection），并处理一些全局性的连接任务。 

回想一下：TCP 可靠地传送一对流控制的字节流，每个方向一个。 两方参与 TCP 连接，每一方同时充当（其自己的出站字节流的）“发送者”和（入站字节流的）“接收者”：

![image.png](https://s2.loli.net/2022/09/13/qy9sbAz8BWP3F7h.png)

两方（上图中的“A”和“B”）被称为连接的“端点”或“对等点”。 您的 `TCPConnection` 充当对等方之一。 它负责接收和发送段，确保发送方和接//收方被告知并有机会为他们关心的传入和传出段的字段做出贡献。 以下是 `TCPConnection` 必须遵循的基本规则：

**接收段**

如图 1 所示，当调用它的段接收方法时，`TCPConnection` 从 `Internet` 接收 `TCPSegments`。 发生这种情况时，`TCPConnection` 会查看该段并：

* 如果设置了 `RST`（重置）标志，则将输入和输出流都设置为错误状态并永久终止连接。否则它。
* 将段提供给 `TCPReceiver`，以便它可以检查传入段上它关心的字段：`seqno`、`SYN`、`payload` 和 `FIN`。 
* 如果设置了 `ACK` 标志，则告诉 `TCPSender` 它关心传入段的字段：`ackno` 和 `window_size`。 
* 如果输入段占用任何序列号，`TCPConnection` 确保至少发送一个段作为回复，以反映确认和窗口大小的更新。 
* 在 `TCPConnection` 的 `segment_received()` 方法中，您必须处理一种额外的特殊情况：响应“keep-alive”段。对等方可能会选择发送带有无效序列号的段，以查看您的 TCP 实现是否仍然有效（如果是，您当前的窗口是什么）。你的 `TCPConnection` 应该回复这些“keep-alives”，即使它们不占用任何序列号。实现它的代码如下所示：

```c++
if (_receiver.ackno().has_value() and (seg.length_in_sequence_space() == 0)
    and seg.header().seqno == _receiver.ackno().value() - 1) {
    _sender.send_empty_segment();
}
```

**发送段** 

`TCPConnection` 将通过 `Internet` 发送 `TCPSegments`： 

* **任何时候** `TCPSender` 将一个段推送到其传出队列，并在传出段上设置它负责的字段：（`seqno`、`SYN` 、`payload` 和 `FIN`）。 
* 在发送段之前，`TCPConnection` 将向 `TCPReceiver` 询问它在传出段上负责的字段：`ackno` 和 `window size`。 如果有 `ackno`，它将设置 `ACK` 标志和 `TCPSegment` 中的字段。

**当时间流逝**

 `TCPConnection` 有一个 `tick` 方法，操作系统会定期调用该方法。 发生这种情况时，`TCPConnection` 需要：

* 告诉 `TCPSender` 时间的流逝。
* 如果连续重传次数超过上限 `TCPConfig::MAX_RETX_ATTEMPTS`，则中止连接，并向对等方发送重置段（设置了 `rst` 标志的空段）。 
* 必要时干净地结束连接（请参阅第 5 节）。

因此，每个 TCPSegment 的整体结构如下所示，“sender-written”和“receiver-written”字段以不同颜色显示：

![image.png](https://s2.loli.net/2022/09/13/rKlvVTmbhYJuZq4.png)



TCPConnection 的完整接口在类文档中。 请花一些时间阅读此内容。 您的大部分实现将涉及将 TCPConnection 的公共 API“连接”到 TCPSender 和 TCPReceiver 中的适当例程。 尽可能地，您希望将任何繁重的工作推迟到您已经实现的发送方和接收方。 也就是说，并非所有事情都那么简单，并且有一些微妙之处涉及整体连接的“全局”行为。 最困难的部分将是决定何时完全终止 TCPConnection 并声明它不再“活动”。 以下是您需要处理的一些常见问题解答和边缘案例的详细信息。



### FAQs 和 特殊情况

**您期望有多少代码？** 

总的来说，我们预计实现（在 `tcp connection.cc` 中）总共需要大约 100-150 行代码。 完成后，测试套件将广泛测试您与您自己的实现以及 Linux 内核的 TCP 实现的互操作性。



**我应该如何开始？**

可能最好的开始方法是将一些“ordinary”方法连接到 `TCPSender` 和 `TCPReceiver` 中的适当调用。 这可能包括诸如`remaining_outbound_capacity()`、`bytes_in_flight()` 和 `unassembled_bytes()` 之类的东西。 然后您可以选择实现“writer”方法：`connect()`、`write() `和 `end_input_stream()`。 其中一些方法可能需要对出站 `ByteStream`（由 `TCPSender` 拥有）做一些事情并告诉 TCPSender。 您可以选择在完全实现每个方法之前开始运行测试套件（ `make check` ）； 测试失败消息可以为您提供有关下一步要解决的问题的线索或指南。



**应用程序如何从`inbound stream`中读取？**

`TCPConnection::inbound_stream()` 已经在头文件中实现。 您无需再做任何事情来支持应用程序读取。



**TCPConnection 是否需要任何花哨的数据结构或算法？**

不，真的没有。 繁重的工作全部由您已经实现的 TCPSender 和 TCPReceiver 完成。 这里的工作实际上只是将所有东西都连接起来，并处理一些挥之不去的连接范围内的微妙之处，这些细节不容易被发送者和接收者考虑在内。



**TCPConnection 是如何实际发送一个段的？**

与 TCPSender 类似——将其推送到分段输出队列。 就您的 TCPConnection 而言，请考虑将其推送到此队列后立即发送。 很快，所有者就会出现并弹出它（使用公共段 out() 访问器方法）并真正发送它。



**TCPConnection 如何了解时间的流逝？**

与 TCPSender 类似——tick() 方法将被定期调用。 请不要使用任何其他方式来告诉时间 - 滴答法是您唯一可以了解时间流逝的方法。 这使事情具有确定性和可测试性。



**如果传入的段设置了一个`RST`标志，TCPConnection 会做什么？**

此标志（“reset”）意味着连接立即死亡。 如果您收到带有 `RST` 的段，您应该在入站和出站 `ByteStreams` 上设置错误标志，并且任何随后对`TCPConnection::active()` 的调用都应该返回 false。



**我应该什么时候发送一个设置了一个`RST`标志的段？**

有两种情况你会想要中止整个连接： 

1. 如果发送者发送了太多连续的重传但没有成功（超过 `TCPConfig::MAX RETX ATTEMPTS`，即 8）。
2. 如果在连接仍处于活动状态时调用了 `TCPConnection` 析构函数（`active()` 返回 `true`）

发送一个带有 `RST` set 的段与接收一个段有类似的效果：连接已死并且不再 `active()`，并且两个 `ByteStreams` 都应该设置为错误状态。



**等等，但我甚至如何制作一个可以设置rst标志的段？序列号是什么？**

任何流出的段都需要有适当的序列号。你可以强制 你可以通过调用 `TCPSender` 的 `send_empty segment()` 方法，强迫它生成一个具有适当序列号的空段。它的 `send_empty_segment()` 方法来强制生成一个具有适当序列号的空段。或者你可以让它填满窗口（如果它有未完成的信息要发送，例如，生成 如果它有未完成的信息要发送，例如，来自流的字节或 `SYN/FIN`），调用它的 `fill_window()` 方法。



**`ACK`标志的作用是什么？不是一直有一个ackno吗？**

**几乎每个**TCPSegment都有一个ackno，并设置了ack标志。例外的情况是在连接的最开始，在接收方有任何需要确认的东西之前。
**在传出段中**，你要尽可能地设置ackno和ack标志。也就是说，只要TCPReceiver的ackno()方法返回一个 `std::optional<WrappingInt32>`，它有一个值，你可以用has value()测试。
**在传入的段中**，只有在ack字段被设置的情况下，你才会想看看ackno。设置。如果是这样，就把这个ackno（和窗口大小）给TCPSender。



**我如何解读这些 "状态 "名称（如 "流开始 "或 "流进行中"）？**

请看实验二和实验三讲义中的图表。
我们想再次强调，这些 "状态 "对测试和调试很有用，但是 我们并不要求你在代码中实现这些状态。你不需要 制作更多的状态变量来跟踪这些状态。状态 "只是一个 公共接口的功能。



**如果 TCP 接收器想要通告一个大于 `TCP Segment::header().win` 字段的窗口大小，我应该发送什么窗口大小？**

尽可能发送最大的价值。 您可能会发现 `std::numeric_limits` 类很有帮助。



**TCP 连接何时“完成”？ active() 什么时候可以返回 false？**

请看下一节



### TCP connection 的结束：共识需要的工作

TCP connection 的一项重要功能是决定 TCP 连接何时完全“完成”。 发生这种情况时，实现将其独占声明释放给本地端口号，停止发送确认以回复传入的段，将连接视为历史，并使其 active() 方法返回 false。有两种方式可以结束连接。在非正常关闭中，TCPConnection 发送或接收设置了 `RST` 标志的段。在这种情况下，出站和入站 ByteStream 都应该处于错误状态，并且 active() 可以立即返回 false。

干净关闭是我们如何“完成”（active() = false）而不会出现错误。这更复杂，但它是一件美好的事情，因为它尽可能确保两个 ByteStream 中的每一个都已完全可靠地传递给接收对等方。在下一节（第 5.1 节）中，我们给出了干净关闭发生时的实际结果，因此，如果您愿意，请随意跳过。

酷，你还在这里。由于两个将军问题，不可能保证两个对等点都能完全关闭，但 TCP 非常接近。就是这样。从一个对等点（一个 TCPConnection，我们将其称为“本地”对等点）的角度来看，在与“远程”对等点的连接中完全关闭有四个先决条件：

* Prereq #1 入站流已完全组装并已结束。 
* Prereq #2 出站流已由本地应用程序结束并完全发送（包括它结束的事实，即带有 `FIN` 的段）到远程对等点。 
* Prereq #3 出站流已被远程对等方完全确认。 
* Prereq #4 本地 TCPConnection 确信远程对等方可以满足先决条件 #3。 这是大脑弯曲的部分。 有两种替代方法可以发生这种情况：
	* 选项A：在两个流结束后徘徊。先决条件 #1 到 #3 为真，并且远程对等点似乎已获得本地对等点对整个流的确认。本地对等点并不确定这一点——TCP 不能可靠地传递确认（它不确认确认）。但是本地对等点非常确信远程对等点已经得到了它的确认，因为远程对等点似乎没有重新传输任何东西，而本地对等点已经等待了一段时间来确定。具体而言，当满足先决条件 #1 到 #3 并且自从本地对等方已收到来自远程对等方的任何段以来，连接已至少是初始重传超时（`cfg.rt_timeout`）的 10 倍。这在两个流完成后称为“延迟”，以确保远程对等方不会尝试重新传输我们需要确认的任何内容。这确实意味着 TCPConnection 需要保持活动一段时间，1 保持对本地端口号的独占声明，并可能发送 ack 作为响应到传入的段，即使在 TCP 发送者和接收者完全完成他们的工作并且两个流都结束之后。
	* 选项 B：被动关闭。先决条件 #1 到 #3 为真，并且本地对等点 100% 确定远程对等点可以满足先决条件 #3。 如果 TCP 不确认确认，这怎么可能？ 因为远程对等方是第一个结束其流的。



> 为什么这条规则有效？这是脑筋急转弯，您无需进一步阅读即可完成本实验，但思考并了解两位将军问题的深层原因以及不可靠网络对可靠性的固有限制是很有趣的。这样做的原因是，在接收并组装远程对等方的 `FIN` （先决条件＃1）之后，本地对等方发送了一个比它以前发送的序列号更大的段（至少，它必须发送自己的 `FIN` 段以满足先决条件#2)，并且该段也有一个确认远程对等端的 `FIN` 位的 ackno。远程对等点确认了该段（以满足先决条件＃3），这意味着远程对等点也必须看到本地对等点对远程对等点 `FIN` 的确认。这保证了远程对等点必须能够满足其自身的先决条件#3。所有这一切意味着本地对等点可以满足先决条件 #4 而不必逗留。哇！我们说这是一个脑筋急转弯。在你的实验室文章中加分：你能找到更好的方法来解释这一点吗？



底线是，如果 TCP Connections 入站流在 TCP Connection 发送 `FIN` 段之前结束，则 TCP Connection 不需要在两个流完成后逗留。



### TCP connection (实践总结)

实际上，这一切意味着您的 TCPConnection 在流完成后有一个名为 `_linger_after_streams_finish` 的成员变量，通过`state()` 方法暴露给测试设备。 变量开始为真。 如果入站流在 `TCPConnection` 在其出站流上达到 `EOF` 之前结束，则需要将此变量设置为 false。 在满足先决条件 #1 到 #3 的任何时候，如果流完成后的 linger 为假，则连接“完成”（并且 active() 应该返回假）。 否则你需要拖延：连接只有在收到最后一个段后经过足够的时间（`10 × _cfg.rt` 超时）后才完成。



## 代码

参考了 https://www.epis2048.net/2022/cs144-lab4/

主要围绕状态图去写代码：https://cxybb.com/article/kimylrong/50933169

![TCP连接状态图](https://img-blog.csdn.net/20160319211210424)



根据状态图来对 TCP 连接进行管理

* LISTEN - 侦听来自远方TCP端口的连接请求； 
* SYN-SENT -在发送连接请求后等待匹配的连接请求； 
* SYN-RECEIVED - 在收到和发送一个连接请求后等待对连接请求的确认； 
* ESTABLISHED- 代表一个打开的连接，数据可以传送给用户； 
* FIN-WAIT-1 - 等待远程TCP的连接中断请求，或先前的连接中断请求的确认；
* FIN-WAIT-2 - 从远程TCP等待连接中断请求； 
* CLOSE-WAIT - 等待从本地用户发来的连接中断请求； 
* CLOSING -等待远程TCP对连接中断的确认； 
* LAST-ACK - 等待原来发向远程TCP的连接中断请求的确认； 
* TIME-WAIT -等待足够的时间以确保远程TCP接收到连接中断请求的确认； 
* CLOSED - 没有任何连接状态；

一个 TCP 又可以拆分为 Receiver 和 Sender，TCP 连接的状态也可以由这两个的状态决定，通过 TCP STATE 我们可以知道各个状态的判断条件 

**Server端典型状态图**

![server端状态图](https://img-blog.csdn.net/20160319211433704)

**Client端典型状态图**

![client端状态图](https://img-blog.csdn.net/20160319211514254)

`tcp_state.cc`

```C++
TCPState::TCPState(const TCPState::State state) {
    switch (state) {
        case TCPState::State::LISTEN:
            _receiver = TCPReceiverStateSummary::LISTEN;
            _sender = TCPSenderStateSummary::CLOSED;
            break;
        case TCPState::State::SYN_RCVD:
            _receiver = TCPReceiverStateSummary::SYN_RECV;
            _sender = TCPSenderStateSummary::SYN_SENT;
            break;
        case TCPState::State::SYN_SENT:
            _receiver = TCPReceiverStateSummary::LISTEN;
            _sender = TCPSenderStateSummary::SYN_SENT;
            break;
        case TCPState::State::ESTABLISHED:
            _receiver = TCPReceiverStateSummary::SYN_RECV;
            _sender = TCPSenderStateSummary::SYN_ACKED;
            break;
        case TCPState::State::CLOSE_WAIT:
            _receiver = TCPReceiverStateSummary::FIN_RECV;
            _sender = TCPSenderStateSummary::SYN_ACKED;
            _linger_after_streams_finish = false;
            break;
        case TCPState::State::LAST_ACK:
            _receiver = TCPReceiverStateSummary::FIN_RECV;
            _sender = TCPSenderStateSummary::FIN_SENT;
            _linger_after_streams_finish = false;
            break;
        case TCPState::State::CLOSING:
            _receiver = TCPReceiverStateSummary::FIN_RECV;
            _sender = TCPSenderStateSummary::FIN_SENT;
            break;
        case TCPState::State::FIN_WAIT_1:
            _receiver = TCPReceiverStateSummary::SYN_RECV;
            _sender = TCPSenderStateSummary::FIN_SENT;
            break;
        case TCPState::State::FIN_WAIT_2:
            _receiver = TCPReceiverStateSummary::SYN_RECV;
            _sender = TCPSenderStateSummary::FIN_ACKED;
            break;
        case TCPState::State::TIME_WAIT:
            _receiver = TCPReceiverStateSummary::FIN_RECV;
            _sender = TCPSenderStateSummary::FIN_ACKED;
            break;
        case TCPState::State::RESET:
            _receiver = TCPReceiverStateSummary::ERROR;
            _sender = TCPSenderStateSummary::ERROR;
            _linger_after_streams_finish = false;
            _active = false;
            break;
        case TCPState::State::CLOSED:
            _receiver = TCPReceiverStateSummary::FIN_RECV;
            _sender = TCPSenderStateSummary::FIN_ACKED;
            _linger_after_streams_finish = false;
            _active = false;
            break;
    }
}

TCPState::TCPState(const TCPSender &sender, const TCPReceiver &receiver, const bool active, const bool linger)
    : _sender(state_summary(sender))
    , _receiver(state_summary(receiver))
    , _active(active)
    , _linger_after_streams_finish(active ? linger : false) {}

string TCPState::state_summary(const TCPReceiver &receiver) {
    if (receiver.stream_out().error()) {
        return TCPReceiverStateSummary::ERROR;
    } else if (not receiver.ackno().has_value()) {
        return TCPReceiverStateSummary::LISTEN;
    } else if (receiver.stream_out().input_ended()) {
        return TCPReceiverStateSummary::FIN_RECV;
    } else {
        return TCPReceiverStateSummary::SYN_RECV;
    }
}

string TCPState::state_summary(const TCPSender &sender) {
    if (sender.stream_in().error()) {
        return TCPSenderStateSummary::ERROR;
    } else if (sender.next_seqno_absolute() == 0) {
        return TCPSenderStateSummary::CLOSED;
    } else if (sender.next_seqno_absolute() == sender.bytes_in_flight()) {
        return TCPSenderStateSummary::SYN_SENT;
    } else if (not sender.stream_in().eof()) {
        return TCPSenderStateSummary::SYN_ACKED;
    } else if (sender.next_seqno_absolute() < sender.stream_in().bytes_written() + 2) {
        return TCPSenderStateSummary::SYN_ACKED;
    } else if (sender.bytes_in_flight()) {
        return TCPSenderStateSummary::FIN_SENT;
    } else {
        return TCPSenderStateSummary::FIN_ACKED;
    }
}

```



![img](https://upload-images.jianshu.io/upload_images/5959612-892c144f8a31df02.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/510/format/webp)

### TCP 3次握手建立连接

1. 客户端发送一个带SYN标志的TCP报文到服务器。。
2. 服务器端回应客户端，这是三次握手中的第2个报文，这个报文同时带ACK标志和SYN标志。因此它表示对刚才客户端SYN报文的回应；同时又标志SYN给客户端，询问客户端是否准备好进行数据通讯。 
3. 客户必须再次回应服务段一个ACK报文。

### TCP 4次挥手结束连接

1. TCP客户端发送一个FIN，用来关闭客户到服务器的数据传送。 
2. 服务器收到这个FIN，它发回一个ACK，确认序号为收到的序号加1。和SYN一样，一个FIN将占用一个序号。 
3. 服务器关闭客户端的连接，发送一个FIN给客户端。 
4. 客户段发回ACK报文确认，并将确认序号设置为收到序号加1。 



### RST 强制结束

如果接收到带 RST 的包就要立即强制结束，并将输入输出流设为错误状态





`tcp_connection.hh`

```c++
#ifndef SPONGE_LIBSPONGE_TCP_FACTORED_HH
#define SPONGE_LIBSPONGE_TCP_FACTORED_HH

#include "tcp_config.hh"
#include "tcp_receiver.hh"
#include "tcp_sender.hh"
#include "tcp_state.hh"

//! \brief A complete endpoint of a TCP connection
class TCPConnection {
  private:
    TCPConfig _cfg;
    TCPReceiver _receiver{_cfg.recv_capacity};
    TCPSender _sender{_cfg.send_capacity, _cfg.rt_timeout, _cfg.fixed_isn};

    //! outbound queue of segments that the TCPConnection wants sent
    std::queue<TCPSegment> _segments_out{};

    //! Should the TCPConnection stay active (and keep ACKing)
    //! for 10 * _cfg.rt_timeout milliseconds after both streams have ended,
    //! in case the remote TCPConnection doesn't know we've received its whole stream?
    bool _linger_after_streams_finish{true};

    bool _is_active{true};

    size_t _time_since_last_segment_received{0};

  public:
    void send_data();

    void force_shutdown();

    //! \name "Input" interface for the writer
    //!@{

    //! \brief Initiate a connection by sending a SYN segment
    void connect();

    //! \brief Write data to the outbound byte stream, and send it over TCP if possible
    //! \returns the number of bytes from `data` that were actually written.
    size_t write(const std::string &data);

    //! \returns the number of `bytes` that can be written right now.
    size_t remaining_outbound_capacity() const;

    //! \brief Shut down the outbound byte stream (still allows reading incoming data)
    void end_input_stream();
    //!@}

    //! \name "Output" interface for the reader
    //!@{

    //! \brief The inbound byte stream received from the peer
    ByteStream &inbound_stream() { return _receiver.stream_out(); }
    //!@}

    //! \name Accessors used for testing

    //!@{
    //! \brief number of bytes sent and not yet acknowledged, counting SYN/FIN each as one byte
    size_t bytes_in_flight() const;
    //! \brief number of bytes not yet reassembled
    size_t unassembled_bytes() const;
    //! \brief Number of milliseconds since the last segment was received
    size_t time_since_last_segment_received() const;
    //!< \brief summarize the state of the sender, receiver, and the connection
    TCPState state() const { return {_sender, _receiver, active(), _linger_after_streams_finish}; };
    //!@}

    //! \name Methods for the owner or operating system to call
    //!@{

    //! Called when a new segment has been received from the network
    void segment_received(const TCPSegment &seg);

    //! Called periodically when time elapses
    void tick(const size_t ms_since_last_tick);

    //! \brief TCPSegments that the TCPConnection has enqueued for transmission.
    //! \note The owner or operating system will dequeue these and
    //! put each one into the payload of a lower-layer datagram (usually Internet datagrams (IP),
    //! but could also be user datagrams (UDP) or any other kind).
    std::queue<TCPSegment> &segments_out() { return _segments_out; }

    //! \brief Is the connection still alive in any way?
    //! \returns `true` if either stream is still running or if the TCPConnection is lingering
    //! after both streams have finished (e.g. to ACK retransmissions from the peer)
    bool active() const;
    //!@}

    //! Construct a new connection from a configuration
    explicit TCPConnection(const TCPConfig &cfg) : _cfg{cfg} {}

    //! \name construction and destruction
    //! moving is allowed; copying is disallowed; default construction not possible

    //!@{
    ~TCPConnection();  //!< destructor sends a RST if the connection is still open
    TCPConnection() = delete;
    TCPConnection(TCPConnection &&other) = default;
    TCPConnection &operator=(TCPConnection &&other) = default;
    TCPConnection(const TCPConnection &other) = delete;
    TCPConnection &operator=(const TCPConnection &other) = delete;
    //!@}
};

#endif  // SPONGE_LIBSPONGE_TCP_FACTORED_HH

```

`tcp_connection.cc`

```c++
#include "tcp_connection.hh"

#include <iostream>

// Dummy implementation of a TCP connection

// For Lab 4, please replace with a real implementation that passes the
// automated checks run by `make check`.

template <typename... Targs>
void DUMMY_CODE(Targs &&.../* unused */) {}

using namespace std;

size_t TCPConnection::remaining_outbound_capacity() const { return _sender.stream_in().remaining_capacity(); }

size_t TCPConnection::bytes_in_flight() const { return _sender.bytes_in_flight(); }

size_t TCPConnection::unassembled_bytes() const { return _receiver.unassembled_bytes(); }

size_t TCPConnection::time_since_last_segment_received() const { return _time_since_last_segment_received; }

void TCPConnection::segment_received(const TCPSegment &seg) {
    if (!_is_active)
        return;

    _time_since_last_segment_received = 0;

    if (seg.header().rst) {
        _receiver.stream_out().set_error();
        _sender.stream_in().set_error();
        _is_active = false;
        return;
    }

    if (TCPState::state_summary(_sender) == TCPSenderStateSummary::CLOSED) {  // CLOSED, LISTEN
        // 收到SYN，说明TCP连接由对方启动，进入Syn-Revd状态
        if (seg.header().syn) {
            // 此时还没有ACK，所以sender不需要ack_received
            _receiver.segment_received(seg);
            // 我们主动发送一个SYN
            connect();
        }

    } else if (TCPState::state_summary(_sender) == TCPSenderStateSummary::SYN_SENT &&
               TCPState::state_summary(_receiver) == TCPReceiverStateSummary::LISTEN) {  // SYN_SENT
        if (seg.header().syn && seg.header().ack) {
            // 收到SYN和ACK，说明由对方主动开启连接，进入Established状态，通过一个空包来发送ACK
            _receiver.segment_received(seg);
            _sender.ack_received(seg.header().ackno, seg.header().win);
            // 发送 ACK
            _sender.send_empty_segment();
            send_data();
        } else if (seg.header().syn && !seg.header().ack) {
            // 只收到了 SYN，说明由双方同时开启连接，进入SYN-RCVD 状态，没有接收到对方的 ACK，我们主动发一个
            _receiver.segment_received(seg);
            _sender.send_empty_segment();
            send_data();
        }

    } else if (TCPState::state_summary(_sender) == TCPSenderStateSummary::SYN_SENT &&
               TCPState::state_summary(_receiver) == TCPReceiverStateSummary::LISTEN) {  // SYN_RCVD
        // 接收ACK，进入Established状态
        _receiver.segment_received(seg);
        _sender.ack_received(seg.header().ackno, seg.header().win);

    } else if (TCPState::state_summary(_sender) == TCPSenderStateSummary::SYN_ACKED &&
               TCPState::state_summary(_receiver) == TCPReceiverStateSummary::SYN_RECV) {  // ESTABLISHED
        // 发送数据，如果接到数据，则更新ACK
        _receiver.segment_received(seg);
        _sender.ack_received(seg.header().ackno, seg.header().win);
        if (seg.length_in_sequence_space() > 0) {
            _sender.send_empty_segment();
        }
        _sender.fill_window();
        send_data();

    } else if (TCPState::state_summary(_sender) == TCPSenderStateSummary::FIN_SENT &&
               TCPState::state_summary(_receiver) == TCPReceiverStateSummary::SYN_RECV) {  // FIN_WAIT_1
        if (seg.header().fin) {
            // 收到 FIN
            _sender.ack_received(seg.header().ackno, seg.header().win);
            _receiver.segment_received(seg);
            // 则发送新 ACK，进入 Closing/Time-Wait
            _sender.send_empty_segment();
            send_data();
        } else if (seg.header().ack) {
            // 收到ACK，进入Fin-Wait-2
            _sender.ack_received(seg.header().ackno, seg.header().win);
            _receiver.segment_received(seg);
            send_data();
        }

    } else if (TCPState::state_summary(_sender) == TCPSenderStateSummary::FIN_ACKED &&
               TCPState::state_summary(_receiver) == TCPReceiverStateSummary::SYN_RECV) {  // FIN_WAIT_2
        _sender.ack_received(seg.header().ackno, seg.header().win);
        _receiver.segment_received(seg);
        _sender.send_empty_segment();
        send_data();

    } else if (TCPState::state_summary(_sender) == TCPSenderStateSummary::FIN_ACKED &&
               TCPState::state_summary(_receiver) == TCPReceiverStateSummary::FIN_RECV) {  // TIME_WAIT
        if (seg.header().fin) {
            // 收到FIN，保持Time-Wait状态
            _sender.ack_received(seg.header().ackno, seg.header().win);
            _receiver.segment_received(seg);
            _sender.send_empty_segment();
            send_data();
        }

    } else if (TCPState::state_summary(_receiver) == TCPReceiverStateSummary::FIN_RECV &&
               TCPState::state_summary(_sender) == TCPSenderStateSummary::SYN_ACKED &&
               _linger_after_streams_finish == false) {  // CLOSE_WAIT
        _receiver.segment_received(seg);
        _sender.ack_received(seg.header().ackno, seg.header().win);
        if (seg.length_in_sequence_space() > 0) {
            _sender.send_empty_segment();
        }
        _sender.fill_window();
        send_data();
    } else {
        _sender.ack_received(seg.header().ackno, seg.header().win);
        _receiver.segment_received(seg);

        _sender.fill_window();
        send_data();
    }
}

bool TCPConnection::active() const { return _is_active; }

size_t TCPConnection::write(const string &data) {
    if (data.empty())
        return 0;
    size_t written_bytes = _sender.stream_in().write(data);
    _sender.fill_window();
    send_data();
    return written_bytes;
}

//! \param[in] ms_since_last_tick number of milliseconds since the last call to this method
void TCPConnection::tick(const size_t ms_since_last_tick) {
    if (!active()) {
        return;
    }

    _time_since_last_segment_received += ms_since_last_tick;
    _sender.tick(ms_since_last_tick);

    if (_sender.consecutive_retransmissions() > TCPConfig::MAX_RETX_ATTEMPTS) {
        force_shutdown();
        return;
    }

    send_data();
}

void TCPConnection::end_input_stream() {
    _sender.stream_in().end_input();
    _sender.fill_window();
    send_data();
}

void TCPConnection::connect() {
    _sender.fill_window();
    send_data();
}
TCPConnection::~TCPConnection() {
    try {
        if (active()) {
            cerr << "Warning: Unclean shutdown of TCPConnection\n";
            force_shutdown();
            // Your code here: need to send a RST segment to the peer
        }
    } catch (const exception &e) {
        std::cerr << "Exception destructing TCP FSM: " << e.what() << std::endl;
    }
}

void TCPConnection::force_shutdown() {
    // sets both the inbound and outbound streams to the error state
    _receiver.stream_out().set_error();
    _sender.stream_in().set_error();
    // and kills the connection permanently
    _is_active = false;
    _linger_after_streams_finish = false;

    TCPSegment seg;
    seg.header().rst = true;
    _segments_out.emplace(seg);
}

void TCPConnection::send_data() {
    // 将sender中的数据保存到 TCP connection 中
    while (!_sender.segments_out().empty()) {
        TCPSegment seg = _sender.segments_out().front();
        _sender.segments_out().pop();

        if (_receiver.ackno().has_value()) {
            seg.header().ack = true;
            seg.header().ackno = _receiver.ackno().value();
            seg.header().win = _receiver.window_size();
        }
        _segments_out.emplace(seg);
    }

    // 如果发送完毕就结束连接
    if (_receiver.stream_out().input_ended()) {
        if (!_sender.stream_in().eof()) {
            _linger_after_streams_finish = false;
        } else if (_sender.bytes_in_flight() == 0) {
            if (!_linger_after_streams_finish || _time_since_last_segment_received >= 10 * _cfg.rt_timeout) {
                _is_active = false;
            }
        }
    }
}

```



## 测试

```shell
Test project /home/zsl/CLionProjects/sponge/build
        Start   1: t_wrapping_ints_cmp
  1/162 Test   #1: t_wrapping_ints_cmp ..............   Passed    0.01 sec
        Start   2: t_wrapping_ints_unwrap
  2/162 Test   #2: t_wrapping_ints_unwrap ...........   Passed    0.00 sec
        Start   3: t_wrapping_ints_wrap
  3/162 Test   #3: t_wrapping_ints_wrap .............   Passed    0.00 sec
        Start   4: t_wrapping_ints_roundtrip
  4/162 Test   #4: t_wrapping_ints_roundtrip ........   Passed    0.12 sec
        Start   5: t_recv_connect
  5/162 Test   #5: t_recv_connect ...................   Passed    0.00 sec
        Start   6: t_recv_transmit
  6/162 Test   #6: t_recv_transmit ..................   Passed    0.03 sec
        Start   7: t_recv_window
  7/162 Test   #7: t_recv_window ....................   Passed    0.00 sec
        Start   8: t_recv_reorder
  8/162 Test   #8: t_recv_reorder ...................   Passed    0.00 sec
        Start   9: t_recv_close
  9/162 Test   #9: t_recv_close .....................   Passed    0.00 sec
        Start  10: t_recv_special
 10/162 Test  #10: t_recv_special ...................   Passed    0.00 sec
        Start  11: t_send_connect
 11/162 Test  #11: t_send_connect ...................   Passed    0.00 sec
        Start  12: t_send_transmit
 12/162 Test  #12: t_send_transmit ..................   Passed    0.03 sec
        Start  13: t_send_retx
 13/162 Test  #13: t_send_retx ......................   Passed    0.00 sec
        Start  14: t_send_window
 14/162 Test  #14: t_send_window ....................   Passed    0.01 sec
        Start  15: t_send_ack
 15/162 Test  #15: t_send_ack .......................   Passed    0.00 sec
        Start  16: t_send_close
 16/162 Test  #16: t_send_close .....................   Passed    0.00 sec
        Start  17: t_send_extra
 17/162 Test  #17: t_send_extra .....................   Passed    0.00 sec
        Start  18: t_strm_reassem_single
 18/162 Test  #18: t_strm_reassem_single ............   Passed    0.00 sec
        Start  19: t_strm_reassem_seq
 19/162 Test  #19: t_strm_reassem_seq ...............   Passed    0.00 sec
        Start  20: t_strm_reassem_dup
 20/162 Test  #20: t_strm_reassem_dup ...............   Passed    0.01 sec
        Start  21: t_strm_reassem_holes
 21/162 Test  #21: t_strm_reassem_holes .............   Passed    0.00 sec
        Start  22: t_strm_reassem_many
 22/162 Test  #22: t_strm_reassem_many ..............   Passed    0.58 sec
        Start  23: t_strm_reassem_overlapping
 23/162 Test  #23: t_strm_reassem_overlapping .......   Passed    0.00 sec
        Start  24: t_strm_reassem_win
 24/162 Test  #24: t_strm_reassem_win ...............   Passed    0.69 sec
        Start  25: t_strm_reassem_cap
 25/162 Test  #25: t_strm_reassem_cap ...............   Passed    0.05 sec
        Start  26: t_byte_stream_construction
 26/162 Test  #26: t_byte_stream_construction .......   Passed    0.00 sec
        Start  27: t_byte_stream_one_write
 27/162 Test  #27: t_byte_stream_one_write ..........   Passed    0.00 sec
        Start  28: t_byte_stream_two_writes
 28/162 Test  #28: t_byte_stream_two_writes .........   Passed    0.00 sec
        Start  29: t_byte_stream_capacity
 29/162 Test  #29: t_byte_stream_capacity ...........   Passed    0.21 sec
        Start  30: t_byte_stream_many_writes
 30/162 Test  #30: t_byte_stream_many_writes ........   Passed    0.01 sec
        Start  31: t_webget
 31/162 Test  #31: t_webget .........................   Passed    2.53 sec
        Start  34: t_tcp_parser
 32/162 Test  #34: t_tcp_parser .....................   Passed    0.43 sec
        Start  35: t_ipv4_parser
 33/162 Test  #35: t_ipv4_parser ....................   Passed    0.11 sec
        Start  36: t_active_close
 34/162 Test  #36: t_active_close ...................   Passed    0.11 sec
        Start  37: t_passive_close
 35/162 Test  #37: t_passive_close ..................   Passed    0.25 sec
        Start  39: t_ack_rst
 36/162 Test  #39: t_ack_rst ........................   Passed    0.00 sec
        Start  41: t_ack_rst_win
 37/162 Test  #41: t_ack_rst_win ....................   Passed    0.00 sec
        Start  43: t_connect
 38/162 Test  #43: t_connect ........................   Passed    0.00 sec
        Start  45: t_listen
 39/162 Test  #45: t_listen .........................   Passed    0.00 sec
        Start  46: t_winsize
 40/162 Test  #46: t_winsize ........................   Passed    0.05 sec
        Start  48: t_retx
 41/162 Test  #48: t_retx ...........................   Passed    0.00 sec
        Start  49: t_retx_win
 42/162 Test  #49: t_retx_win .......................   Passed    0.00 sec
        Start  50: t_loopback
 43/162 Test  #50: t_loopback .......................   Passed    0.23 sec
        Start  51: t_loopback_win
 44/162 Test  #51: t_loopback_win ...................   Passed    0.34 sec
        Start  52: t_reorder
 45/162 Test  #52: t_reorder ........................   Passed    0.72 sec
        Start  53: t_address_dt
 46/162 Test  #53: t_address_dt .....................   Passed    0.03 sec
        Start  54: t_parser_dt
 47/162 Test  #54: t_parser_dt ......................   Passed    0.00 sec
        Start  55: t_socket_dt
 48/162 Test  #55: t_socket_dt ......................   Passed    0.01 sec
        Start  56: t_udp_client_send
 49/162 Test  #56: t_udp_client_send ................   Passed    2.08 sec
        Start  57: t_udp_server_send
 50/162 Test  #57: t_udp_server_send ................   Passed    0.26 sec
        Start  58: t_udp_client_recv
 51/162 Test  #58: t_udp_client_recv ................   Passed    0.24 sec
        Start  59: t_udp_server_recv
 52/162 Test  #59: t_udp_server_recv ................   Passed    0.24 sec
        Start  60: t_udp_client_dupl
 53/162 Test  #60: t_udp_client_dupl ................   Passed    0.24 sec
        Start  61: t_udp_server_dupl
 54/162 Test  #61: t_udp_server_dupl ................   Passed    0.24 sec
        Start  62: t_ucS_1M_32k
 55/162 Test  #62: t_ucS_1M_32k .....................   Passed    0.27 sec
        Start  63: t_ucS_128K_8K
 56/162 Test  #63: t_ucS_128K_8K ....................   Passed    0.26 sec
        Start  64: t_ucS_16_1
 57/162 Test  #64: t_ucS_16_1 .......................   Passed    0.25 sec
        Start  65: t_ucS_32K_d
 58/162 Test  #65: t_ucS_32K_d ......................   Passed    0.24 sec
        Start  66: t_ucR_1M_32k
 59/162 Test  #66: t_ucR_1M_32k .....................   Passed    0.29 sec
        Start  67: t_ucR_128K_8K
 60/162 Test  #67: t_ucR_128K_8K ....................   Passed    0.25 sec
        Start  68: t_ucR_16_1
 61/162 Test  #68: t_ucR_16_1 .......................   Passed    0.24 sec
        Start  69: t_ucR_32K_d
 62/162 Test  #69: t_ucR_32K_d ......................   Passed    0.24 sec
        Start  70: t_ucD_1M_32k
 63/162 Test  #70: t_ucD_1M_32k .....................   Passed    0.32 sec
        Start  71: t_ucD_128K_8K
 64/162 Test  #71: t_ucD_128K_8K ....................   Passed    0.26 sec
        Start  72: t_ucD_16_1
 65/162 Test  #72: t_ucD_16_1 .......................   Passed    0.26 sec
        Start  73: t_ucD_32K_d
 66/162 Test  #73: t_ucD_32K_d ......................   Passed    0.25 sec
        Start  74: t_usS_1M_32k
 67/162 Test  #74: t_usS_1M_32k .....................   Passed    0.29 sec
        Start  75: t_usS_128K_8K
 68/162 Test  #75: t_usS_128K_8K ....................   Passed    0.26 sec
        Start  76: t_usS_16_1
 69/162 Test  #76: t_usS_16_1 .......................   Passed    0.25 sec
        Start  77: t_usS_32K_d
 70/162 Test  #77: t_usS_32K_d ......................   Passed    0.25 sec
        Start  78: t_usR_1M_32k
 71/162 Test  #78: t_usR_1M_32k .....................   Passed    0.30 sec
        Start  79: t_usR_128K_8K
 72/162 Test  #79: t_usR_128K_8K ....................   Passed    0.25 sec
        Start  80: t_usR_16_1
 73/162 Test  #80: t_usR_16_1 .......................   Passed    0.24 sec
        Start  81: t_usR_32K_d
 74/162 Test  #81: t_usR_32K_d ......................   Passed    0.25 sec
        Start  82: t_usD_1M_32k
 75/162 Test  #82: t_usD_1M_32k .....................   Passed    0.32 sec
        Start  83: t_usD_128K_8K
 76/162 Test  #83: t_usD_128K_8K ....................   Passed    0.26 sec
        Start  84: t_usD_16_1
 77/162 Test  #84: t_usD_16_1 .......................   Passed    0.26 sec
        Start  85: t_usD_32K_d
 78/162 Test  #85: t_usD_32K_d ......................   Passed    0.25 sec
        Start  86: t_ucS_128K_8K_l
 79/162 Test  #86: t_ucS_128K_8K_l ..................   Passed    0.25 sec
        Start  87: t_ucS_128K_8K_L
 80/162 Test  #87: t_ucS_128K_8K_L ..................   Passed    0.49 sec
        Start  88: t_ucS_128K_8K_lL
 81/162 Test  #88: t_ucS_128K_8K_lL .................   Passed    0.47 sec
        Start  89: t_ucR_128K_8K_l
 82/162 Test  #89: t_ucR_128K_8K_l ..................   Passed    0.53 sec
        Start  90: t_ucR_128K_8K_L
 83/162 Test  #90: t_ucR_128K_8K_L ..................   Passed    0.27 sec
        Start  91: t_ucR_128K_8K_lL
 84/162 Test  #91: t_ucR_128K_8K_lL .................   Passed    0.50 sec
        Start  92: t_ucD_128K_8K_l
 85/162 Test  #92: t_ucD_128K_8K_l ..................   Passed    0.44 sec
        Start  93: t_ucD_128K_8K_L
 86/162 Test  #93: t_ucD_128K_8K_L ..................   Passed    0.58 sec
        Start  94: t_ucD_128K_8K_lL
 87/162 Test  #94: t_ucD_128K_8K_lL .................   Passed    0.60 sec
        Start  95: t_usS_128K_8K_l
 88/162 Test  #95: t_usS_128K_8K_l ..................   Passed    0.25 sec
        Start  96: t_usS_128K_8K_L
 89/162 Test  #96: t_usS_128K_8K_L ..................   Passed    0.43 sec
        Start  97: t_usS_128K_8K_lL
 90/162 Test  #97: t_usS_128K_8K_lL .................   Passed    0.45 sec
        Start  98: t_usR_128K_8K_l
 91/162 Test  #98: t_usR_128K_8K_l ..................   Passed    0.49 sec
        Start  99: t_usR_128K_8K_L
 92/162 Test  #99: t_usR_128K_8K_L ..................   Passed    0.25 sec
        Start 100: t_usR_128K_8K_lL
 93/162 Test #100: t_usR_128K_8K_lL .................   Passed    0.46 sec
        Start 101: t_usD_128K_8K_l
 94/162 Test #101: t_usD_128K_8K_l ..................   Passed    0.62 sec
        Start 102: t_usD_128K_8K_L
 95/162 Test #102: t_usD_128K_8K_L ..................   Passed    0.46 sec
        Start 103: t_usD_128K_8K_lL
 96/162 Test #103: t_usD_128K_8K_lL .................   Passed    0.62 sec
        Start 104: t_ipv4_client_send
 97/162 Test #104: t_ipv4_client_send ...............   Passed    0.60 sec
        Start 105: t_ipv4_server_send
 98/162 Test #105: t_ipv4_server_send ...............   Passed    0.26 sec
        Start 106: t_ipv4_client_recv
 99/162 Test #106: t_ipv4_client_recv ...............   Passed    0.24 sec
        Start 107: t_ipv4_server_recv
100/162 Test #107: t_ipv4_server_recv ...............   Passed    0.24 sec
        Start 108: t_ipv4_client_dupl
101/162 Test #108: t_ipv4_client_dupl ...............   Passed    0.24 sec
        Start 109: t_ipv4_server_dupl
102/162 Test #109: t_ipv4_server_dupl ...............   Passed    0.25 sec
        Start 110: t_icS_1M_32k
103/162 Test #110: t_icS_1M_32k .....................   Passed    0.32 sec
        Start 111: t_icS_128K_8K
104/162 Test #111: t_icS_128K_8K ....................   Passed    0.26 sec
        Start 112: t_icS_16_1
105/162 Test #112: t_icS_16_1 .......................   Passed    0.26 sec
        Start 113: t_icS_32K_d
106/162 Test #113: t_icS_32K_d ......................   Passed    0.25 sec
        Start 114: t_icR_1M_32k
107/162 Test #114: t_icR_1M_32k .....................   Passed    0.34 sec
        Start 115: t_icR_128K_8K
108/162 Test #115: t_icR_128K_8K ....................   Passed    0.26 sec
        Start 116: t_icR_16_1
109/162 Test #116: t_icR_16_1 .......................   Passed    0.24 sec
        Start 117: t_icR_32K_d
110/162 Test #117: t_icR_32K_d ......................   Passed    0.25 sec
        Start 118: t_icD_1M_32k
111/162 Test #118: t_icD_1M_32k .....................   Passed    0.38 sec
        Start 119: t_icD_128K_8K
112/162 Test #119: t_icD_128K_8K ....................   Passed    0.28 sec
        Start 120: t_icD_16_1
113/162 Test #120: t_icD_16_1 .......................   Passed    0.27 sec
        Start 121: t_icD_32K_d
114/162 Test #121: t_icD_32K_d ......................   Passed    0.26 sec
        Start 122: t_isS_1M_32k
115/162 Test #122: t_isS_1M_32k .....................   Passed    0.33 sec
        Start 123: t_isS_128K_8K
116/162 Test #123: t_isS_128K_8K ....................   Passed    0.26 sec
        Start 124: t_isS_16_1
117/162 Test #124: t_isS_16_1 .......................   Passed    0.24 sec
        Start 125: t_isS_32K_d
118/162 Test #125: t_isS_32K_d ......................   Passed    0.25 sec
        Start 126: t_isR_1M_32k
119/162 Test #126: t_isR_1M_32k .....................   Passed    0.33 sec
        Start 127: t_isR_128K_8K
120/162 Test #127: t_isR_128K_8K ....................   Passed    0.26 sec
        Start 128: t_isR_16_1
121/162 Test #128: t_isR_16_1 .......................   Passed    0.25 sec
        Start 129: t_isR_32K_d
122/162 Test #129: t_isR_32K_d ......................   Passed    0.25 sec
        Start 130: t_isD_1M_32k
123/162 Test #130: t_isD_1M_32k .....................   Passed    0.39 sec
        Start 131: t_isD_128K_8K
124/162 Test #131: t_isD_128K_8K ....................   Passed    0.29 sec
        Start 132: t_isD_16_1
125/162 Test #132: t_isD_16_1 .......................   Passed    0.26 sec
        Start 133: t_isD_32K_d
126/162 Test #133: t_isD_32K_d ......................   Passed    0.25 sec
        Start 134: t_icS_128K_8K_l
127/162 Test #134: t_icS_128K_8K_l ..................   Passed    0.26 sec
        Start 135: t_icS_128K_8K_L
128/162 Test #135: t_icS_128K_8K_L ..................   Passed    0.37 sec
        Start 136: t_icS_128K_8K_lL
129/162 Test #136: t_icS_128K_8K_lL .................   Passed    0.54 sec
        Start 137: t_icR_128K_8K_l
130/162 Test #137: t_icR_128K_8K_l ..................   Passed    0.45 sec
        Start 138: t_icR_128K_8K_L
131/162 Test #138: t_icR_128K_8K_L ..................   Passed    0.26 sec
        Start 139: t_icR_128K_8K_lL
132/162 Test #139: t_icR_128K_8K_lL .................   Passed    0.55 sec
        Start 140: t_icD_128K_8K_l
133/162 Test #140: t_icD_128K_8K_l ..................   Passed    0.54 sec
        Start 141: t_icD_128K_8K_L
134/162 Test #141: t_icD_128K_8K_L ..................   Passed    0.42 sec
        Start 142: t_icD_128K_8K_lL
135/162 Test #142: t_icD_128K_8K_lL .................   Passed    0.64 sec
        Start 143: t_isS_128K_8K_l
136/162 Test #143: t_isS_128K_8K_l ..................   Passed    0.27 sec
        Start 144: t_isS_128K_8K_L
137/162 Test #144: t_isS_128K_8K_L ..................   Passed    0.47 sec
        Start 145: t_isS_128K_8K_lL
138/162 Test #145: t_isS_128K_8K_lL .................   Passed    0.70 sec
        Start 146: t_isR_128K_8K_l
139/162 Test #146: t_isR_128K_8K_l ..................   Passed    0.53 sec
        Start 147: t_isR_128K_8K_L
140/162 Test #147: t_isR_128K_8K_L ..................   Passed    6.27 sec
        Start 148: t_isR_128K_8K_lL
141/162 Test #148: t_isR_128K_8K_lL .................   Passed    0.60 sec
        Start 149: t_isD_128K_8K_l
142/162 Test #149: t_isD_128K_8K_l ..................   Passed    0.59 sec
        Start 150: t_isD_128K_8K_L
143/162 Test #150: t_isD_128K_8K_L ..................   Passed    0.50 sec
        Start 151: t_isD_128K_8K_lL
144/162 Test #151: t_isD_128K_8K_lL .................   Passed    0.56 sec
        Start 152: t_icnS_128K_8K_l
145/162 Test #152: t_icnS_128K_8K_l .................   Passed    0.13 sec
        Start 153: t_icnS_128K_8K_L
146/162 Test #153: t_icnS_128K_8K_L .................   Passed    0.26 sec
        Start 154: t_icnS_128K_8K_lL
147/162 Test #154: t_icnS_128K_8K_lL ................   Passed    0.30 sec
        Start 155: t_icnR_128K_8K_l
148/162 Test #155: t_icnR_128K_8K_l .................   Passed    0.88 sec
        Start 156: t_icnR_128K_8K_L
149/162 Test #156: t_icnR_128K_8K_L .................   Passed    0.29 sec
        Start 157: t_icnR_128K_8K_lL
150/162 Test #157: t_icnR_128K_8K_lL ................   Passed    0.56 sec
        Start 158: t_icnD_128K_8K_l
151/162 Test #158: t_icnD_128K_8K_l .................   Passed    0.42 sec
        Start 159: t_icnD_128K_8K_L
152/162 Test #159: t_icnD_128K_8K_L .................   Passed    0.31 sec
        Start 160: t_icnD_128K_8K_lL
153/162 Test #160: t_icnD_128K_8K_lL ................   Passed    0.48 sec
        Start 161: t_isnS_128K_8K_l
154/162 Test #161: t_isnS_128K_8K_l .................   Passed    1.14 sec
        Start 162: t_isnS_128K_8K_L
155/162 Test #162: t_isnS_128K_8K_L .................   Passed    0.36 sec
        Start 163: t_isnS_128K_8K_lL
156/162 Test #163: t_isnS_128K_8K_lL ................   Passed    0.45 sec
        Start 164: t_isnR_128K_8K_l
157/162 Test #164: t_isnR_128K_8K_l .................   Passed    0.46 sec
        Start 165: t_isnR_128K_8K_L
158/162 Test #165: t_isnR_128K_8K_L .................   Passed    0.29 sec
        Start 166: t_isnR_128K_8K_lL
159/162 Test #166: t_isnR_128K_8K_lL ................   Passed    0.59 sec
        Start 167: t_isnD_128K_8K_l
160/162 Test #167: t_isnD_128K_8K_l .................   Passed    0.78 sec
        Start 168: t_isnD_128K_8K_L
161/162 Test #168: t_isnD_128K_8K_L .................   Passed    0.37 sec
        Start 169: t_isnD_128K_8K_lL
162/162 Test #169: t_isnD_128K_8K_lL ................   Passed    1.30 sec

100% tests passed, 0 tests failed out of 162

Total Test time (real) =  57.59 sec
[100%] Built target check_lab4

```



**性能测试**

```shell
CPU-limited throughput                : 0.33 Gbit/s
CPU-limited throughput with reordering: 0.05 Gbit/s
```

满足实验要求不过性能还是比较低的，后面需要做进一步优化
