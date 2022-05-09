# Lab: Xv6 and Unix utilities

## xargs ([moderate](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

Write a simple version of the UNIX xargs program: read lines from the standard input and run a command for each line, supplying the line as arguments to the command. Your solution should be in the file `user/xargs.c`.

The following example illustrates xarg's behavior:

```
    $ echo hello too | xargs echo bye
    bye hello too
    $
  
```

Note that the command here is "echo bye" and the additional arguments are "hello too", making the command "echo bye hello too", which outputs "bye hello too".

Please note that xargs on UNIX makes an optimization where it will feed more than argument to the command at a time. We don't expect you to make this optimization. To make xargs on UNIX behave the way we want it to for this lab, please run it with the -n option set to 1. For instance

```
    $ echo "1\n2" | xargs -n 1 echo line
    line 1
    line 2
    $
  
```

Some hints:

- Use `fork` and `exec` to invoke the command on each line of input. Use `wait` in the parent to wait for the child to complete the command.
- To read individual lines of input, read a character at a time until a newline ('\n') appears.
- kernel/param.h declares MAXARG, which may be useful if you need to declare an argv array.
- Add the program to `UPROGS` in Makefile.
- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.

xargs, find, and grep combine well:

```
  $ find . b | xargs grep hello
  
```

will run "grep hello" on each file named b in the directories below ".".

To test your solution for xargs, run the shell script xargstest.sh. Your solution is correct if it produces the following output:

```
  $ make qemu
  ...
  init: starting sh
  $ sh < xargstest.sh
  $ $ $ $ $ $ hello
  hello
  hello
  $ $   
  
```

You may have to go back and fix bugs in your find program. The output has many `$` because the xv6 shell doesn't realize it is processing commands from a file instead of from the console, and prints a `$` for each command in the file.

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

