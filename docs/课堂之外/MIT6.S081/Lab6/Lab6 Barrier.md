# Lab6 Barrier

## 要求

In this assignment you'll implement a [barrier](http://en.wikipedia.org/wiki/Barrier_(computer_science)): a point in an application at which all participating threads must wait until all other participating threads reach that point too. You'll use pthread condition variables, which are a sequence coordination technique similar to xv6's sleep and wakeup.

You should do this assignment on a real computer (not xv6, not qemu).

The file `notxv6/barrier.c` contains a broken barrier.

```sh
$ make barrier
$ ./barrier 2
barrier: notxv6/barrier.c:42: thread: Assertion `i == t' failed.
```

The 2 specifies the number of threads that synchronize on the barrier ( `nthread` in `barrier.c`). Each thread executes a loop. In each loop iteration a thread calls `barrier()` and then sleeps for a random number of microseconds. The assert triggers, because one thread leaves the barrier before the other thread has reached the barrier. The desired behavior is that each thread blocks in `barrier()` until all `nthreads` of them have called `barrier()`.

Your goal is to achieve the desired barrier behavior. In addition to the lock primitives that you have seen in the `ph` assignment, you will need the following new pthread primitives; look [here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_wait.html) and [here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_broadcast.html) for details.

```c
pthread_cond_wait(&cond, &mutex);  // go to sleep on cond, releasing lock mutex, acquiring upon wake up
pthread_cond_broadcast(&cond);     // wake up every thread sleeping on cond
```

Make sure your solution passes `make grade`'s `barrier` test.

`pthread_cond_wait` releases the `mutex` when called, and re-acquires the `mutex` before returning.

We have given you `barrier_init()`. Your job is to implement `barrier()` so that the panic doesn't occur. We've defined `struct barrier` for you; its fields are for your use.

There are two issues that complicate your task:

- You have to deal with a succession of barrier calls, each of which we'll call a round. `bstate.round` records the current round. You should increment `bstate.round` each time all threads have reached the barrier.
- You have to handle the case in which one thread races around the loop before the others have exited the barrier. In particular, you are re-using the `bstate.nthread` variable from one round to the next. Make sure that a thread that leaves the barrier and races around the loop doesn't increase `bstate.nthread` while a previous round is still using it.

Test your code with one, two, and more than two threads.

## 解决方案

非 xv6 和 qemu，在有多核处理器的 Linux 真机上编程

`notxv6/ph.c`

利用`pthread`的锁和条件变量来实现`barrier`，`barrier`挡住到达的线程直到所有线程到达才放行



### pthread cond

* 声明一个条件变量：`pthread_cond_t cond`
* ` pthread_cond_wait(&cond, &mutex);`做两件事情
	* 让当前线程在`cond`在这个条件上睡眠，线程状态变为阻塞态（暂时不抢锁了）
	* 释放线程持有的锁
* `pthread_cond_broadcast(&cond); `唤醒所有在这个条件变量上睡眠的线程，线程状态变为就绪态，不过但是还要等待锁释放才能抢到所有权

![img](https://pic4.zhimg.com/v2-af6e36a7023fb6337e3687df2a4b8557_b.jpg)



>https://www.cnblogs.com/alan666/p/8311962.html
>
>* pthread_cond_wait() 用于阻塞当前线程，等待别的线程使用pthread_cond_signal()或pthread_cond_broadcast来唤醒它。 
>* pthread_cond_wait() 必须与pthread_mutex配套使用。pthread_cond_wait()函数一进入wait状态就会自动release mutex。当其他线程通
>* pthread_cond_signal()或pthread_cond_broadcast，把该线程唤醒，使pthread_cond_wait()通过（返回）时，该线程又自动获得该mutex。
>*  pthread_cond_signal函数的作用是发送一个信号给另外一个正在处于阻塞等待状态的线程,使其脱离阻塞状态,继续执行.如果没有线程处在阻塞等待状态,pthread_cond_signal也会成功返回。
>*  使用pthread_cond_signal一般不会有“惊群现象”产生，他最多只给一个线程发信号。假如有多个线程正在阻塞等待着这个条件变量的话，那么是根据各等待线程优先级的高低确定哪个线程接收到信号开始继续执行。如果各线程优先级相同，则根据等待时间的长短来确定哪个线程获得信号。但无论如何一个pthread_cond_signal调用最多发信一次。
>*  但是pthread_cond_signal在多处理器上可能同时唤醒多个线程，当你只能让一个线程处理某个任务时，其它被唤醒的线程就需要继续 wait，而且规范要求pthread_cond_signal至少唤醒一个pthread_cond_wait上的线程，其实有些实现为了简单在单处理器上也会唤醒多个线程. 
>* 另外，某些应用，如线程池，pthread_cond_broadcast唤醒全部线程，但我们通常只需要一部分线程去做执行任务，所以其它的线程需要继续wait.所以强烈推荐对pthread_cond_wait() 使用while循环来做条件判断.

### 实现 barrier

注意保证每个分支执行流锁的获取和释放对称

```c
static void 
barrier()
{
  // YOUR CODE HERE
  //
  // Block until all threads have called barrier() and
  // then increment bstate.round.
  //
    pthread_mutex_lock(&bstate.barrier_mutex);
    ++bstate.nthread;
    if(bstate.nthread < nthread) {
        pthread_cond_wait(&bstate.barrier_cond,&bstate.barrier_mutex);	
        // 包含两个动作
        // 1. 当前线程在这个条件变量上睡眠
        // 2. 释放线程持有的锁
    } else {
        bstate.nthread = 0;
        ++bstate.round;
        pthread_cond_broadcast(&bstate.barrier_cond);	// 唤醒条件变量上的所有线程，但是还没释放锁，所以抢不到
    }
    pthread_mutex_unlock(&bstate.barrier_mutex);	// 释放锁，可以被抢到了
}
```



```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <assert.h>
#include <pthread.h>

