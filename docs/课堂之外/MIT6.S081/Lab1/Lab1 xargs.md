<h1>Lab: Xv6 and Unix utilities</h1>
<h2>xargs <script>g("moderate")</script></h2>

<div class="required">
<p>Write a simple version of the UNIX xargs program: read lines from
  the standard input and run a command for each line, supplying the line as
  arguments to the command.   Your solution
  should be in the file <tt>user/xargs.c</tt>.
</div>

  The following example illustrates xarg's
  behavior:
  <pre>
    $ <kbd>echo hello too | xargs echo bye</kbd>
    bye hello too
    $
  </pre>
  Note that the command here is "echo bye" and the additional
  arguments are "hello too", making the command "echo bye hello too",
  which outputs "bye hello too".

  <p> Please note that xargs on UNIX makes an optimization where it will feed more than argument to the command at a time. We don't expect you to make this optimization. To make xargs on UNIX behave the way we want it to for this lab, please run it with the -n option set to 1. For instance</p>
  <pre>
    $ <kbd>echo "1\n2" | xargs -n 1 echo line</kbd>
    line 1
    line 2
    $
  </pre>  

<p>Some hints:
  <ul>
    <li>Use <tt>fork</tt> and <tt>exec</tt> to invoke the
      command on each line of input.  Use <tt>wait</tt> in the parent
      to wait for the child to complete the command.
    <li>To read individual lines of input, read a character at a time
        until a newline ('\n') appears.
    <li>kernel/param.h declares MAXARG, which may be useful if you need
      to declare an argv array.
    <li>Add the program to <tt>UPROGS</tt> in Makefile.
    <li>Changes to the file system persist across runs of qemu; to get
    a clean file system run <kdb>make clean</kdb> and then <kdb>make qemu</kdb>.
  </ul>

<p>xargs, find, and grep combine well:
  <pre>
  $ <kbd>find . b | xargs grep hello</kbd>
  </pre>
  will run "grep hello" on each file named b in the directories below
  ".".

<p>To test your solution for xargs, run the shell script xargstest.sh.
Your solution is correct if it produces the following output:
  <pre>
  $ <kbd>make qemu</kbd>
  ...
  init: starting sh
  $ <kbd>sh < xargstest.sh</kbd>
  $ $ $ $ $ $ hello
  hello
  hello
  $ $   
  </pre>
You may have to go back and fix bugs in your find program.  The output has
many <tt>$</tt> because the xv6 shell doesn't realize
it is processing commands from a file instead of from the console, and
prints a <tt>$</tt> for each command in the file.


## 前置知识

### 管道

`sh.c`

```c
struct pipecmd {
  int type;
  struct cmd *left;
  struct cmd *right;
};
```



```c
  case PIPE:
    pcmd = (struct pipecmd*)cmd;
    if(pipe(p) < 0)
      panic("pipe");
    if(fork1() == 0){
      close(1);
      dup(p[1]);
      close(p[0]);
      close(p[1]);
      runcmd(pcmd->left);
    }
    if(fork1() == 0){
      close(0);
      dup(p[0]);
      close(p[0]);
      close(p[1]);
      runcmd(pcmd->right);
    }
    close(p[0]);
    close(p[1]);
    wait(0);
    wait(0);
    break;
```





### xargs



## 过程

```cpp
#include "../kernel/types.h"
#include "../kernel/stat.h"
#include "../kernel/param.h"
#include "../user/user.h"


int
main(int argc, char* argv[]) {
	if(argc < 2) {
		fprintf(2,"xargs [command] [params...]\n");
		exit(1);
	} else if(argc + 1 > MAXARG) {
		fprintf(2,"Too many arguments\n");
		exit(1);
	}

	char * params[MAXARG],buf[512];

	for(int i = 1;i < argc; ++i) {
		params[i-1] = argv[i];
	}

	params[argc] = 0;

	while(1) {
		int i = 0;
		while(1) {
			int n = read(0,&buf[i],1);
			if(n == 0 || buf[i] == '\n') break;
			++i;
		}

		if(0 == i)break;
		buf[i] = 0;
		params[argc - 1] = buf;

		int pid = fork();
		if(0 == pid) {
			exec(params[0],params);
			exit(0);
		} else if(0 < pid) {
			wait((int *) 0);
		} else {
			fprintf(2,"fork error\n");
		}
	}
	exit(0);
}
```

