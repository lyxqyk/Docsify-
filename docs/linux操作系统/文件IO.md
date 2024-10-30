文件描述符：在进程中打开的文件，一个进程启动后默认打开三个文件描述符：0标准输入 1标准输出 2标准错误

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1715579715999-631f83c5-b085-49ee-b6c7-eb1eff798cf5.png)

在Linux系统中，`open`和`close`是用于打开和关闭文件的两个重要系统调用。

## Open和Close函数
### open系统调用
`open`系统调用用于打开文件或者创建文件描述符。它的基本语法如下：

```c
int open(const char *pathname, int flags);
int open(const char *pathname, int flags)，mode_t mode;
```

+ `pathname`参数是要打开的文件的路径名。
+ `flags`参数指定了文件的打开方式和权限。常见的标志包括： （<font style="color:#DF2A3F;">用|连接</font>）
    - `O_RDONLY`：只读方式打开文件。
    - `O_WRONLY`：只写方式打开文件。
    - `O_RDWR`：读写方式打开文件。（<font style="color:#DF2A3F;">前三个必须选择一个且只允许选一个</font>）
    - `O_CREAT`：如果文件不存在，则创建文件。
    - `O_EXCL`：如果与`O_CREAT`同时使用，文件存在时返回错误。
    - `O_TRUNC`：如果文件存在且为只写或读写方式打开，将其长度截断为0。

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1715579877614-5061e196-2fc4-4e02-b1a8-9be862d9012f.png)

普通文件非阻塞是不会发生阻塞。非阻塞读不到内容返回-1.



###### 返回值
        `open`函数返回一个新的文件描述符（非负整数），用于后续对文件的读写操作。如果出错，返回值为-1，并设置`errno`为相应的错误代码。

### close系统调用
`close`系统调用用于关闭一个文件描述符。它的基本语法如下：

```c
int close(int fd);
```

+ `fd`参数是要关闭的文件描述符。

成功关闭文件时，`close`函数返回0；如果出错，返回-1，并设置`errno`为相应的错误代码。

### 用法示例
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int fd;
    char buf[1024];

    // 打开文件
    fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    // 读取文件内容
    ssize_t nread;
    nread = read(fd, buf, sizeof(buf));
    if (nread == -1) {
        perror("read");
        return 1;
    }

    // 输出文件内容
    write(STDOUT_FILENO, buf, nread);

    // 关闭文件
    if (close(fd) == -1) {
        perror("close");
        return 1;
    }

    return 0;
}
```

这个示例程序首先使用`open`系统调用打开名为"example.txt"的文件，然后使用`read`系统调用读取文件内容，最后使用`close`系统调用关闭文件。



在Linux系统中，`read`和`write`是用于在文件描述符上进行读取和写入操作的两个重要系统调用。

## read和write函数
### read系统调用
**<font style="color:#DF2A3F;">换行符会刷新缓冲区</font>**

`read`系统调用用于从文件描述符中读取数据。它的基本语法如下：

```c
ssize_t read(int fd, void *buf, size_t count);
```

+ `fd`参数是要读取数据的文件描述符。
+ `buf`参数是用于存放读取数据的缓冲区。（返回值类型）
+ `count`参数是要读取的字节数。

`read`函数返回实际读取的字节数，如果返回0表示已到达文件末尾（EOF），如果返回-1表示出错，错误代码存储在`errno`中。

### write系统调用
`write`系统调用用于向文件描述符中写入数据。它的基本语法如下：

```c
ssize_t write(int fd, const void *buf, size_t count);
```

+ `fd`参数是要写入数据的文件描述符。
+ `buf`参数是要写入的数据的缓冲区。
+ `count`参数是要写入的字节数。

`write`函数返回实际写入的字节数，如果返回-1表示出错，错误代码存储在`errno`中。

### 用法示例
下面是一个简单的示例，演示了如何使用`read`和`write`函数从一个文件读取数据并写入到另一个文件中：

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main() {
    int src_fd, dest_fd;
    char buf[1024];
    ssize_t nread, nwrite;

    // 打开源文件和目标文件
    src_fd = open("source.txt", O_RDONLY);
    if (src_fd == -1) {
        perror("open source.txt");
        return 1;
    }

    dest_fd = open("destination.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (dest_fd == -1) {
        perror("open destination.txt");
        close(src_fd);
        return 1;
    }

    // 读取源文件内容，并写入目标文件
    while ((nread = read(src_fd, buf, sizeof(buf))) > 0) {
        nwrite = write(dest_fd, buf, nread);
        if (nwrite == -1) {
            perror("write");
            close(src_fd);
            close(dest_fd);
            return 1;
        }
    }

    if (nread == -1) {
        perror("read");
        close(src_fd);
        close(dest_fd);
        return 1;
    }

    // 关闭文件
    close(src_fd);
    close(dest_fd);

    return 0;
}
```

这个示例程序首先使用`open`系统调用打开源文件"source.txt"和目标文件"destination.txt"，然后使用`read`系统调用读取源文件内容，接着使用`write`系统调用将读取的数据写入目标文件。最后，使用`close`系统调用关闭文件描述符。

## lseek
在Linux系统中，`lseek`函数用于设置文件描述符的读/写位置偏移量。它的基本语法如下：

```c
off_t lseek(int fd, off_t offset, int whence);
```

+ `fd`参数是文件描述符。
+ `offset`参数是要设置的偏移量。
+ `whence`参数指定了偏移量的基准位置，它可以是以下三个值之一： 
    - `SEEK_SET`：相对于文件的起始位置进行偏移。
    - `SEEK_CUR`：相对于当前读/写位置进行偏移。
    - `SEEK_END`：相对于文件的末尾位置进行偏移。

`lseek`函数返回新的文件读/写位置偏移量，如果出错返回-1，并设置`errno`为相应的错误代码。

```cpp
lseek(fd,0,SEEK_SET) //覆盖
```

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1715669815372-8bdb1c9d-bd08-45d1-9cbc-e12ddab2c2df.png)

文件拓展又叫扩容

### 用法示例
下面是一个简单的示例，演示了如何使用`lseek`函数从文件的末尾开始读取数据：

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

#define BUF_SIZE 1024

int main() {
    int fd;
    char buf[BUF_SIZE];
    ssize_t nread;

    // 打开文件
    fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    // 将文件读取位置设置到末尾
    off_t pos = lseek(fd, 0, SEEK_END);
    if (pos == -1) {
        perror("lseek");
        close(fd);
        return 1;
    }

    // 从末尾开始读取数据
    while ((pos = lseek(fd, pos - BUF_SIZE, SEEK_SET)) != -1) {
        nread = read(fd, buf, BUF_SIZE);
        if (nread == -1) {
            perror("read");
            close(fd);
            return 1;
        }
        write(STDOUT_FILENO, buf, nread);
        if (pos == 0) break;
    }

    // 关闭文件
    close(fd);

    return 0;
}
```

这个示例程序首先使用`open`系统调用打开文件"example.txt"，然后使用`lseek`函数将文件读取位置设置到文件末尾。接着，使用`lseek`函数从文件末尾开始逐步往前读取数据，每次读取的数据量为BUF_SIZE。最后，使用`close`系统调用关闭文件描述符。















