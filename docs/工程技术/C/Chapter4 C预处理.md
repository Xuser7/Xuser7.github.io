
!!! note "预处理器"

    C 预处理器(CPP)是编译过程中的独立阶段，在实际编译前对源代码进行文本处理。主要功能包括：

    - 文件包含

    - 条件编译

    - 特殊指令处理

    - 宏展开

    Chapter0中了解到在将一个C源程序转换为可执行程序的过程中, 编译预处理是最初的步骤。​这一步骤是由预处理器来完成的。

    我们将把 C 预处理器（C Preprocessor）简写为 CPP。C 预处理器不是编译器的组成部分，但是它是编译过程中一个单独的步骤。

    简言之，C 预处理器只不过是一个文本替换工具而已，它们会指示编译器在实际编译之前完成所需的预处理。

    所有的预处理器命令都是以井号 # 开头。它必须是第一个非空字符，为了增强可读性，预处理器指令应从第一列开始。

## 1.#define 定义宏

`#define A  B`， A和B文本是相互转换的 ，宏定义本质：文本替换

### 指定文本替换数据类型

既然定义宏本质只是文本替换，那么数据是以默认形式替换的，即默认为有符号整型，相当于signed int。

```c
默认：默认为有符号整型，相当于 signed int
U(u)：表示该常数用无符号整型方式存储，相当于 unsigned int
L(l)：表示该常数用有符号长整型方式存储，相当于 signed long
LL(ll)：表示该常数用有符号长长整型方式存储，相当于 signed long long
UL(ul)：表示该常数用无符号长整型方式存储，相当于 unsigned int
ULL(ull)：表示该常数用无符号长长整型方式存储，相当于 unsigned int
F(f)：表示该常数用浮点方式存储，相当于 float
```

### 替换与连接

```c
//# ：将传入文本，添加上“”，如下，TXT会被替换为“TXT”文本
# define PRINT(TXT) printf(# TXT)

//##：将传入文本相连接，如下，连接文本为AABB
# define LINK(AA,BB) AA##BB
```

### 优秀代码中的常用定义宏

```c
//获取数据大小
#define ARR_SIZE(n) (sizeof(n)/sizeof((n)[0]))
//位操作
#define SET_BIT(reg, bit)     ((reg) |= (bit))
#define CLEAR_BIT(reg, bit)   ((reg) &= ~(bit))
#define READ_BIT(reg, bit)    ((reg) & (bit))
#define CLEAR_REG(reg)        ((reg) = (0x0))
#define WRITE_REG(reg, value)   ((reg) = (value))
#define READ_REG(reg)         ((reg))
#define MODIFY_REG(reg, CLEARMASK, SETMASK)  WRITE_REG((reg), (((READ_REG(reg)) & (~(CLEARMASK))) | (SETMASK)))
#define POSITION_VAL(value)     (__CLZ(__RBIT(value)))
```

### 带参宏

## 2.#include 文件包含

## 3.条件编译宏

顾名思义，根据条件选择参与编译的语句。条件编译宏以 #endif 结尾。

### 常见条件编译

- 防止头文件重复定义

    ```C
    #ifndef HEADER_FILE
    #define HEADER_FILE
    ...
    ...
    #endif
    ```
    
- 条件引用头文件

    ```c
    #if CASE_1
    # include "headfile_1.h"
    #elif CASE_2
    # include "headfile_2.h"
    #elif CASE_3
    ...
    #else
    ...
    #endif
    ```

### ifdef和if defined （ifndef和if !defined）

`#if defined` 可以组成复杂的预编译条件

`#ifdef` 只判断单个宏是否定义

```c
//例
#if defined(AAA) && !defined(BBB)
//例
#if defined(AAA) || VERSION > 12
```

## 4.特殊指令

### #error 标准错误输出消息

### #pragma 编译器内存对齐

常用于设置结构体内存对齐，使之按照固定大小补位

需要使用 #pargma pack(push,1) 和 #pragma pack(pop) 类似代码将结构体包裹起来，形式如下。


```c
//以下代码运行后。我们会得到：
//pack_size:5
//unpack_size:8
#pragma pack(push)
#pragma pack(1)
typedef struct 
{
    char a;
    float b;
}pack;
#pragma pack(pop)

typedef struct 
{
    char a;
    float b;
}unpack;

int main(void)
{
    pack pack;
    unpack unpack;

    printf("pack_size:%d\n",sizeof(pack));
    printf("unpack_size:%d\n",sizeof(unpack));

}
```

在C语言中，结构是一种复合类型，其构成元素可以是基本数据类型(char short int float long double)等，也可以是复合类型(数组，指针，结构，联合)。在结构中，编译器为结构中的每个成员按其自然对界(alignment)条件分配空间，各成员按照被声明的顺序在内存中顺序存储，第一个成员的地址和整个结构的地址相同。

如果不用#pargma pack()包裹，则结构体按编译器默认对其方式（成员中size最大的那个）对齐。这里包括以下三个原则：

- 原则1：数据成员对齐规则：结构（struct或联合union）的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员存储的起始位置要从该成员大小的整数倍开始（比如int在32位机为4字节，则要从4的整数倍地址开始存储）。

- 原则2：结构体作为成员：如果一个结构里有某些结构体成员，则结构体成员要从其内部最大元素大小的整数倍地址开始存储。（struct a里有struct b，b里有char，int，double等元素，那b应该从8的整数倍开始存储。）

- 原则3：收尾工作：结构体的总大小，也就是sizeof的结果，必须是其内部最大成员的整数倍，不足的要补齐。

[C #pragma pack(push,1) #pragma pack(pop)解析](https://blog.csdn.net/qq_22398523/article/details/81671121)

## GCC编译器内置宏

ANSI C 定义了许多宏。在编程中您可以使用这些宏，但是不能直接修改这些预定义的宏。

```c
__DATE__：当前日期，以”MMM DD YYYY“的格式表示字符常量
printf("Date :%s\n", __DATE__ );  

__TIME__：当前日期，以”HH:MM:SS“的格式表示字符常量
printf("Time :%s\n", __TIME__ );  

__FILE__：会包含当前文件名，一个字符串常量
printf("File :%s\n", __FILE__ );  

__LINE__：包含当前行号，一个十进制常量
printf("Line :%d\n", __LINE__ );  

__FUNCTION__ ：表示正在处理的函数名
printf("Function:%s\n",__FUNCTION__);

__STDC__：当编译器以ANSI标准编译时，则定义为1
printf("ANSI :%d\n", __STDC__ );
```