static int nthread = 1;
static int round = 0;

struct barrier {
  pthread_mutex_t barrier_mutex;
  pthread_cond_t barrier_cond;
  int nthread;      // Number of threads that have reached this round of the barrier
  int round;     // Barrier round
} bstate;

static void
barrier_init(void)
{
  assert(pthread_mutex_init(&bstate.barrier_mutex, NULL) == 0);
  assert(pthread_cond_init(&bstate.barrier_cond, NULL) == 0);
  bstate.nthread = 0;
}

static void 
barrier()
{
  // YOUR CODE HERE
  //
  // Block until all threads have called barrier() and
  // then increment bstate.round.
  //
    pthread_mutex_lock(&bstate.barrier_mutex);
    ++bstate.nthread;
    if(bstate.nthread < nthread) {
        pthread_cond_wait(&bstate.barrier_cond,&bstate.barrier_mutex);
    } else {
        bstate.nthread = 0;
        ++bstate.round;
        pthread_cond_broadcast(&bstate.barrier_cond);
    }
    pthread_mutex_unlock(&bstate.barrier_mutex);
}

static void *
thread(void *xa)
{
  long n = (long) xa;
  long delay;
  int i;

  for (i = 0; i < 20000; i++) {
    int t = bstate.round;
    assert (i == t);
    barrier();
    usleep(random() % 100);
  }

  return 0;
}

int
main(int argc, char *argv[])
{
  pthread_t *tha;
  void *value;
  long i;
  double t1, t0;

  if (argc < 2) {
    fprintf(stderr, "%s: %s nthread\n", argv[0], argv[0]);
    exit(-1);
  }
  nthread = atoi(argv[1]);
  tha = malloc(sizeof(pthread_t) * nthread);
  srandom(0);

  barrier_init();

  for(i = 0; i < nthread; i++) {
    assert(pthread_create(&tha[i], NULL, thread, (void *) i) == 0);
  }
  for(i = 0; i < nthread; i++) {
    assert(pthread_join(tha[i], &value) == 0);
  }
  printf("OK; passed\n");
}

```



