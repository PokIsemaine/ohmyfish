# Lab0

实验手册：https://cs144.github.io/assignments/lab0.pdf

前面几个小练习就不谈了

## 实现 webget

### 设计

`webget` 是一个使用操作系统的 TCP 支持和流套接字抽象在 `Internet` 上获取网页的程序

```c++
// 提供 host 和 path 获取网页
// Usage	" HOST PATH\n";
// Example	" stanford.edu /class/cs144\n";
void get_URL(const string &host, const string &path);
```

主要靠 `TCPSocket` 和 `Address` 来实现

`TCPSocket`文档：https://cs144.github.io/doc/lab0/class_t_c_p_socket.html

```c++
const uint16_t portnum = ((std::random_device()()) % 50000) + 1025;
 
// create a TCP socket, bind it to a local address, and listen
TCPSocket sock1;
sock1.bind(Address("127.0.0.1", portnum));
sock1.listen(1);
 
// create another socket and connect to the first one
TCPSocket sock2;
sock2.connect(Address("127.0.0.1", portnum));
 
// accept the connection
auto sock3 = sock1.accept();
sock3.write("hi there");
 
auto recvd = sock2.read();
sock2.write("hi yourself");
 
auto recvd2 = sock3.read();
 
sock1.close();              // don't need to accept any more connections
sock2.close();              // you can call close(2) on a socket
sock3.shutdown(SHUT_RDWR);  // you can also shutdown(2) a socket
if (recvd != "hi there" || recvd2 != "hi yourself") {
    throw std::runtime_error("wrong data received");
}
```



`Address`文档：https://cs144.github.io/doc/lab0/class_address.html

```c++
// Wrapper around IPv4 addresses and DNS operations.

// For example, you can do DNS lookups:

const Address google_webserver("www.google.com", "https");
// or you can specify an IP address and port number:

const Address a_dns_server("18.71.0.151", 53);
// Once you have an address, you can convert it to other useful representations, e.g.,

const uint32_t a_dns_server_numeric = a_dns_server.ipv4_numeric();
```

按照下面 TCP 客户端的流程编写代码

