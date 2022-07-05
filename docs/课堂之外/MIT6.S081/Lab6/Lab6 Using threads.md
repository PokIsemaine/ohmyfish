# Lab6 Using threads

## 要求

In this assignment you will explore parallel programming with threads and locks using a hash table. You should do this assignment on a real Linux or MacOS computer (not xv6, not qemu) that has multiple cores. Most recent laptops have multicore processors.

This assignment uses the UNIX `pthread` threading library. You can find information about it from the manual page, with `man pthreads`, and you can look on the web, for example [here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_lock.html), [here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_init.html), and [here](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_create.html).

The file `notxv6/ph.c` contains a simple hash table that is correct if used from a single thread, but incorrect when used from multiple threads. In your main xv6 directory (perhaps `~/xv6-labs-2021`), type this:

```
$ make ph
$ ./ph 1
```

Note that to build `ph` the Makefile uses your OS's gcc, not the 6.S081 tools. The argument to `ph` specifies the number of threads that execute put and get operations on the the hash table. After running for a little while, `ph 1` will produce output similar to this:

```
100000 puts, 3.991 seconds, 25056 puts/second
0: 0 keys missing
100000 gets, 3.981 seconds, 25118 gets/second
```

The numbers you see may differ from this sample output by a factor of two or more, depending on how fast your computer is, whether it has multiple cores, and whether it's busy doing other things.

`ph` runs two benchmarks. First it adds lots of keys to the hash table by calling `put()`, and prints the achieved rate in puts per second. The it fetches keys from the hash table with `get()`. It prints the number keys that should have been in the hash table as a result of the puts but are missing (zero in this case), and it prints the number of gets per second it achieved.

You can tell `ph` to use its hash table from multiple threads at the same time by giving it an argument greater than one. Try `ph 2`:

```
$ ./ph 2
100000 puts, 1.885 seconds, 53044 puts/second
1: 16579 keys missing
0: 16579 keys missing
200000 gets, 4.322 seconds, 46274 gets/second
```

The first line of this `ph 2` output indicates that when two threads concurrently add entries to the hash table, they achieve a total rate of 53,044 inserts per second. That's about twice the rate of the single thread from running `ph 1`. That's an excellent "parallel speedup" of about 2x, as much as one could possibly hope for (i.e. twice as many cores yielding twice as much work per unit time).

However, the two lines saying `16579 keys missing` indicate that a large number of keys that should have been in the hash table are not there. That is, the puts were supposed to add those keys to the hash table, but something went wrong. Have a look at `notxv6/ph.c`, particularly at `put()` and `insert()`.

Why are there missing keys with 2 threads, but not with 1 thread? Identify a sequence of events with 2 threads that can lead to a key being missing. Submit your sequence with a short explanation in answers-thread.txt

To avoid this sequence of events, insert lock and unlock statements in `put` and `get` in `notxv6/ph.c` so that the number of keys missing is always 0 with two threads. The relevant pthread calls are:

```c
pthread_mutex_t lock;            // declare a lock
pthread_mutex_init(&lock, NULL); // initialize the lock
pthread_mutex_lock(&lock);       // acquire lock
pthread_mutex_unlock(&lock);     // release lock
```

You're done when `make grade` says that your code passes the `ph_safe` test, which requires zero missing keys with two threads. It's OK at this point to fail the `ph_fast` test.

Don't forget to call `pthread_mutex_init()`. Test your code first with 1 thread, then test it with 2 threads. Is it correct (i.e. have you eliminated missing keys?)? Does the two-threaded version achieve parallel speedup (i.e. more total work per unit time) relative to the single-threaded version?

There are situations where concurrent `put()`s have no overlap in the memory they read or write in the hash table, and thus don't need a lock to protect against each other. Can you change `ph.c` to take advantage of such situations to obtain parallel speedup for some `put()`s? Hint: how about a lock per hash bucket?

Modify your code so that some `put` operations run in parallel while maintaining correctness. You're done when `make grade` says your code passes both the `ph_safe` and `ph_fast` tests. The `ph_fast` test requires that two threads yield at least 1.25 times as many puts/second as one thread.



## 解决方案

非 xv6 和 qemu，在有多核处理器的 Linux 真机上编程

`notxv6/ph.c`给了一个单线程下正确的哈希表，任务就是利用`pthread`来实现一个多线程哈希表



### pthread mutex

那么首先熟悉一下 `pthread`库的`API`

**RTFM**：https://www.man7.org/linux/man-pages/man0/pthread.h.0p.html

* 声明一把锁`pthread_mutex_t lock;`

* 初始化锁`pthread_mutex_init(&lock, NULL);` 

