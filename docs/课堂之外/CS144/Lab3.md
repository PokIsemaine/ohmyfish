# Lab3 实现 TCP 发送方

## 概述

在 Lab 0 中，您实现了流控制字节流 (ByteStream) 的抽象。 在实验 1 和 2 中，您实现了将不可靠数据报中携带的段转换为传入字节流的工具：StreamReassembler 和 TCPReceiver。 

现在，在实验 3 中，您将实现连接的另一端。 TCPSender 是一种工具，可将传出字节流转换为将成为不可靠数据报有效负载的段。 最后，在实验 4 中，您将结合前面的工作和实验来创建一个有效的 TCP 实现：一个包含 TCPSender 和 TCPReceiver 的 TCPConnection。 您将使用它与 Internet 上的真实服务器通信。

![image-20220726103422931](https://s2.loli.net/2022/07/26/e5I4HkRsF7calqN.png)

## 设计

TCP 是一种通过不可靠的数据报可靠地传送一对流控制字节流（每个方向一个）的协议。两方参与 TCP 连接，每一方同时充当（其自己的传出字节流的）“发送者”和（传入字节流的）“接收者”。两方被称为连接的“端点”或“对等点”。

本周，您将实现 TCP 的“发送方”部分，**负责读取 ByteStream（由某些发送方应用程序创建和写入），并将流转换为一系列传出 TCP 段**。在远程端，TCP 接收器 1 将这些段（那些到达的——它们可能不会全部到达）转换回原始字节流，并将确认和窗口广告发送回发送者。 

TCP 发送方和接收方各自负责 TCP 段的一部分。 TCP 发送方写入 TCPSegment 中与实验 2 中的 TCPReceiver 相关的所有字段：即序列号、SYN 标志、有效负载和 FIN 标志。但是，**TCP 发送方只读取接收方写入的段中的字段：确认和窗口大小**。这是 TCP 段的结构，仅突出显示发送方将读取的字段：

![image.png](https://s2.loli.net/2022/09/07/BDPEA1Upct9KMoz.png)

您的 TCPSender 将负责： 

*  跟踪接收者的窗口（处理传入的确认 `ackno` 和窗口大小 `window size`） 
*  尽可能填充窗口，通过从 ByteStream 读取，创建新的 TCP 段（包括 SYN 和 FIN 标志，如果需要）， 并发送它们。 发送方应继续发送段，直到窗口已满或 ByteStream 为空。 
* 跟踪哪些段已发送但接收方尚未确认——我们称这些“未完成”段 (`outstanding segments`)
* 如果自发送后经过足够长的时间且尚未确认，则重新发送未完成的段

> 我为什么要这样做？ 
> 基本原则是发送接收方允许我们发送的任何内容（填充窗口），并继续重传，直到接收方确认每个段。 **这称为“自动重复请求”（ARQ）。** 发送者将字节流分成多个段并发送它们，只要接收者的窗口允许。 感谢您上周的工作，我们知道远程 TCP 接收器可以重建字节流，只要它至少接收每个带索引标记的字节一次 — 无论顺序如何。 发送者的工作是确保接收者至少获得每个字节一次。



### TCPSender 如何知道段丢失了

您的 TCPSender 将发送一堆 TCPSegments。 每个将包含来自传出 ByteStream 的（可能为空的）子字符串，用序列号索引以指示其在流中的位置，并在流的开头用 SYN 标志标记，在结尾用 FIN 标志标记。

除了发送这些段之外，TCPSender 还必须跟踪其未完成的段，直到它们占据的序列号被完全确认。 TCPSender 的所有者会定期调用 TCPSender 的 tick 方法，指示时间的流逝。 TCPSender 负责查看其未完成的 TCPSegments 的集合，并确定最旧的发送段是否在未确认的情况下已未完成的时间过长（即，未确认其所有序列号）。 如果是，则需要重新发送（再次发送）。

以下是“outstanding for too long”的含义的规则。你将要实现这个逻辑，它有点详细，但我们不希望你担心隐藏的测试用例试图绊倒你 或将其视为 SAT 上的应用题。 本周我们会给你一些合理的单元测试，一旦你完成了整个 TCP 实现，我们将在 Lab 4 中进行更全面的集成测试。 只要你 100% 通过这些测试并且你的实现是合理的，你会没事的。



>我为什么要这样做？ 
>总体目标是让发件人及时发现段丢失并需要重新发送。 重新发送之前等待的时间很重要：您不希望发送方等待太久来重新发送一个段（因为这会延迟流向接收应用程序的字节），但您也不希望它重新发送一个 如果发件人稍等片刻，该段将被确认——这浪费了 Internet 的宝贵容量。



1. 每隔几毫秒，您的 TCPSender 的 tick 方法将被调用，并带有一个参数，该参数告诉它自上次调用该方法以来已经过去了多少毫秒。使用它来维护 TCPSender 已存活的总毫秒数的概念。**请不要尝试从操作系统或 CPU 调用任何“时间”或“时钟”函数**——tick 方法是您唯一可以访问时间流逝的方法。这使事情具有确定性和可测试性。
2. 当 TCPSender 被构造时，它被赋予一个参数，告诉它重传超时 (RTO) 的“初始值”。 **RTO 是在重新发送未完成的 TCP 段之前等待的毫秒数。 RTO 的值会随时间变化，但“初始值”保持不变**。起始代码将 RTO 的“初始值”保存在称为初始重传超时的成员变量中。 
3. 您将实现重传计时器：可以在特定时间启动的警报，并且一旦 RTO 过去，警报就会关闭（或“到期”）。**我们强调这个时间流逝的概念来自被调用的 tick 方法,而不是通过获取一天中的实际时间**
4. 每次发送一个包含数据的段（序列空间中的非零长度）时（无论是第一次还是重传），如果定时器没有运行，则启动它，使其在 RTO 毫秒后到期（对于当前值为 RTO）。 通过“过期”，我们的意思是时间将在未来用完一定的毫秒数。
5. 当所有未完成的数据都被确认后，停止重传计时器
6. 如果调用了 tick 并且重传计时器已到期：

	  * 重传尚未被 TCP 接收方完全确认的最早（最低序列号）段。 您需要将未完成的段存储在一些内部数据结构中，以便可以执行此操作。

	* 如果窗口大小不为零： 
		* 跟踪**连续**重传的次数，并增加它，因为你刚刚重传了一些东西。 您的 TCPConnection 将使用此信息来确定连接是否无望（连续重传太多）并需要中止。
		* RTO 的数值翻倍。 这被称为“指数退避”——它会减慢糟糕网络上的重传速度，以避免进一步混淆工作。 

	* 重置重传计时器并启动它，使其在 RTO 毫秒后到期（考虑到您可能刚刚将 RTO 的值翻了一番！）。

7. 当接收方给发送方一个确认成功接收新数据的`ackno`（`ackno`反映的`absolute sequence number`大于任何先前的`ackno`）：

	  * 将 RTO 设置回其“初始值”。

	  * 如果发送方有任何未完成的数据，重新启动重传计时器，使其在 RTO 毫秒后到期（对于 RTO 的当前值）。 

	  * 将“连续重传”计数重置为零。


我们建议在单独的类中实现重传计时器的功能，但这取决于您。 如果这样做，请将其添加到现有文件（`tcp_sender.hh` 和 `tcp_sender.cc`）中。



### 实现 TCP sender

好的！ 我们已经讨论了 TCP 发送方做什么的基本概念（给定一个传出的 ByteStream，将其分成多个段，将它们发送给接收方，如果它们没有尽快得到确认，则继续重新发送它们）。 我们已经讨论过什么时候可以得出结论，一个未完成的片段丢失了，需要重新发送。 

现在是时候使用 TCPSender 将提供的具体接口了。 它需要处理四个重要事件，每个事件最终都可能发送一个 TCPSegment：

1. `void fill_window()`
	TCPSender 被要求填充窗口：它从其输入 ByteStream 中读取，并以 TCPSegments 的形式发送尽可能多的字节，只要有新的字节要读取并且窗口中有可用空间。

	您需要确保您发送的每个 TCPSegment 都完全适合接收器的窗口。 使每个单独的 TCPSegment 尽可能大，但不大于 `TCPConfig::MAX_PAYLOADSIZE`（1452 字节）给出的值。 

	可以使用 `TCPSegment::length_in_sequence_space()` 方法统计一个段占用的序号总数。 请记住，SYN 和 FIN 标志也各自占用一个序列号，这意味着它们占用了窗口中的空间。

	> 如果窗口大小为零怎么办？ 
	> 如果接收方宣布窗口大小为零，则填充窗口方法应该像窗口大小为一一样。 发送方最终可能会发送一个被接收方拒绝（且未确认）的字节，但这也可能促使接收方发送一个新的确认段，在该段中它显示在其窗口中打开了更多空间。 没有这个，发件人永远不会知道它被允许再次开始发送。

2. `void ack_received( const WrappingInt32 ackno, const uint16 t window size)`
	从接收器接收到一个段，传送窗口的新左（= ackno）和右（= ackno + window size）边缘。 TCPSender 应该查看它的未完成段的集合并删除任何现在已经完全确认的（确认大于段中的所有序列号）。 **如果打开了新空间，TCPSender 应该再次填充窗口。**

3. `void tick( const size t ms since last tick )`
	时间已经过去：自上次调用此方法以来经过了一定的毫秒数。 发送方可能需要重新传输未完成的段。

4. `void send_empty_segment()`
	TCPSender 应该生成并发送一个 TCPSegment，该 TCPSegment 在序列空间中长度为零，并且序列号设置正确。 如果所有者（您将在下周实现的 TCPConnection）想要发送一个空的 ACK 段，这将非常有用。 
	注意：像这样的段，不占用序列号，不需要被跟踪为“未完成”，并且永远不会被重新传输。

要完成实验 3，请查看文档中的完整界面，网址为 https://cs144.github.io/doc/lab3/class_t_c_p_sender.html 并在 `tcp_sender.hh` 和 `tcp_sender.cc` 文件中实现完整的 TCPSender 公共接口。 我们希望您想要添加私有方法和成员变量，可能还有一个辅助类。



### 测试理论

为了测试你的代码，测试套件会期望它经历一系列的情况——从发送第一个 SYN 段，到发送所有数据，到发送 FIN 段，最后确认 FIN 段。 我们认为您不希望创建更多的状态变量来跟踪这些“状态”——这些状态只是由 TCPSender 类的已经公开的公共接口定义的。 但是为了帮助您理解测试输出，这里是 TCPSender 在流的整个生命周期中的预期演变图。 （在实验 4 之前，您不必担心错误状态或 RST 标志。）

![image.png](https://s2.loli.net/2022/09/09/D5sSgnOjuYltTKZ.png)

### FAQs 和特殊情况

**1 我如何“发送”一个片段？** 

将其推送到分段输出队列。 就您的 TCPSender 而言，请考虑将其推送到此队列后立即发送。 很快，所有者就会出现并弹出它（使用公共的 `segements_out()` 访问器方法）并真正发送它。



**2 等等，我如何既“发送”一个段，又如何跟踪同一段未完成，这样我就知道以后要重新传输什么？ 那我不是必须复制每个片段吗？ 这很浪费吗？**

当您发送包含数据的段时，您可能希望将其推送到段输出队列中，并在数据结构内部保留一份副本，以便您跟踪未完成的段以进行可能的重新传输。 事实证明这并不是很浪费，因为段的有效负载存储为一个引用计数的只读字符串（一个 Buffer 对象）。 所以不用担心——它实际上并没有复制有效负载数据。



**3 在我从接收方获得 ACK 之前，我的 TCPSender 应该假设接收方的窗口大小是多少？**

一字节



**4 如果确认仅部分确认了某些未完成的段，我该怎么办？ 我应该尝试剪掉得到确认的字节吗？**

TCP 发送方可以做到这一点，但就`class`（课程 or 类？）而言，无需花哨。 将每个段视为完全未完成，直到它被完全确认 - 它占用的所有序列号都小于`ackno`。



**5 如果我发送三个包含“a”、“b”和“c”的单独段，但它们从未得到确认，我以后可以在一个包含“abc”的大段中重新传输它们吗？ 还是我必须单独重新传输每个片段？**

再说一遍：TCP 发送者可以做到这一点，但是对于这个`class`（课程 or 类？）的目的，不需要花哨。 只需单独跟踪每个未完成的段，当重传计时器到期时，再次发送最早的未完成段。



 **6 我应该在我的 “outstanding” 数据结构中存储空段并在必要时重新传输它们吗？**

不，唯一应该被跟踪为未完成并可能重新传输的段是那些传送一些数据的段。即， 在序列空间中消耗一些长度。 不占用序列号（无负载、SYN 或 FIN）的段不需要被记住或重新传输。



## 代码

![image.png](https://s2.loli.net/2022/09/13/qMKbiXUxAPRwc12.png)

###  重传计时器

* 启动和判断是否启动了
* 停止
* 计时
* 判断是否超时（启动状态 + 超时）



### tick

* `tick` 被调用，计时器的时间开始流逝
* 如果超时则尝试重传最早的 `outstanding segment`
	* 如果存在最早的 `outstanding segment`
		* 重新放到待发送队列中
		* 如果接收方窗口有空余说明是网络堵塞而不是窗口慢了造成的，使用指数退避将超时时间翻倍，做一个形式化的拥塞控制
		* **连续**重传次数 + 1
		* 重启计时器 



### fill_window

* 能填充的容量 = 现在发送方窗口的容量 减去 `outstanding segment` 的字节数

	* 如果发送方窗口容量为 0 则设为 1

* 如果可以填充窗口

	* 如果还没发过 `syn` 就在 `header`  里设置并标记
	* 设置 `header` 的 `seqno` 为 `next_seqno`
	* 计算 payload 的最大的大小（包含 syn 占用的），并按这个大小从流中读取
		* 注意实际读取的和最大可能不一样，可能流没那么数据
	* 看要不要设置 `fin`
		* `stream` 到达 `eof` 状态
		* 还有 1 空间容量留给 `fin`
		* 没有设置过 `fin`
	* 把读到的数据推到 `segment` 的 `payload` 里
	* 如果 `segment` 不占序列号直接退出，否则发送 `segment` 到 待发送队列，并进行跟踪，更新 next_seqno
	* 如果 `segment` 被设置了 `fin` 也直接推出

	

###  ack_received

* 判断序列号是否合法
* 根据 ackno 去除 `outstanding segment` 的中已经被完全确认的 `segment`
	* 如果发生去除，那么超时时间设为初始超时时间并重启计时器
* 重置连续重传次数
* 重置接收方窗口大小并再次调用 `fill_window`



`tcp_sender.hh`

```c++
#ifndef SPONGE_LIBSPONGE_TCP_SENDER_HH
#define SPONGE_LIBSPONGE_TCP_SENDER_HH

#include "byte_stream.hh"
#include "tcp_config.hh"
#include "tcp_segment.hh"
#include "wrapping_integers.hh"

#include <functional>
#include <map>
#include <queue>

class RetransmissionTimer {
  private:
    size_t _ticks{0};
    bool _started{false};

  public:
    RetransmissionTimer() : _ticks(0), _started(false) {}

    void start() {
        _ticks = 0;
        _started = true;
    }

    void stop() {
        _ticks = 0;
        _started = false;
    }

    void timePass(const size_t ms_since_last_tick) { _ticks += ms_since_last_tick; }

    bool isStart() const { return _started; }

    bool isExpired(const unsigned int timeout) { return _started && _ticks >= timeout; }
};
//! \brief The "sender" part of a TCP implementation.

//! Accepts a ByteStream, divides it up into segments and sends the
//! segments, keeps track of which segments are still in-flight,
//! maintains the Retransmission Timer, and retransmits in-flight
//! segments if the retransmission timer expires.
class TCPSender {
  private:
    //! our initial sequence number, the number for our SYN.
    WrappingInt32 _isn;

    //! outbound queue of segments that the TCPSender wants sent
    std::queue<TCPSegment> _segments_out{};

    //! retransmission timer for the connection
    unsigned int _initial_retransmission_timeout;

    //! outgoing stream of bytes that have not yet been sent
    ByteStream _stream;

    //! the (absolute) sequence number for the next byte to be sent
    uint64_t _next_seqno{0};

    RetransmissionTimer _timer{};

    size_t _receiver_window_size{1};

    //! number of consecutive retransmissions
    size_t _consecutive_retransmission_count{0};

    bool _set_syn_flag{false};

    bool _set_fin_flag{false};

    size_t _outstanding_bytes{0};

    std::map<size_t, TCPSegment> _outstanding_segments{};

    unsigned int _curr_rto;

  public:
    //! Initialize a TCPSender
    TCPSender(const size_t capacity = TCPConfig::DEFAULT_CAPACITY,
              const uint16_t retx_timeout = TCPConfig::TIMEOUT_DFLT,
              const std::optional<WrappingInt32> fixed_isn = {});

    //! \name "Input" interface for the writer
    //!@{
    ByteStream &stream_in() { return _stream; }
    const ByteStream &stream_in() const { return _stream; }
    //!@}

    //! \name Methods that can cause the TCPSender to send a segment
    //!@{

    //! \brief A new acknowledgment was received
    void ack_received(const WrappingInt32 ackno, const uint16_t window_size);

    //! \brief Generate an empty-payload segment (useful for creating empty ACK segments)
    void send_empty_segment();

    //! \brief create and send segments to fill as much of the window as possible
    void fill_window();

    //! \brief Notifies the TCPSender of the passage of time
    void tick(const size_t ms_since_last_tick);
    //!@}

    //! \name Accessors
    //!@{

    //! \brief How many sequence numbers are occupied by segments sent but not yet acknowledged?
    //! \note count is in "sequence space," i.e. SYN and FIN each count for one byte
    //! (see TCPSegment::length_in_sequence_space())
    size_t bytes_in_flight() const;

    //! \brief Number of consecutive retransmissions that have occurred in a row
    unsigned int consecutive_retransmissions() const;

    //! \brief TCPSegments that the TCPSender has enqueued for transmission.
    //! \note These must be dequeued and sent by the TCPConnection,
    //! which will need to fill in the fields that are set by the TCPReceiver
    //! (ackno and window size) before sending.
    std::queue<TCPSegment> &segments_out() { return _segments_out; }
    //!@}

    //! \name What is the next sequence number? (used for testing)
    //!@{

    //! \brief absolute seqno for the next byte to be sent
    uint64_t next_seqno_absolute() const { return _next_seqno; }

    //! \brief relative seqno for the next byte to be sent
    WrappingInt32 next_seqno() const { return wrap(_next_seqno, _isn); }
    //!@}

    // help send segment
    void send_segment(TCPSegment &segment);
    // help trace segment
    void trace_segment(TCPSegment &segment);
};

#endif  // SPONGE_LIBSPONGE_TCP_SENDER_HH

```



`tcp_sender.cc`

```c++
#include "tcp_sender.hh"

#include "tcp_config.hh"

#include <random>

// Dummy implementation of a TCP sender

// For Lab 3, please replace with a real implementation that passes the
// automated checks run by `make check_lab3`.

template <typename... Targs>
void DUMMY_CODE(Targs &&.../* unused */) {}

using namespace std;

//! \param[in] capacity the capacity of the outgoing byte stream
//! \param[in] retx_timeout the initial amount of time to wait before retransmitting the oldest outstanding segment
//! \param[in] fixed_isn the Initial Sequence Number to use, if set (otherwise uses a random ISN)
TCPSender::TCPSender(const size_t capacity, const uint16_t retx_timeout, const std::optional<WrappingInt32> fixed_isn)
    : _isn(fixed_isn.value_or(WrappingInt32{random_device()()}))
    , _initial_retransmission_timeout{retx_timeout}
    , _stream(capacity)
    , _curr_rto{retx_timeout} {}

uint64_t TCPSender::bytes_in_flight() const { return _outstanding_bytes; }

void TCPSender::send_segment(TCPSegment &segment) {
    _segments_out.emplace(segment);
    // if the timer is not running, start it running
    if (!_timer.isStart()) {
        _timer.start();
    }
}

void TCPSender::trace_segment(TCPSegment &segment) {
    _outstanding_bytes += segment.length_in_sequence_space();
    _outstanding_segments.emplace(next_seqno_absolute(), segment);
}

void TCPSender::fill_window() {
    size_t cur_window_size = _receiver_window_size ? _receiver_window_size : 1;

    while (cur_window_size > _outstanding_bytes) {
        TCPSegment segment;

        // syn
        if (!_set_syn_flag) {
            segment.header().syn = true;
            _set_syn_flag = true;
        }
        // occupy ?
        segment.header().seqno = next_seqno();

        // payload
        size_t payload_size =
            min(cur_window_size - _outstanding_bytes - segment.header().syn, TCPConfig::MAX_PAYLOAD_SIZE);

        string payload = _stream.read(payload_size);  // payload.size() != payload_size !!!

        // fin
        if (_stream.eof() && !_set_fin_flag && payload.size() + _outstanding_bytes + 1 <= cur_window_size) {
            _set_fin_flag = true;
            segment.header().fin = true;
        }

        segment.payload() = Buffer(std::move(payload));
        // empty data
        if (segment.length_in_sequence_space() == 0) {
            break;
        }

        send_segment(segment);
        trace_segment(segment);
        _next_seqno += segment.length_in_sequence_space();

        if (segment.header().fin) {
            break;
        }
    }
}

//! \param ackno The remote receiver's ackno (acknowledgment number)
//! \param window_size The remote receiver's advertised window size
void TCPSender::ack_received(const WrappingInt32 ackno, const uint16_t window_size) {
    uint64_t abs_seqno = unwrap(ackno, _isn, _next_seqno);
    if (abs_seqno > _next_seqno) {
        return;
    }

    // remove any that have now been fully acknowledged
    for (auto iter = _outstanding_segments.begin(); iter != _outstanding_segments.end();) {
        if (iter->first + iter->second.length_in_sequence_space() - 1 < abs_seqno) {
            _outstanding_bytes -= iter->second.length_in_sequence_space();
            iter = _outstanding_segments.erase(iter);

            // Set the RTO back to its “initial value.”
            _curr_rto = _initial_retransmission_timeout;
            // If the sender has any outstanding data, restart the retransmission timer so that it
            // will expire after RTO milliseconds (for the current value of RTO).
            _timer.start();
        } else {
            break;
        }
    }

    // When all outstanding data has been acknowledged, stop the retransmission timer
    if (!bytes_in_flight()) {
        _timer.stop();
    }

    // Reset the count of “consecutive retransmissions” back to zero.
    _consecutive_retransmission_count = 0;

    _receiver_window_size = window_size;
    fill_window();
}

//! \param[in] ms_since_last_tick the number of milliseconds since the last call to this method
void TCPSender::tick(const size_t ms_since_last_tick) {
    // time passing comes from the tick method being called
    _timer.timePass(ms_since_last_tick);

    //  retransmission timer has expired
    if (_timer.isExpired(_curr_rto)) {
        auto earliest_outstanding_segment = _outstanding_segments.begin();
        if (earliest_outstanding_segment != _outstanding_segments.end()) {
            // Retransmit the earliest (the lowest sequence number) segment that hasn’t been fully
            // acknowledged by the TCP receiver
            // You do not need to track resended segments !!!
            send_segment(_outstanding_segments.begin()->second);

            if (_receiver_window_size > 0) {  // window size > 0 && timeout --> network congestion
                // Double the value of RTO. This is called “exponential backoff”
                _curr_rto = _curr_rto * 2;
            }

            // Keep track of the number of consecutive retransmissions, and increment it
            _consecutive_retransmission_count++;

            // Reset the retransmission timer and start it
            _timer.start();
        }
    }
}

unsigned int TCPSender::consecutive_retransmissions() const { return _consecutive_retransmission_count; }

void TCPSender::send_empty_segment() {  // help to send ACK seg
    TCPSegment segment;
    segment.header().seqno = next_seqno();
    _segments_out.emplace(segment);
}

```



## 测试

```c++
Test project /home/zsl/CLionProjects/sponge/build
      Start  1: t_wrapping_ints_cmp
 1/33 Test  #1: t_wrapping_ints_cmp ..............   Passed    0.01 sec
      Start  2: t_wrapping_ints_unwrap
 2/33 Test  #2: t_wrapping_ints_unwrap ...........   Passed    0.00 sec
      Start  3: t_wrapping_ints_wrap
 3/33 Test  #3: t_wrapping_ints_wrap .............   Passed    0.00 sec
      Start  4: t_wrapping_ints_roundtrip
 4/33 Test  #4: t_wrapping_ints_roundtrip ........   Passed    0.16 sec
      Start  5: t_recv_connect
 5/33 Test  #5: t_recv_connect ...................   Passed    0.00 sec
      Start  6: t_recv_transmit
 6/33 Test  #6: t_recv_transmit ..................   Passed    0.03 sec
      Start  7: t_recv_window
 7/33 Test  #7: t_recv_window ....................   Passed    0.00 sec
      Start  8: t_recv_reorder
 8/33 Test  #8: t_recv_reorder ...................   Passed    0.00 sec
      Start  9: t_recv_close
 9/33 Test  #9: t_recv_close .....................   Passed    0.00 sec
      Start 10: t_recv_special
10/33 Test #10: t_recv_special ...................   Passed    0.00 sec
      Start 11: t_send_connect
11/33 Test #11: t_send_connect ...................   Passed    0.00 sec
      Start 12: t_send_transmit
12/33 Test #12: t_send_transmit ..................   Passed    0.03 sec
      Start 13: t_send_retx
13/33 Test #13: t_send_retx ......................   Passed    0.00 sec
      Start 14: t_send_window
14/33 Test #14: t_send_window ....................   Passed    0.02 sec
      Start 15: t_send_ack
15/33 Test #15: t_send_ack .......................   Passed    0.01 sec
      Start 16: t_send_close
16/33 Test #16: t_send_close .....................   Passed    0.01 sec
      Start 17: t_send_extra
17/33 Test #17: t_send_extra .....................   Passed    0.01 sec
      Start 18: t_strm_reassem_single
18/33 Test #18: t_strm_reassem_single ............   Passed    0.00 sec
      Start 19: t_strm_reassem_seq
19/33 Test #19: t_strm_reassem_seq ...............   Passed    0.00 sec
      Start 20: t_strm_reassem_dup
20/33 Test #20: t_strm_reassem_dup ...............   Passed    0.01 sec
      Start 21: t_strm_reassem_holes
21/33 Test #21: t_strm_reassem_holes .............   Passed    0.00 sec
      Start 22: t_strm_reassem_many
22/33 Test #22: t_strm_reassem_many ..............   Passed    0.62 sec
      Start 23: t_strm_reassem_overlapping
23/33 Test #23: t_strm_reassem_overlapping .......   Passed    0.00 sec
      Start 24: t_strm_reassem_win
24/33 Test #24: t_strm_reassem_win ...............   Passed    0.79 sec
      Start 25: t_strm_reassem_cap
25/33 Test #25: t_strm_reassem_cap ...............   Passed    0.08 sec
      Start 26: t_byte_stream_construction
26/33 Test #26: t_byte_stream_construction .......   Passed    0.00 sec
      Start 27: t_byte_stream_one_write
27/33 Test #27: t_byte_stream_one_write ..........   Passed    0.00 sec
      Start 28: t_byte_stream_two_writes
28/33 Test #28: t_byte_stream_two_writes .........   Passed    0.00 sec
      Start 29: t_byte_stream_capacity
29/33 Test #29: t_byte_stream_capacity ...........   Passed    0.25 sec
      Start 30: t_byte_stream_many_writes
30/33 Test #30: t_byte_stream_many_writes ........   Passed    0.01 sec
      Start 53: t_address_dt
31/33 Test #53: t_address_dt .....................   Passed    0.14 sec
      Start 54: t_parser_dt
32/33 Test #54: t_parser_dt ......................   Passed    0.00 sec
      Start 55: t_socket_dt
33/33 Test #55: t_socket_dt ......................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 33

Total Test time (real) =   2.27 sec
[100%] Built target check_lab3

```

