# I/O: input & output,是一切实现的基础

1. stdio 标准IO（优先使用）
2. sysio 系统调用IO（文件IO）

——————

## stido 标准IO

先补充main函数：

//参数形式: int main(int argv,char** argv)   argc 形参表示传入参数的个数，包括应用程序自身路径和程序名

举个例子：

```
./hello 112233
```

那么此时参数个数为 2
argv[0]等于"./hello"
argv[1]等于"112233"



stdio: FILE类型贯穿始终

```
//FILE结构体的形式
//stdio.h
typedef struct _iobuf {
    char*  _ptr;        //文件输入的下一个位置
    int    _cnt;        //当前缓冲区的相对位置
    char*  _base;       //文件初始位置
    int    _flag;       //文件标志
    int    _file;       //文件有效性
    int    _charbuf;    //缓冲区是否可读取
    int    _bufsiz;     //缓冲区字节数
    char*  _tmpfname;   //临时文件名
} FILE;

```

*fopen() , fclose() :文件的读写

（内存申请在堆区，fopen()->malloc,fclose()->free）

// 参数形式：

FILE * fopen(const char *path,const  char *mode)  , int fclose(FILE *stream)

fopen()对文件操作的六种形式：“r”，“r+”，“w”，“w+”，“a”，“a+”

分别表示 只读，读写，有则清除开写|无则创建开写，读写，续写，读续写。

fclose():关闭成功则返回0，否则返回EOF。



*fgetc(),fputc()fgets(),fputs(),fread(),fwrite():字符串的读写

//参数形式：

- fgetc(char *s, int size, FILE *stream),fputc(const char *s, FILE *stream)
- fgets(char *s, int size, FILE *stream),fputs(const char *s, FILE *stream)
- fread(void *ptr, size_t size, size_t nmemb, FILE *stream),fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream)

  对于fread和fwrite，**size_t size,size_t nmemb分别表示一次读取或写入的对象大小和对象个数**

​	当打开文件时，会将读写偏移量设置为指向文件开始位置处，以后每次调用 read()、       	write()将自动对其进行调整，以指向已读或已写数据后的下一字节

*printf(),scanf(),fprintf()

//参数形式：int fprintf( FILE *stream, const char *format, ... )

fprintf()函数根据指定的format(格式)发送信息(参数)到由 stream(流)指定的文件.因此      fprintf()可以使得信息输出到指定的文件，比如：
    

```
 char name[20] = "Mary";
 FILE *out;
 out = fopen( "output.txt", "w" );
 if( out != NULL )
 fprintf( out, "Hello %s\n", name );
```

 fprintf()和printf()一样工作. 
 printf是打印输出到屏幕，fprintf是打印输出到文件。 
 fprintf()的返回值是输出的字符数,发生错误时返回一个负值. 



*fseek(),ftell(),rewind():文件位置指针的操作

//参数形式：fseek(FILE*stream,long offset,int whence),ftell(FILE *stream),rewind(FILE *stream);最多定位2G的文件大小

-  fseek()函数简单的理解，**功能就是用来设置打开文件中光标的位置**。fseek()函数有三个参数，第一个参数是需要操作的文件指针，第二个参数是光标的偏移量，第三个参数为确定起点模式，也就是设置在文件中光标的起点位置。

-  ftell()函数**主要用来读取，当前光标在文件中的位置。**它的参数只有一个，就是需要读取的文件指针，返回值的类型是long，数字的大小代表当前光标距离文件起始处的字节数。

- rewind ()函数**用于将文件内部的位置指针重新指向一个流（数据流或者文件）的起始位置。**这里需要注意的是，这里的“指针”表示的不是文件指针，而是文件内部的位置指针。即随着对文件的读写，文件的位置指针（指向当前读写字节）向后移动。而文件指针指向整个文件，如果不重新赋值，文件指针不会发生改变。



*fflush();

//参数形式：int fflush(FILE *stream);

- fflush()是一个 C 标准库函数，用于刷新流的缓冲区，以确保缓冲区中的数据被写入到文件中。