* 获取锁`pthread_mutex_lock(&lock);`

* 释放锁`pthread_mutex_unlock(&lock); `

* 创建线程`pthread_create`是共四个参数：

	- 第一个参数为指向线程[标识符](http://baike.baidu.com/view/390932.htm)的指针，type: `pthread_t*`

	- 第二个参数用来设置线程属性

	- 第三个参数是线程运行函数的起始地址， type: `(void*)(*)(void*)`

	- 第四个参数是运行函数的参数，type: `void *`

* 等待一个线程的结束`pthread_join`线程间同步的操作 ，共两个参数：

	- 第一个参数为线程标识符，即线程ID，type: `pthread_t`

	- 第二个参数retval为用户定义的指针，用来存储线程的返回值，`type: void**`

	

### 单线程哈希表

先来看看哈希表结构体

```c
struct entry {
  int key;
  int value;
  struct entry *next;
};
struct entry *table[NBUCKET];
int keys[NKEYS];
```

可以发现是一个链式的哈希表

![img](https://images2017.cnblogs.com/blog/1281268/201712/1281268-20171206081611534-142637156.png)

```c
static void 
insert(int key, int value, struct entry **p, struct entry *n)
{
  struct entry *e = malloc(sizeof(struct entry));
  e->key = key;
  e->value = value;
  e->next = n;
  *p = e;
}

static 
void put(int key, int value)
{
  int i = key % NBUCKET;

  // is the key already present?
  struct entry *e = 0;
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key)
      break;
  }
  if(e){
    // update the existing key.
    e->value = value;
  } else {
    // the new is new.
    insert(key, value, &table[i], table[i]);
  }

}

static struct entry*
get(int key)
{
  int i = key % NBUCKET;


  struct entry *e = 0;
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key) break;
  }

  return e;
}
```



```c
static void *
put_thread(void *xa)
{
  int n = (int) (long) xa; // thread number
  int b = NKEYS/nthread;

  for (int i = 0; i < b; i++) {
    put(keys[b*n + i], n);
  }

  return NULL;
}

static void *
get_thread(void *xa)
{
  int n = (int) (long) xa; // thread number
  int missing = 0;

  for (int i = 0; i < NKEYS; i++) {
    struct entry *e = get(keys[i]);
    if (e == 0) missing++;
  }
  printf("%d: %d keys missing\n", n, missing);
  return NULL;
}
```



```c
int
main(int argc, char *argv[])
{
  pthread_t *tha;
  void *value;
  double t1, t0;


  if (argc < 2) {
    fprintf(stderr, "Usage: %s nthreads\n", argv[0]);
    exit(-1);
  }
  nthread = atoi(argv[1]);
  tha = malloc(sizeof(pthread_t) * nthread);
  srandom(0);
  assert(NKEYS % nthread == 0);
  for (int i = 0; i < NKEYS; i++) {
    keys[i] = random();
  }

  //
  // first the puts
  //
  t0 = now();
  for(int i = 0; i < nthread; i++) {
    assert(pthread_create(&tha[i], NULL, put_thread, (void *) (long) i) == 0);
  }
  for(int i = 0; i < nthread; i++) {
    assert(pthread_join(tha[i], &value) == 0);
  }
  t1 = now();

  printf("%d puts, %.3f seconds, %.0f puts/second\n",
         NKEYS, t1 - t0, NKEYS / (t1 - t0));

  //
  // now the gets
  //
  t0 = now();
  for(int i = 0; i < nthread; i++) {
    assert(pthread_create(&tha[i], NULL, get_thread, (void *) (long) i) == 0);
  }
  for(int i = 0; i < nthread; i++) {
    assert(pthread_join(tha[i], &value) == 0);
  }
  t1 = now();

  printf("%d gets, %.3f seconds, %.0f gets/second\n",
         NKEYS*nthread, t1 - t0, (NKEYS*nthread) / (t1 - t0));
}
```



![image-20220705150714620](../../../../../../.config/Typora/typora-user-images/image-20220705150714620.png)





### 多线程哈希表

在`main`中初始化锁给整个哈希表一个大锁，并在`put`和`get`中加锁来保证多线程哈希表的正确性

```c
pthread_mutex_t lock;

static 
void put(int key, int value)
{
  int i = key % NBUCKET;

  // is the key already present?
  struct entry *e = 0;
  
  pthread_mutex_lock(&lock);  // 获取锁
    
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key)
      break;
  }
  if(e){
    // update the existing key.
    e->value = value;
  } else {
    // the new is new.
    insert(key, value, &table[i], table[i]);
  }
  pthread_mutex_unlock(&lock);    //释放锁
}

static struct entry*
get(int key)
{
  int i = key % NBUCKET;

  pthread_mutex_lock(&lock);  // 获取锁

  struct entry *e = 0;
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key) break;
  }
  pthread_mutex_unlock(&lock); //释放锁

  return e;
}

int
main(int argc, char *argv[])
{
  pthread_t *tha;
  void *value;
  double t1, t0;


  if (argc < 2) {
    fprintf(stderr, "Usage: %s nthreads\n", argv[0]);
    exit(-1);
  }
  nthread = atoi(argv[1]);
  tha = malloc(sizeof(pthread_t) * nthread);
  srandom(0);
  assert(NKEYS % nthread == 0);
  for (int i = 0; i < NKEYS; i++) {
    keys[i] = random();
  }

  pthread_mutex_init(&lock,NULL);   //初始化锁
  //
  // first the puts
  // 省略

  //
  // now the gets
  // 省略
}

```

![image-20220705164033748](../../../../../../.config/Typora/typora-user-images/image-20220705164033748.png)

注意到哈希表多线程正确性满足了，但是性能却低于单线程，考虑减小锁的粒度来优化性能

**性能**

将`main`中整个哈希表一把锁改为哈希表中每个桶一把锁

```c
pthread_mutex_t lock[NBUCKET];	//每个桶一把锁

static 
void put(int key, int value)
{
  int i = key % NBUCKET;

  // is the key already present?
  struct entry *e = 0;

  pthread_mutex_lock(&lock[i]);  // 获取锁

  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key)
      break;
  }
  if(e){
    // update the existing key.
    e->value = value;
  } else {
    // the new is new.
    insert(key, value, &table[i], table[i]);
  }
  pthread_mutex_unlock(&lock[i]);    //释放锁
}

static struct entry*
get(int key)
{
  int i = key % NBUCKET;

  pthread_mutex_lock(&lock[i]);  // 获取锁

  struct entry *e = 0;
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key) break;
  }
  pthread_mutex_unlock(&lock[i]); //释放锁

  return e;
}

int
main(int argc, char *argv[])
{
  pthread_t *tha;
  void *value;
  double t1, t0;


  if (argc < 2) {
    fprintf(stderr, "Usage: %s nthreads\n", argv[0]);
    exit(-1);
  }
  nthread = atoi(argv[1]);
  tha = malloc(sizeof(pthread_t) * nthread);
  srandom(0);
  assert(NKEYS % nthread == 0);
  for (int i = 0; i < NKEYS; i++) {
    keys[i] = random();
  }

    for (int i = 0; i < NBUCKET; ++i) {
        pthread_mutex_init(&lock[i],NULL);   //每个桶一把锁
    }

  //
  // first the puts
  // 省略

  //
  // now the gets
  // 省略
}

```

现在性能有了显著提升

![image-20220705164728500](../../../../../../.config/Typora/typora-user-images/image-20220705164728500.png)

再次分析 `get`函数，发现实际上并没有对哈希表做修改，多线程同时读是可以的，所以我们可以直接把`get`的锁给去掉

```c
static struct entry*
get(int key)
{
  int i = key % NBUCKET;

//  pthread_mutex_lock(&lock[i]);  // 获取锁

  struct entry *e = 0;
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key) break;
  }
//  pthread_mutex_unlock(&lock[i]); //释放锁

  return e;
}
```

![image-20220705165244642](../../../../../../.config/Typora/typora-user-images/image-20220705165244642.png)







## 问题

Why are there missing keys with 2 threads, but not with 1 thread? Identify a sequence of events with 2 threads that can lead to a key being missing. Submit your sequence with a short explanation in answers-thread.txt

多线程进行`put`写操作的时候存在竞态条件，造成错误出现

```c
static 
void put(int key, int value)
{
  int i = key % NBUCKET;
  //////////////////////////////////////// POINT A		
  // is the key already present?
  struct entry *e = 0;
  for (e = table[i]; e != 0; e = e->next) {
    if (e->key == key)
      break;
  }
  //////////////////////////////////////// POINT B	
  if(e){
    // update the existing key.
    e->value = value;
  } else {
    // the new is new.
    insert(key, value, &table[i], table[i]);
  }
  //////////////////////////////////////// POINT C	
}
```

例子

```c
thread1:POINT A ready to put key1 into bucket1
thread1:POINT B not exist key1

----------schedule------------

thread2:POINT A ready to put key2 into bucket1
thread2:POINT B not exist key2
thread2:POINT C insert key2 into bucket1[x]

----------schedule------------

thread2:POINT C insert key1 into bucket1[x]  <--------- key2 missing
```



### 扩展

OSTEP