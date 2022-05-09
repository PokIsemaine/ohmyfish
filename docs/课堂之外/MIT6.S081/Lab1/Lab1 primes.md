# Lab: Xv6 and Unix utilities

## primesg

<div class="required">
  <p>Write a concurrent version of prime sieve using pipes.  This idea
    is due to Doug McIlroy, inventor of Unix pipes.  The picture
    halfway down <a href="http://swtch.com/~rsc/thread/">this page</a>
    and the surrounding text explain how to do it.  Your
    solution should be in the file <tt>user/primes.c</tt>.
</div>
<p>Your goal is to use <tt>pipe</tt> and <tt>fork</tt> to set up
the pipeline. The first process feeds the numbers 2 through 35
into the pipeline.  For each prime number, you will arrange to
create one process that reads from its left neighbor over a pipe
and writes to its right neighbor over another pipe. Since xv6 has
limited number of file descriptors and processes, the first
process can stop at 35.</p>



<p>Some hints:
  <ul>
    <li>Be careful to close file descriptors that a process doesn't
    need, because otherwise your program will run xv6 out of resources
    before the first process reaches 35.
    <li>Once the first process reaches 35, it should wait until the
entire pipeline terminates, including all children, grandchildren,
&c. Thus the main primes process should only exit after all the
output has been printed, and after all the other primes processes
have exited.
    <li>Hint: <tt>read</tt> returns zero when the write-side of
a pipe is closed.
	<li>It's simplest to directly write 32-bit (4-byte) <tt>int</tt>s to the
    pipes, rather than using formatted ASCII I/O.
	<li>You should create the processes in the pipeline only as they are
  needed.
	<li>Add the program to <tt>UPROGS</tt> in Makefile.
  </ul>


<p>Your solution is correct if it implements a pipe-based
sieve and produces the following output:
  <pre>
    $ <kbd>make qemu</kbd>
    ...
    init: starting sh
    $ <kbd>primes</kbd>
    prime 2
    prime 3
    prime 5
    prime 7
    prime 11
    prime 13
    prime 17
    prime 19
    prime 23
    prime 29
    prime 31
    $
  </pre>

## 需求

利用`pipe`和`fork`写一个并发素数筛

## 思路

![image-20220404212942209](../../../../../../.config/Typora/typora-user-images/image-20220404212942209.png)

素数筛法其实就是埃式筛的筛倍数，然后利用`pipe`和`fork`实现并发

CSP并发模型

https://zhuanlan.zhihu.com/p/455843256

https://www.cnblogs.com/sunsky303/p/9115530.html#:~:text=CSP

* 共享内存与CSP（不要用buf?)
* CSP与Actor 耦合？





## 代码

```c
#include "../kernel/types.h"
#include "../kernel/stat.h"
#include "../user/user.h"

#define PIPE_READ 0
#define PIPE_WRITE 1

int 
getPrime(int *pipefd){
	close(pipefd[PIPE_WRITE]);
	int next_pipefd[2];
	pipe(next_pipefd);

	int filter;
	if(read(pipefd[PIPE_READ],&filter,sizeof(filter)) == sizeof(filter)) {
		printf("prime %d\n",filter);
		int pid = fork();
		if(0 == pid) {
			//children
			getPrime(next_pipefd);
			exit(0);
		} else if(0 < pid) {
			//parent
			close(next_pipefd[PIPE_READ]);
			int num;
			while(read(pipefd[PIPE_READ],&num,sizeof(num)) == sizeof(num)) {
				if(0 != num % filter) {
					write(next_pipefd[PIPE_WRITE],&num,sizeof(num));
				}
			}
			close(next_pipefd[PIPE_WRITE]);
			wait((int*)0);
		} else {
			fprintf(2,"fork error\n");
			exit(1);
		}
	}
	exit(0);
}

int
main(void) {
	int pipefd[2];
	pipe(pipefd);

	int pid = fork();

	if(0 == pid) {
		//children 
		getPrime(pipefd);
		exit(0);
	} else if(0 < pid) {
		//parent
		close(pipefd[PIPE_READ]);
		for(int i = 2; i <= 35; ++i) {
			write(pipefd[PIPE_WRITE],&i,sizeof(i));
		}
		close(pipefd[PIPE_WRITE]);
		wait((int*)0);

	} else {
		fprintf(2,"fork error\n");
		exit(1);
	}

	exit(0);
} 
```