![img](https://img-blog.csdn.net/20161125153219394)





发送请求的时候，将实验手册的 `Fetch a Web page` 练习转换为代码即可，换行对应 `\r\n`

![image-20220830080419538](../../../../../.config/Typora/typora-user-images/image-20220830080419538.png)



### 代码实现

```c++
#include "socket.hh"
#include "util.hh"

#include <cstdlib>
#include <iostream>

using namespace std;

void get_URL(const string &host, const string &path) {
    // Your code here.

    // You will need to connect to the "http" service on
    // the computer whose name is in the "host" string,
    // then request the URL path given in the "path" string.

    // Then you'll need to print out everything the server sends back,
    // (not just one call to read() -- everything) until you reach
    // the "eof" (end of file).
    TCPSocket sock;
    // 创建连接
    sock.connect(Address(host, "http"));
    // 发送请求
    sock.write("GET " + path + " HTTP/1.1\r\n");
    sock.write("HOST:" + host + "\r\n");
    sock.write("Connection: close\r\n");
    sock.write("\r\n");
    // 关闭请求
    sock.shutdown(SHUT_WR);
    // 读取内容
    while(!sock.eof()) {
        cout << sock.read();
    }
    // 关闭连接
    sock.close();
}

int main(int argc, char *argv[]) {
    try {
        if (argc <= 0) {
            abort();  // For sticklers: don't try to access argv[0] if argc <= 0.
        }

        // The program takes two command-line arguments: the hostname and "path" part of the URL.
        // Print the usage message unless there are these two arguments (plus the program name
        // itself, so arg count = 3 in total).
        if (argc != 3) {
            cerr << "Usage: " << argv[0] << " HOST PATH\n";
            cerr << "\tExample: " << argv[0] << " stanford.edu /class/cs144\n";
            return EXIT_FAILURE;
        }

        // Get the command-line arguments.
        const string host = argv[1];
        const string path = argv[2];
        // Call the student-written function.
        get_URL(host, path);
    } catch (const exception &e) {
        cerr << e.what() << "\n";
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}

```

### 测试

![image-20220830081244576](../../../../../.config/Typora/typora-user-images/image-20220830081244576.png)



## 可靠内存字节流

为了结束本周的实验，您将在单台计算机的内存中实现一个提供此抽象的对象。 （您可能在 CS 110 中做过类似的事情。）

* 字节被写入“输入”端，并且可以以相同的顺序从“输出”端读取。 字节流是有限的：`writer` 可以结束输入，然后就不能再写入字节了。 当 `reader` 读取到流的末尾时，它将到达“EOF”（文件结尾）并且无法读取更多字节。

* 您的字节流也将受到流量控制，以在**任何**给定时间限制其内存消耗。 对象使用特定的“容量”进行初始化：在任何给定点它愿意在自己的内存中存储的最大字节数。 字节流将限制 `writer` 在**任何**给定时刻可以写入的数量，以确保流不会超过其存储容量。 当 `reader` 读取字节并将它们从流中排出时，`writer` 被允许写入更多。
*  您的字节流用于单线程——您不必担心并发写入者
* 需要明确的是：字节流是有限的，但在`writer`结束输入并完成流之前，它几乎可以是任意长度 。 您的实现必须能够处理比容量长得多的流。 **容量限制在给定点保存在内存中（已写入但尚未读取）的字节数，但不限制流的长度。** 一个只有一个字节容量的对象仍然可以携带一个 TB 和 TB 长的流，只要写入器保持一次写入一个字节并且读取器在写入器被允许写入下一个字节之前读取每个字节

FAQ：

1. **如果调用者试图以`len`比可用的更大的值`peek`或`pop`，我的程序的行为应该是什么？**
2. 如果调用者试图 `peek` 或 `pop` 比流中可用的更多内容，我们对行为没有任何偏好，只要您行为合理并且不会在这种情况下崩溃。如果您想 `peek` / `pop` 可用的最大值，我们可以。如果您想抛出异常（对于超长 `peek` 或超长`pop`），我们也可以。

3. **我的代码应该有多快？**
4. 这是一个合理的经验法则：如果您的任何测试花费的时间超过 2 秒，那么继续探索表示 ByteStream 的其他方式可能会很好。该实验室不需要任何复杂的数据结构，但可能仍需要反复试验才能找到合理的方法。有时“big-O”分析可能会误导实际程序的性能——常数因素可能很重要！


 ![image-20220830091920995](../../../../../.config/Typora/typora-user-images/image-20220830091920995.png)

### 设计

* 使用`deque<char>` 来作为 `ByteStream` 的基本数据结构
* 如果容量不够了，那么能写多少写多少，超出的直接抛弃，通过`wrtie`的返回值告知具体写了多少
* 判断`eof`的条件：字节流写入结束，不再写入内存；并且把内存中的字节流全部读取完毕
* 读太多：如果`peek`或者`pop`的`len`参数比内存中未读取的字节流长度还大，那么自动调整为读取长度，能读多少就读多少



### 代码实现

`byte_stream.hh`

```c++
#ifndef SPONGE_LIBSPONGE_BYTE_STREAM_HH
#define SPONGE_LIBSPONGE_BYTE_STREAM_HH

#include <string>
#include <deque>

//! \brief An in-order byte stream.

//! Bytes are written on the "input" side and read from the "output"
//! side.  The byte stream is finite: the writer can end the input,
//! and then no more bytes can be written.
class ByteStream {
  private:
    // Your code here -- add private members as necessary.

    // Hint: This doesn't need to be a sophisticated data structure at
    // all, but if any of your tests are taking longer than a second,
    // that's a sign that you probably want to keep exploring
    // different approaches.
    std::deque<char> m_buffer = {};
    size_t m_capacity;
    size_t m_bytesWritten = 0;
    size_t m_bytesRead = 0;
    bool is_end = false;
    bool _error{};  //!< Flag indicating that the stream suffered an error.

  public:
    //! Construct a stream with room for `capacity` bytes.
    ByteStream(const size_t capacity);

    //! \name "Input" interface for the writer
    //!@{

    //! Write a string of bytes into the stream. Write as many
    //! as will fit, and return how many were written.
    //! \returns the number of bytes accepted into the stream
    size_t write(const std::string &data);

    //! \returns the number of additional bytes that the stream has space for
    size_t remaining_capacity() const;

    //! Signal that the byte stream has reached its ending
    void end_input();

    //! Indicate that the stream suffered an error.
    void set_error() { _error = true; }
    //!@}

    //! \name "Output" interface for the reader
    //!@{

    //! Peek at next "len" bytes of the stream
    //! \returns a string
    std::string peek_output(const size_t len) const;

    //! Remove bytes from the buffer
    void pop_output(const size_t len);

    //! Read (i.e., copy and then pop) the next "len" bytes of the stream
    //! \returns a string
    std::string read(const size_t len);

    //! \returns `true` if the stream input has ended
    bool input_ended() const;

    //! \returns `true` if the stream has suffered an error
    bool error() const { return _error; }

    //! \returns the maximum amount that can currently be read from the stream
    size_t buffer_size() const;

    //! \returns `true` if the buffer is empty
    bool buffer_empty() const;

    //! \returns `true` if the output has reached the ending
    bool eof() const;
    //!@}

    //! \name General accounting
    //!@{

    //! Total number of bytes written
    size_t bytes_written() const;

    //! Total number of bytes popped
    size_t bytes_read() const;
    //!@}
};

#endif  // SPONGE_LIBSPONGE_BYTE_STREAM_HH

```



`byte_stream.cc`

```c++
#include "byte_stream.hh"
// Dummy implementation of a flow-controlled in-memory byte stream.

// For Lab 0, please replace with a real implementation that passes the
// automated checks run by `make check_lab0`.

// You will need to add private members to the class declaration in `byte_stream.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

ByteStream::ByteStream(const size_t capacity)\
    : m_capacity(capacity) {}

size_t ByteStream::write(const string &data) {
    size_t remain = remaining_capacity();
    size_t written = 0;
    for(size_t i = 0; i < min(remain, data.size()); ++i) {
        m_buffer.emplace_back(data[i]);
        ++written;
    }
    m_bytesWritten += written;
    return written;
}

//! \param[in] len bytes will be copied from the output side of the buffer
string ByteStream::peek_output(const size_t len) const {
    string ret;
    size_t curSize = buffer_size();
    for(size_t i = 0; i < min(len, curSize); ++i) {
        ret += m_buffer[i];
    }
    return ret;
}

//! \param[in] len bytes will be removed from the output side of the buffer
void ByteStream::pop_output(const size_t len) {
    size_t curSize = buffer_size();
    for(size_t i = 0; i < min(len, curSize); ++i) {
        m_buffer.pop_front();
        ++m_bytesRead;
    }
}

//! Read (i.e., copy and then pop) the next "len" bytes of the stream
//! \param[in] len bytes will be popped and returned
//! \returns a string
std::string ByteStream::read(const size_t len) {
    string ret = peek_output(len);
    pop_output(len);
    return ret;
}

void ByteStream::end_input() { is_end = true; }

bool ByteStream::input_ended() const { return is_end; }

size_t ByteStream::buffer_size() const { return m_buffer.size(); }

bool ByteStream::buffer_empty() const { return m_buffer.empty(); }

bool ByteStream::eof() const { return buffer_empty() && input_ended(); }

size_t ByteStream::bytes_written() const { return m_bytesWritten; }

size_t ByteStream::bytes_read() const { return m_bytesRead; }

size_t ByteStream::remaining_capacity() const { return m_capacity - m_buffer.size(); }

```



### 测试

![image-20220830104910777](../../../../../.config/Typora/typora-user-images/image-20220830104910777.png)
