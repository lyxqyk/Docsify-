GCC编译器是Linux系统下最常用的CIC++编译器，大部分Linux发行版中都会默认安装。GCC编译器通常以gcc指令的形式在终端中使用。

## 一.gcc指令
### 1.直接编译
创建a.c文件

```objectivec
#使用gcc对程序进行编译，默认得到可执行文件的文件名为a.out
gcc [文件名].c
gcc a.c
./a.out
```

### 2.分步编译
-o:重命名.out文件，默认是a.out

```objectivec
#预处理
--对.c程序文件进行预处理得到.i预处理文件（还是C语言）
gcc -E[文件名].c -o[文件名].i
gcc -E a.c -o a.i

#编译
--通过编译得到.s汇编文件(如果操作对象是.c文件，会进行预处理+编译)
gcc -S [文件名].i
gcc -S a.i

#汇编
--通过汇编得到 hello.o机器码文件(如果操作对象是.c文件，会进行预处理+编译+汇编;如果操作对象是.i文件?)
gcc -c[文件名].s
gcc -c a.s

#链接
--通过链接得到a.out可执行文件(如果操作对象是.c文件，进行完整编译步骤)
gcc [文件名].o
gcc a.o -o a
```

## 二、编译步骤
从hello.c到hello (或a.out）文件，必须历经hello.i、hello.s、hello.o、hello (或a.out)）文件，分别对应着预处理、编译、汇编和链接4个步骤，整个过程如图所示:

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1709370237577-3136f136-0591-45c2-a64c-278aa0376835.png)

+ 预处理:通过预处理器处理头文件展开、宏定义扩展等，生成.i文件，还是C代码;
+ 编译:将预处理得到的源代码文件进行语法词法分析,“翻译转换"产生出机器语言的.s文件，得到机器语言的汇编文件;
+ 汇编:将汇编代码翻译成了机器码，得到.o目标文件，但是还不可以运行;
+ 链接:处理可重定位文件，把各种符号引用和符号定义转换成为可执行文件中的合适信息，通常是虚拟地址。

## 三、目标文件
创建a.c文件，a.c中调用外部函数thanks()

```objectivec
#include<stdio.h>

void print_thank();

int main(int argc,char *argv[])
{
    print_thank();
    printf("Hello World!\n");
    return 0;
}
```

创建thank.c包含 ptintf_thank()函数

```objectivec
#include<stdio.h>
void print_thank ( ) ;
{
    printf("Thank you\n");
}
```

```objectivec
gcc a.c thank.c
./a.out
```

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1709370970623-31bfebcc-cc10-43c5-b175-005a5a657df7.png)

当多文件编译时，如果更新了某个文件的内容，需要将文件都重新编译。只需要重新编译二进制.o文件，再重新生成.out文件

```objectivec
#修改了thank.c文件
gcc -c thank.c
gcc a.o thank.o
./a.out
```

目标文件常常按照特定格式来组织，在Linux下，它是ELF格式(Executable LinkableFormat，可执行可链接格式)。

而通常目标文件有三种形式:

+ 可执行目标文件(executable)。即我们通常所认识的，可直接运行的二进制文件。
+ 可重定位目标文件(relocatable)。包含了二进制的代码和数据，可以与其他可重定位目标文件合并，并创建一个可执行目标文件。(.o文件)
+ 共享目标文件(shared object)。它是一种在加载或者运行时进行链接的特殊可重定位目标文件。

  

