- 如果没有调用 fflush，则可能会出现数据没有被写入文件的情况。这是因为标准库通常使用缓冲区来提高 I/O 性能。**当我们向文件写入数据时，数据首先被写入到缓冲区中，然后在适当的时候才被写入到文件中。**如果我们没有调用 fflush，则缓冲区中的数据可能会一直保留，直到程序结束或缓冲区满了才被写入到文件中。这可能会导致数据丢失或不一致的情况。

  缓冲区的作用：大多数情况下是好事，合并系统调用。

  - 行缓冲:换行的时候刷新，满了的时候刷新，强制刷新（标准输出是这样的，因为是终端设备）
  - 全缓冲：满了的时候刷新，强制刷新（默认，只要不是终端设备）
  - 无缓冲：如stderr，需要立即输出的内容

​                   

*getline()

//参数形式：ssize_t getline(char **lineptr,size_t *n,FILE *stream);

返回值是成功读到的字符个数，包括分隔符（空格，回车），但不包含最后的'\0'。

返回-1则表示读取失败



*临时文件：1.如何不冲突 2.及时销毁

创建临时文件的两个函数：tmpnam(),tmpfile()

//参数形式：char *tmpnam(char *s), FILE *tmpfile(void)

- tmpnam():功能:系统自己创建一个文件，文件名系统自己给定。并且返回这个文件的路径名指针。参数ptr要求是一个指向一个长度至少是L_tmpnam个字符的数组，或者ptr为NULL,此时，系统在一个静态区中存放新建文件的路径名。该静态区是公用的，下一次ptr为NULL，调用该函数时，新的文件路径名仍存在这个静态区中。
- tmpfile():函数创建一个临时二进制文件(w+b),在关闭该文件或程序结束时,该文件自动删除。如果发生错误，tmpfile会返回一个空指针，并且设置errno变量。

**Sum：一个程序调用tmpnam生成一个唯一文件名的临时文件。如果我们要使用这个临时文件，我们可以立即打开，从而来减小另一个程序会使用同一个文件名打开这个文件的风险。tmpfile调用会同时创建并打开一个临时文件，从而避免了这种风险。**



## sysio 系统调用IO /文件IO

###文件描述符（fd 即file descriptor）是在文件IO中贯穿是始终的类型



####文件描述符的概念：整形数（即open() **[ps:不是fopen()]**函数返回的整数值，一般从3开始，最大为1024，其中整数0，1，2分别代表stdin，stdout，stderr三个文件），数组下标，文件描述符优先使用当前可用范围内最小的的数

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a8b7728184bcfa9db9e6834df5f05a9.png)

⚫ 一个进程内多次 open 打开同一个文件，那么会得到多个不同的文件描述符 fd
⚫ 一个进程内多次 open 打开同一个文件，在内存中并不会存在多份动态文件。
⚫ 一个进程内多次 open 打开同一个文件，不同文件描述符所对应的读写位置偏移量是相互独立的。



####文件IO操作：open，close，read，write，lseek（标准IO操作依附于文件IO操作）

1.open()://参数形式有两种：

 (1) int open (const char *pathname,int flags);flags表示权限信息，返回值是一个新的文件描述符，如果失败则返回-1。其中，flags是由必需文件访问模式和可选模式一起构成的(通过按位或`|`)：**橙色大写英文都是宏**

