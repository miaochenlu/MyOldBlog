---
title: Xv6 and Unix utilities [OS lab1]
date: 2020-12-03 12:43:13
index_img: /img/xv6lab.jpg
tags: [Operating System, xv6]
categories: Operating System
---



<a href="https://pdos.csail.mit.edu/6.828/2020/labs/util.html">MIT Lab1: Xv6 and Unix utilities</a>



# Find Some References

[MIT 6.828 - 1. Lab 01: Xv6 and Unix utilities](https://www.cnblogs.com/nlp-in-shell/p/11909472.html)

[MIT-6.s081-OS lab util Unix utilities_RedemptionC的博客-CSDN博客](https://blog.csdn.net/RedemptionC/article/details/106484363?utm_medium=distribute.pc_relevant.none-task-blog-OPENSEARCH-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-2.channel_param)

[MIT6.828实验1 -- Lab Utilities](https://cloud.tencent.com/developer/article/1639599)

[zhayujie/xv6-riscv-fall19](https://github.com/zhayujie/xv6-riscv-fall19/tree/util/user)

[MIT 6.828 Lab 1: Xv6 and Unix utilities * The Real World](https://abcdlsj.github.io/post/mit-6.828-lab1xv6-and-unix-utilities/)

[6.S081-LAB1 Xv6 and Unix utilities](https://qinstaunch.github.io/2020/07/04/6-S081-LAB1-Xv6-and-Unix-utilities/)



# Boot xv6 (easy)

```bash
$ git clone git://g.csail.mit.edu/xv6-labs-2020
Cloning into 'xv6-labs-2020'...
...
$ cd xv6-labs-2020
$ git checkout util
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'
```

Build and run xv6:

```bash
$ make qemu
riscv64-unknown-elf-gcc    -c -o kernel/entry.o kernel/entry.S
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/start.o kernel/start.c
...  
riscv64-unknown-elf-ld -z max-page-size=4096 -N -e main -Ttext 0 -o user/_zombie user/zombie.o user/ulib.o user/usys.o user/printf.o user/umalloc.o
riscv64-unknown-elf-objdump -S user/_zombie > user/zombie.asm
riscv64-unknown-elf-objdump -t user/_zombie | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > user/zombie.sym
mkfs/mkfs fs.img README  user/xargstest.sh user/_cat user/_echo user/_forktest user/_grep user/_init user/_kill user/_ln user/_ls user/_mkdir user/_rm user/_sh user/_stressfs user/_usertests user/_grind user/_wc user/_zombie 
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
balloc: first 591 blocks have been allocated
balloc: write bitmap block at sector 45
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ 

```

If you type ls at the prompt, you should see output similar to the following:

```bash
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2059
xargstest.sh   2 3 93
cat            2 4 24256
echo           2 5 23080
forktest       2 6 13272
grep           2 7 27560
init           2 8 23816
kill           2 9 23024
ln             2 10 22880
ls             2 11 26448
mkdir          2 12 23176
rm             2 13 23160
sh             2 14 41976
stressfs       2 15 24016
usertests      2 16 148456
grind          2 17 38144
wc             2 18 25344
zombie         2 19 22408
console        3 20 0

```

These are the files that mkfs includes in the initial file system; most are programs you can run. You just ran one of them: ls.

xv6 has no ps command, but, if you type Ctrl-p, the kernel will print information about each process. If you try it now, you'll see two lines: one for init, and one for sh.

To quit qemu type: Ctrl-a x.

# sleep 

{% note primary %}

Implement the UNIX program `sleep` for xv6; your sleep should pause for a user-specified number of ticks. A tick is a notion of time defined by the xv6 kernel, namely the time between two interrupts from the timer chip. Your solution should be in the file `user/sleep.c`.

{% endnote %}

### Some hints:

{% note success%}

* Before you start coding, read Chapter 1 of the [xv6 book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf). 

{%  endnote %}



{% note success%}

* Look at some of the other programs in  (e.g. `user/echo.c`, `user/grep.c`,  `user/rm.c` ) to see how you can obtain the command-line arguments passed to a program.

{%  endnote %}

了解command-line arguments的传递。这是通过`argc` ，`argv`完成的。`argc`记录了参数的个数，`argv`记录了每个参数的内容

看 `user/rm.c`这个例子

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  int i;

  if(argc < 2){
    fprintf(2, "Usage: rm files...\n");
    exit(1);
  }

  for(i = 1; i < argc; i++){
    if(unlink(argv[i]) < 0){
      fprintf(2, "rm: %s failed to delete\n", argv[i]);
      break;
    }
  }

  exit(0);
}
```

`rm`命令的使用方式是`rm filename...`。`argv`这个参数数组中第一个参数是`rm`这个命令，后面每一个filename是一个单独的参数。因此，如果`argc < 2`说明只有`rm`这一个参数，不知道要`rm`哪个文件，这个时候应该报错, 这里通过`fprintf`向stderr输出错误，然后`exit(1)`

参数个数合理的话，就对后面对每一个文件调用`ulink` system call来删除文件

{% note success %}

* If the user forgets to pass an argument, sleep should print an error message.

{% endnote %}

`sleep`这个命令需要接受一个tick参数。参考`rm.c`的分析，应该写成

```c
if(argc != 2) {
	fprintf(2, "...");
	exit(1);
}
```

注意: `2` 是标准错误输出，另外 `0`是标准输入, `1` 是标准输出

{% note success %}

* 是The command-line argument is passed as a string; you can convert it to an integer using `atoi` (see `user/ulib.c`).

{% endnote %}

`sleep`接收一个int类型的参数，而`argv`中的参数是char*类型，因此需要将其转化为int。这是通过调用`atoi`函数来实现的

`atoi(char* s)`

```c
int
atoi(const char *s)
{
  int n;

  n = 0;
  while('0' <= *s && *s <= '9')
    n = n*10 + *s++ - '0';
  return n;
}
```

{% note success %}

* Use the system call `sleep`.

{% endnote %}



{%note success%}

See `kernel/sysproc.c` for the xv6 kernel code that implements the  `sleep` system call (look for `sys_sleep`),  `user/user.h` for the C definition of  callable from a user program, and `user/usys.S` for the assembler code that jumps from user code into the kernel for `sleep.`

{% endnote %}

我们实现的`user/sleep.c`属于user space的函数，他通过调用system call `sleep`实现他的功能。在下一个lab会知道如何从user space跳转到kernel space

`kernel/sys_sleep`

```c
uint64
sys_sleep(void)
{
  int n;
  uint ticks0;

  if(argint(0, &n) < 0)
    return -1;
  acquire(&tickslock);
  ticks0 = ticks;
  while(ticks - ticks0 < n){
    if(myproc()->killed){
      release(&tickslock);
      return -1;
    }
    sleep(&ticks, &tickslock);
  }
  release(&tickslock);
  return 0;
}
```

`user/user.h` 

```c
int sleep(int);
```

`user/usys.S`

```assembly
.global sleep
sleep:
 li a7, SYS_sleep
 ecall
 ret
```

容易发现一个问题, 在用户模式下调用 `sleep` 是有参数的，但是kernel下的调用没有参数 `uint64 sys_sleep(void)`，那么参数是如何传递的呢？

Passing arguments from user-level functions to kernel-level functions cannot be done in XV6. XV6 has its own built-in functions for passing arguments into a kernel function. For instance, to pass in an integer, the `argint()` function is called. 

```c
argint(0, &pid);
```

... to get the first argument which is the Process ID, and:

```c
argint(1, &pty);
```

{% note success %}

* Make sure `main` calls  `exit()` in order to exit your program.

{% endnote %}

最后加一条`exit(0)` .



{% note success %}

* Add your  `sleep` program to `UPROGS` in Makefile; once you've done that,  `make qemu` will compile your program and you'll be able to run it from the xv6 shell.

{% endnote %}

Run the program from the xv6 shell:

```bash
$ make qemu
...
init: starting sh
$ sleep 10
(nothing happens for a little while)
$
    
```

Your solution is correct if your program pauses when run as shown above. Run make grade to see if you indeed pass the sleep tests.

Note that make grade runs all tests, including the ones for the assignments below. If you want to run the grade tests for one assignment, type:

```bash
$ ./grade-lab-util sleep
```

This will run the grade tests that match "sleep". Or, you can type:

```bash
$ make GRADEFLAGS=sleep grade
```

which does the same.

## Final Code

<span class="label label-danger">user/sleep.c</span>

```c
#include "kernel/types.h"
#include "user/user.h"
int main(int argc, char* argv[]) {
	if(argc != 2) {
		fprintf(2, "Usage: sleep [ticks]\n");
		exit(1);
	}
	if(sleep(atoi(argv[1]) < 0)) {
		fprintf(2, "Failed to sleep %s ticks.\n", argv[1]);
		exit(1);
	}
	exit(0);
}
```



<span class="label label-danger">Makefile</span>

```makefile
...
UPROGS=\
	...
	$U/_sleep\
...
```



# pingpong

{% note primary %}

Write a program that uses UNIX system calls to "ping-pong" a byte between two processes over a pair of pipes, one for each direction. The parent should send a byte to the child; the child should print `<pid>: received ping`, where pid is its process ID, write the byte on the pipe to the parent, and exit; the parent should read the byte from the child, print `<pid>: received pong`, and exit. Your solution should be in the file `user/pingpong.c`.

{% endnote %}

## Some hints:

{% note success%}

- Use  `pipe` to create a pipe.

{% endnote %}



{% note success %}

- Use  `fork` to create a child.

{% endnote %}



{%note success%}

- Use  `read` to read from the pipe, and  `write` to write to the pipe.

{% endnote %}



{% note success %}

- Use  `getpid` to find the process ID of the calling process.

{% endnote %}



{% note success%}

- Add the program to  `UPROGS` in Makefile.

{% endnote %}



{% note success%}

- User programs on xv6 have a limited set of library functions available to them.

{% endnote %}

Run the program from the xv6 shell and it should produce the following output:

```bash
    $ make qemu
    ...
    init: starting sh
    $ pingpong
    4: received ping
    3: received pong
    $
```

Your solution is correct if your program exchanges a byte between two processes and produces output as shown above.

## Final Code

关于pipe的内容见之前的[这篇文章](https://miaochenlu.github.io/2020/12/01/pipe/)

这里就是创建两个pipe, 一个pipe的方向是parent-->child, 另一个pipe的方向是child-->parent

<span class="label label-danger">user/pingpong.c</span>

```c
#include "kernel/types.h"
#include "user/user.h"

int 
main(int argc, char* argv[]) {
    int parent_fd[2], child_fd[2];
    char buf[10];

    if(pipe(parent_fd) < 0 || pipe(child_fd) < 0) {
        fprintf(2, "Error: Can't create pipe!\n");
        exit(1);
    }
    int pid = fork();

    if(pid == 0) {  //children process
        close(parent_fd[1]); //close write
        close(child_fd[0]);
        read(parent_fd[0], buf, 1);
        if(buf[0] == 'i') {
            printf("%d: received ping\n", getpid());
        }
        write(child_fd[1], "o", 1);
        close(parent_fd[0]);
        close(child_fd[1]);
    } else {
        close(parent_fd[0]);
        close(child_fd[1]);
        write(parent_fd[1], "i", 1);
        read(child_fd[0], buf, 1);
        if(buf[0] == 'o') {
            printf("%d: received pong\n", getpid());
        }
        close(parent_fd[1]);
        close(child_fd[0]);
    }
    exit(0);

}
```

<span class="label label-danger">Makefile</span>

```makefile
...
UPROGS=\
	...
	$U/_sleep\
	$U/_pingpong\
...
```

# Primes

{% note primary%}

Write a concurrent version of prime sieve using pipes. This idea is due to Doug McIlroy, inventor of Unix pipes. The picture halfway down [this page](http://swtch.com/~rsc/thread/) and the surrounding text explain how to do it. Your solution should be in the file `user/primes.c`.

<br>

Your goal is to use `pipe` and `fork` to set up the pipeline. The first process feeds the numbers 2 through 35 into the pipeline. For each prime number, you will arrange to create one process that reads from its left neighbor over a pipe and writes to its right neighbor over another pipe. Since xv6 has limited number of file descriptors and processes, the first process can stop at 35.

{% endnote %}

[【译文】Bell Labs and CSP Threads, Russ Cox.](https://qinstaunch.github.io/2020/07/04/%E3%80%90%E8%AF%91%E6%96%87%E3%80%91Bell-Labs-and-CSP-Threads-Russ-Cox/#more)



<center><video id="video" controls="" preload="none" width="400"><source id="mp4" src="https://miaochenlu.github.io/2020/12/03/xv6-lab1/prime3.mov"> 
      <p>Your user agent does not support the HTML5 Video element.</p>
</video></center>

思路: 

以pipeline的形式来处理，和shell pipeline非常相似。

对每个进程来说，他得到的第一个数是素数。他的任务是筛选出不能整除这个素数的数，交给下一个进程处理。



<span class="label label-danger">pseudocode</span>

```c
void primes() {
  p = read from left         // 从左边接收到的第一个数一定是素数
  if (fork() == 0): 
    primes()                 // 子进程，进入递归
  else: 
    loop: 
      n = read from left     // 父进程，循环接收左边的输入  
      if (p % n != 0): 
        write n to right     // 不能被p整除则向右输出   
}
```

<span class="label label-danger">user/primes.c</span>

```cpp
#include "kernel/types.h"
#include "user/user.h"

void primes() {
    int p[2];
    int num;

    if(read(0, (void*)&num, sizeof(num)) <= 0)
        return;
    
    printf("prime %d\n", num);
    if(pipe(p) < 0) {
        fprintf(2, "Error: cannot create pipe");
        exit(1);
    }
    int pid = fork();
    if(pid == 0) {
        close(0);
        dup(p[0]);
        close(p[0]);
        close(p[1]);
        primes();
    } else {
        close(1);
        dup(p[1]);
        close(p[0]);
        close(p[1]);
        int tmpnum = 0;
        while(read(0, (void*)&tmpnum, sizeof(tmpnum))) {
            if(tmpnum % num != 0) {
                write(1, (void*)&tmpnum, sizeof(tmpnum));
            }
        }
        close(1);
        wait(&pid);
    }

    
}

int main(int argc, char* argv[]) {
    int p[2];
    if(pipe(p) < 0) {
        fprintf(2, "Error: cannot create pipe");
        exit(1);
    }

    int pid = fork();
    if(pid == 0) {
        close(0);
        dup(p[0]);
        close(p[0]);
        close(p[1]);
        primes();
    } else {
        close(1);
        dup(p[1]);
        close(p[0]);
        close(p[1]);
        for(int i = 2; i <= 35; i++) {
            write(1, (void*)&i, sizeof(i));
        }
        close(1);
        wait(&pid);
    }
    exit(0);
}
```

<span class="label label-danger">user/pingpong.c</span>

抽象出了一个`redirect`函数简化代码

```cpp
#include "kernel/types.h"
#include "user/user.h"

void redirect(int k, int p[]) {
    close(k);
    dup(p[k]);
    close(p[0]);
    close(p[1]);
}

void primes() {
    int p[2];
    int num;

    if(read(0, (void*)&num, sizeof(num)) <= 0)
        return;
    
    printf("prime %d\n", num);
    if(pipe(p) < 0) {
        fprintf(2, "Error: cannot create pipe");
        exit(1);
    }
    int pid = fork();
    if(pid == 0) {
        redirect(0, p);
        primes();
    } else {
        redirect(1, p);
        int tmpnum = 0;
        while(read(0, (void*)&tmpnum, sizeof(tmpnum))) {
            if(tmpnum % num != 0) {
                write(1, (void*)&tmpnum, sizeof(tmpnum));
            }
        }
        close(1);
        wait(&pid);
    } 
}

int main(int argc, char* argv[]) {
    int p[2];
    if(pipe(p) < 0) {
        fprintf(2, "Error: cannot create pipe");
        exit(1);
    }

    int pid = fork();
    if(pid == 0) {
        redirect(0, p);
        primes();
    } else {
        redirect(1, p);
        for(int i = 2; i <= 35; i++) {
            write(1, (void*)&i, sizeof(i));
        }
        close(1);
        wait(&pid);
    }
    exit(0);
}
```





# find

{%note primary%}

Write a simple version of the UNIX find program: find all the files in a directory tree with a specific name. Your solution should be in the file `user/find.c`.

{%endnote%}

## Some hints:

{%note success%}

* Look at `user/ls.c` to see how to read directories.

{% endnote %}

这是`user/ls.c`的代码

```cpp
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char*
fmtname(char *path)
{
  static char buf[DIRSIZ+1];
  char *p;

  // Find first character after last slash.
  for(p=path+strlen(path); p >= path && *p != '/'; p--)
    ;
  p++;

  // Return blank-padded name.
  if(strlen(p) >= DIRSIZ)
    return p;
  memmove(buf, p, strlen(p));
  memset(buf+strlen(p), ' ', DIRSIZ-strlen(p));
  return buf;
}

void
ls(char *path)
{
  char buf[512], *p;
  int fd;
  struct dirent de;
  struct stat st;

  if((fd = open(path, 0)) < 0){
    fprintf(2, "ls: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){
    fprintf(2, "ls: cannot stat %s\n", path);
    close(fd);
    return;
  }

  switch(st.type){
  case T_FILE:
    printf("%s %d %d %l\n", fmtname(path), st.type, st.ino, st.size);
    break;

  case T_DIR:
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
      printf("ls: path too long\n");
      break;
    }
    strcpy(buf, path);
    p = buf+strlen(buf);
    *p++ = '/';
    while(read(fd, &de, sizeof(de)) == sizeof(de)){
      if(de.inum == 0)
        continue;
      memmove(p, de.name, DIRSIZ);
      p[DIRSIZ] = 0;
      if(stat(buf, &st) < 0){
        printf("ls: cannot stat %s\n", buf);
        continue;
      }
      printf("%s %d %d %d\n", fmtname(buf), st.type, st.ino, st.size);
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])
{
  int i;

  if(argc < 2){
    ls(".");
    exit(0);
  }
  for(i=1; i<argc; i++)
    ls(argv[i]);
  exit(0);

```

总结一下就是

* 由于需要获取文件的类型等信息，用`fstat`函数获取文件的stat信息

```cpp
if(fstat(fd, &st) < 0){
    fprintf(2, "ls: cannot stat %s\n", path);
    close(fd);
    return;
}
```

* 根据文件类型 `st.type`分别进行处理

  * 如果是`T_FILE`, 直接输出文件信息

  * 如果是`T_DIR`,  需要输出目录包含的文件的信息目录文件通过一个个struct direct (directory entry)来记录他所包含的文件

    建了一个buf数组来存放目录中文件的path, 他将会存储的信息是目录path+"/"+filename+'/0'

    因此，这里先对path的size做了限制，不能超过buf的大小

    然后，将directory的path拷贝到buf中，并且加入了一个'/'

    接着循环读取目录中的dirent结构到de。根据de, 获得filename, filename拷贝到buf， 输出文件信息。



{%note success%}

- Use recursion to allow find to descend into sub-directories.

{% endnote %}

{%note success%}

- Don't recurse into "." and "..".

{% endnote %}

这里如果不做处理会进入死循环

{%note success%}

- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.

{% endnote %}

{%note success%}

- You'll need to use C strings. Have a look at K&R (the C book), for example Section 5.5.

{% endnote %}

{%note success%}

- Note that `==` does not compare strings like in Python. Use strcmp() instead.

{% endnote %}

{%note success%}

- Add the program to `UPROGS` in Makefile.

{% endnote %}

## Final Code

思路: find命令的使用方式是`find dir filename`

打开directory, 获取里面每个文件的信息

* 如果是文件： 文件名字与需要查找的文件名相同，就组织成path输出

* 如果是目录，并且不是".", "..", 就递归查找

```cpp
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"


void find(char* path, char* filename) {
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if((fd = open(path, 0)) < 0) {
        fprintf(2, "find: cannot open %s\n", path);
        return;
    }

    if(fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }

    if(st.type != T_DIR) {
        fprintf(2, "find: %s is not a directory\n", path);
        close(fd);
        return;
    }
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
        printf("find: path too long\n");
        close(fd);
        return;
    }

    strcpy(buf, path);
    p = buf+strlen(buf);
    *p++ = '/';

    while(read(fd, &de, sizeof(de)) == sizeof(de)) {
        if(de.inum == 0) 
            continue;
        if(strcmp(de.name, ".") == 0 || strcmp(de.name, "..") == 0)
            continue;
        memmove(p, de.name, DIRSIZ);
        p[DIRSIZ] = 0;
        if(stat(buf, &st) < 0) {
            fprintf(2, "find: cannot stat %s\n", buf);;
            continue;
        }

        switch (st.type)
        {
        case T_FILE:
            if(strcmp(de.name, filename) == 0) {
                printf("%s\n", buf);
            }
            break;
        
        case T_DIR:
            find(buf, filename);
        }
    }
    close(fd);
}

int
main(int argc, char *argv[])
{
  if(argc < 3){
    fprintf(2, "Usage: find path filename\n");
    exit(1);
  }
  find(argv[1], argv[2]);
  exit(0);
}

```





# xargs

{% note primary %}

Write a simple version of the UNIX xargs program: read lines from the standard input and run a command for each line, supplying the line as arguments to the command. Your solution should be in the file `user/xargs.c`.

{% endnote %}



The following example illustrates xarg's behavior:

```shell
$ echo hello too | xargs echo bye
bye hello too
$
```

Note that the command here is "echo bye" and the additional arguments are "hello too", making the command "echo bye hello too", which outputs "bye hello too".

Please note that xargs on UNIX makes an optimization where it will feed more than argument to the command at a time. We don't expect you to make this optimization. To make xargs on UNIX behave the way we want it to for this lab, please run it with the -n option set to 1. For instance

```shell
$ echo "1\n2" | xargs -n 1 echo line
line 1
line 2
$
```

我感觉这个命令的意思是从标准输入读取数据作为后面命令的参数

## Some hints:

{% note success %}

- Use `fork` and `exec` to invoke the command on each line of input. Use `wait` in the parent to wait for the child to complete the command.

{% endnote %}

{% note success %}

- To read individual lines of input, read a character at a time until a newline ('\n') appears.

{% endnote %}

{% note success %}

- `kernel/param.h declares MAXARG`, which may be useful if you need to declare an argv array.

{% endnote %}



`#define MAXARG       32  // max exec arguments`

{% note success %}

- Add the program to `UPROGS` in Makefile.

{% endnote %}



{% note success %}

- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.

{% endnote %}

xargs, find, and grep combine well:

```shell
$ find . b | xargs grep hello
```

will run "grep hello" on each file named b in the directories below ".".







## Final Code

这个命令的重点在于重新组织`argv`, 将标准输入中的内容作为参数放入`argv`

```cpp
#include "kernel/types.h"
#include "kernel/param.h"
#include "user/user.h"

int main(int argc, char* argv[]) {
    char *nargs[MAXARG];
    int argCount = 0;
    for(int i = 1; i < argc; i++) {
        nargs[argCount++] = argv[i];
    }

    char tmpchar = 0;
    char buffer[1024];

    int status = 1;
    while(status) {
        int pos = 0;
        int argStartPos = 0;
        
        while(1) {
            status = read(0, &tmpchar, 1);
            if(status == 0) exit(0);
            if(tmpchar != ' ' && tmpchar != '\n') {
                buffer[pos++] = tmpchar;
            } else if(tmpchar == ' ') {
                buffer[pos++] = 0;
                nargs[argCount++] = &buffer[argStartPos];
                argStartPos = pos;
            } else if(tmpchar == '\n') {
                nargs[argCount++] = &buffer[argStartPos];
                argStartPos = pos;
                break;
            }
        }

        if(fork() == 0) {
            exec(nargs[0], nargs);
        } else {
            wait(0);
        }
    }
    exit(0);
}
```

