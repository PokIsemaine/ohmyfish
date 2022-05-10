# Lab: Xv6 and Unix utilities

## find ([moderate](https://pdos.csail.mit.edu/6.S081/2021/labs/guidance.html))

Write a simple version of the UNIX find program: find all the 
files in a directory tree with a specific name. Your solution should be in the file `user/find.c`.

Some hints:

- Look at user/ls.c to see how to read directories.
- Use recursion to allow find to descend into sub-directories.
- Don't recurse into "." and "..".
- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.
- You'll need to use C strings. Have a look at K&R (the C book), for example Section 5.5.
- Note that == does not compare strings like in Python. Use strcmp() instead.
- Add the program to `UPROGS` in Makefile.

Your solution is correct if produces the following output (when the file system contains the files `b` and `a/b`):

```shell
    $ make qemu
    ...
    init: starting sh
    $ echo > b
    $ mkdir a
    $ echo > a/b
    $ find . b
    ./b
    ./a/b
    $ 
```

## 过程

照着`ls.c`改一下就可以啦

```c
#include "../kernel/types.h"
#include "../kernel/stat.h"
#include "../user/user.h"
#include "../kernel/fs.h"

char*
fmtname(char *path)
{
//   static char buf[DIRSIZ+1];
  char *p;

  // Find first character after last slash.
  for(p=path+strlen(path); p >= path && *p != '/'; p--)
    ;
  p++;

  return p;
}

void
find(char *path,char* expression)
{
  char buf[512], *p;
  int fd;
  struct dirent de;
  struct stat st;

  if((fd = open(path, 0)) < 0){
    fprintf(2, "find: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){
    fprintf(2, "find: cannot stat %s\n", path);
    close(fd);
    return;
  }

  switch(st.type){
  case T_FILE:
	if(0 == strcmp(fmtname(path),expression)) {
		printf("%s\n",path);
	}
    break;

  case T_DIR:
	if(path[strlen(path)-1]!='.'||strlen(path) == 1) {
		if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
			printf("find: path too long\n");
			break;
		}
		strcpy(buf, path);
		p = buf+strlen(buf);
		if(*(p-1) != '/')*p++ = '/'; //if 特殊处理一下pathname/的情况
		while(read(fd, &de, sizeof(de)) == sizeof(de)){
			if(de.inum == 0)
				continue;
			memmove(p, de.name, DIRSIZ);
			p[DIRSIZ] = 0;
			if(stat(buf, &st) < 0){
				printf("find: cannot stat %s\n", buf);
				continue;
			}
			
			find(buf,expression);
		}
	
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])
{
  int i;

  if(argc < 3){
    fprintf(2,"find [pathname] [expression]\n");
    exit(1);
  }
  for(i=2; i<argc; i++)
    find(argv[1],argv[i]);
  exit(0);
}

```