![img](https://img-blog.csdnimg.cn/direct/d6037ad355104c719f1106ab4da2209d.png)

(2) int open(const char *pathname, int flags, mode_t mode);

在第一种调用方式上，加上了第三个参数mode，主要是搭配O_CREAT使用，这个参数规定了用户、同组用户和其他人对文件的文件操作权限。只列出部分：

![img](https://img-blog.csdnimg.cn/direct/8a0fe75d0e1b47bf9f35d222126f5c72.png)

  2.close() //参数形式：int close(int fd);

- 返回 0 表示成功，或者 -1 表示有错误发生，并设值errno；

  3.read() //参数形式：ssize_t read(int fd, void *buf, size_t count);

- 从与文件描述符fd相关联的文件中读取前count字节的内容，并且写入到数据区buf中
- read系统调用返回的是实际读入的字节数，发生错误返回`-1`

  4.write() //参数形式：ssize_t write(int fd, const void *buf, size_t count);

- 把缓存区buf中的前count字节写入到与文件描述符fd有关的文件中
- write系统调用返回的是实际写入到文件中的字节数，发生错误返回-1，注意返回0不是发生错误，而是写入的字节数为0

  5.lseek() //参数形式：off_t lseek(int fd, off_t offset, int whence);

- lseek设置文件位置为给定的偏移 offset，参数 offset 意味着从给定的 whence 位置查找的字节数。

`whence`取值：

| 字段       | 含义         |
| ---------- | ------------ |
| `SEEK_SET` | 文件开头     |
| `SEEK_END` | 文件末尾     |
| `SEEK_CUR` | 文件当前位置 |



####文件IO与标准IO的区别（有无缓存区）

文件I/O：文件I/O又称为**无缓冲IO，**低级磁盘I/O，遵循POSIX相关标准。任何兼容POSIX标准的操作系统上都支持文件I/O。

标准I/O：标准I/O是ANSI C建立的一个标准I/O模型，又称为高级磁盘I/O，是一个标准函数包和stdio.h头文件中的定义，具有一定的可移植性。标准I/O库处理很多细节。例如缓存分配，以优化长度执行I/O等。**标准的I/O提供了三种类型的缓存**（行缓存、全缓存和无缓存）。
缓存是内存上的某一块区域。**缓存的一个作用是合并系统调用，即将多次的标准IO操作合并为一个系统调用操作。**

**文件IO不使用缓存**，每次调用读写函数时，从用户态切换到内核态，对磁盘上的实际文件进行读写操作，因此响应速度快，坏处是频繁的系统调用会增加系统开销（用户态和内核态来回切换），例如调用write写入一个字符时，磁盘上的文件中就多了一个字符。

**标准IO使用缓存**，未刷新缓冲前的多次读写时，实际上操作的是内存上的缓冲区，与磁盘上的实际文件无关，直到刷新缓冲时，才调用一次文件IO，从用户态切换到内核态，对磁盘上的实际文件进行操作。因此标准IO吞吐量大，相应的响应时间比文件IO长。但是差别不大，建议使用标准IO来操作文件。

**文件io（系统调用io）响应速度快，它在输出端没有缓冲区，有数据来了，就直接处理；标准io吞吐量更大，因为它有缓冲区，所以是等满足一定条件了，再把缓冲区中的数据一起发送出去。**
**从实际的用户体验来说，吞吐量大会感觉更快。所以在文件io和标准io都能够调用的时候，选择标准io是更好的**。

两种IO可以相互转化：
fileno：返回结构体FILE的成员_file，即文件描述符。标准IO->文件IO

```
int fileno(FILE *stream);
```


fdopen：通过文件描述符fd，返回FILE结构体。文件IO->标准IO

```
FILE *fdopen(int fd, const char *mode);
```

举个例子：

```
#include<stdio.h>
#include<stdlib.h>
#include <unistd.h>
#define BUFFSIZE 1024

int main(){
    putchar('a');
    write(1,"b",1);

    putchar('a');
    write(1,"b",1);

    putchar('a');
    write(1,"b",1);

    exit(0);
}
/*输出是 bbbaaa，可见很好的印证了上面所说的标准IO和系统IO的区别，write是系统IO，所以立即输出；而putchar是标准IO，有缓冲区，并未立即输出。*/
```



####IO的效率问题

BUFSIZE对IO效率的影响

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/ede867d37fa1432d83bf29bad95fb162.png)

图中用户CPU时间（user）是程序在用户态下的执行时间；系统CPU时间（sys）是程序在内核态下的执行时间；时钟时间（real）是两个时间的总和；

BUFSIZE受栈大小的影响；此测试所用的文件系统是Linux ext4文件系统，其磁盘块长度为4096字节。这也证明了图中系统 CPU 时间的几个最小值差不多出现在BUFFSIZE 为4096 及以后的位置，继续增加缓冲区长度对此时间几乎没有影响。



####文件共享

文件共享指的是同一个文件（譬如磁盘上的同一个文件，对应同一个 inode）被多个独立的读写体同时进行 IO 操作
读写体理解成文件描述符

可以有以下三种方式实现：

1.同一个进程中多次调用 open 函数打开同一个文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3c34c1a98d2545f68253d0fabf12bcaa.png)

2.不同进程中分别使用 open 函数打开同一个文件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/55d131f6dcc2e4db8a7463bcc5942d93.png)

3.同一个进程中通过 dup（[dup2](https://so.csdn.net/so/search?q=dup2&spm=1001.2101.3001.7020)）函数对文件描述符进行复制

![img](https://i-blog.csdnimg.cn/blog_migrate/67d763ba7767c2b63d88ec2af2a2e938.png)



####原子操作

文件中的重定向：dup，dup2

同步：常用函数有sync, fsync, fdatasync

fcntl(),ioctl()

/dev/fd/目录：

















