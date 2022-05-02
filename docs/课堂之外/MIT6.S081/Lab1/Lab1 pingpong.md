<h1>Lab: Xv6 and Unix utilities</h1>
<h2>pingpong <script>g("easy")</script></h2>

<div class="required">
<p> Write a program that uses UNIX system calls to ''ping-pong'' a
  byte between two processes over a pair of pipes, one for each
  direction.
  The parent should send a byte to the child;
  the child should print "&lt;pid&gt;: received ping",
  where &lt;pid&gt; is its process ID,
  write the byte on the pipe to the parent,
  and exit;
  the parent should read the byte from the child,
  print "&lt;pid&gt;: received pong",
  and exit.
  Your
  solution should be in the file <tt>user/pingpong.c</tt>.
</div>

<p>Some hints:
  <ul>
    <li>Use <tt>pipe</tt> to create a pipe.
    <li>Use <tt>fork</tt> to create a child.
    <li>Use <tt>read</tt> to read from the pipe, and <tt>write</tt> to write to the pipe.
    <li>Use <tt>getpid</tt> to find the process ID of the calling process.
    <li>Add the program to <tt>UPROGS</tt> in Makefile.
    <li>User programs on xv6 have a limited set of library
    functions available to them. You can see the list in
    <tt>user/user.h</tt>; the source (other than for system calls)
    is in <tt>user/ulib.c</tt>, <tt>user/printf.c</tt>,
    and <tt>user/umalloc.c</tt>.
  </ul>
  <p>Run the program from the xv6 shell and it should produce the
  following output:
  <pre>
    $ <kbd>make qemu</kbd>
    ...
    init: starting sh
    $ <kbd>pingpong</kbd>
    4: received ping
    3: received pong
    $
  </pre>
    <p>Your solution is correct if your program exchanges a byte
    between two processes and produces output as shown above.



## 需求

通过管道实现父子进程通信

1. 父进程通过管道向子进程发送"ping"
2. 子进程从管道读取"ping"
3. 子进程打印"pid: received ping"
4. 子进程通过管道向父进程发送"pong"
5. 父进程从管道读取"pong"
6. 父进程打印"pid: received pong"

答案写在`user/pingpong.c`

## 思路

### 创建两个管道

![image-20220404170543583](https://s2.loli.net/2022/04/04/Toi3naIkz4Crjet.png)

### fork()创建子进程

![image-20220404170515547](https://s2.loli.net/2022/04/04/vXBSFA7ZbTaGPY4.png)



### 关闭对应读写端

![image-20220404170730923](https://s2.loli.net/2022/04/04/vzjcdHBsqmJhi3f.png)

### 读写数据

* 读管道：
	* 管道中有数据，read返回实际读到的字节数。
	* 管道中无数据
		* 写端被全部关闭，read返回0（相当于读到文件的末尾）
		* 写端没有完全关闭，read阻塞等待
* 写管道：
	* 管道读端全部被关闭，进程异常终止（进程收到SIGPIPE信号）
	* 管道读端没有全部关闭：
		        管道已满，write阻塞
		        管道没有满，write将数据写入，并返回实际写入的字节数

记得写完后关闭管道，否则read会一直阻塞等待

## 答案

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(void) {
	char buf[15];
	
	int ctp[2];//children->parent
	int ptc[2];//paretn->children

	pipe(ctp);
	pipe(ptc);

	int pid = fork();
	if(pid ==0) {	//children 
	//children->parent
		close(ctp[0]);
		close(ptc[1]);
		read(ptc[0],buf,4);
		printf("%d: received %s\n",getpid(),buf);
		write(ctp[1],"pong",4);
		close(ctp[1]);//close ctp write,write finish
	} else if(pid > 0){	//parent
	//paretn->children
		close(ctp[1]);	
		close(ptc[0]);
		write(ptc[1],"ping",4);
		close(ptc[1]);//close ptc write,write finish
		read(ctp[0],buf,4);
		printf("%d: received %s\n",getpid(),buf);
	} else {	//error
		fprintf(2,"fork error");
		exit(1);
	}

	exit(0);
} 
```

