## typedef 定义重命名

typedef 关键字的作用是给数据类型起一个新的名字

```c
//如：为单字节整数定义别名Byte
typedef uint8_t Byte;
Byte info = 0；
```

```c
//常用于别名定义的结构体
//创建结构体时可以使用别名而不加关键字
typedef struct
{
  char[] name;
  unsigned int price;
  ...
} Book;

Book CPrimerPlus
```

## struct 结构体

struct 的作用是定义一个结构类型，结构是一个复合的类型，其构成元素可以是基本数据类型(char short int float long double)等，也可以是复合类型(数组，指针，结构，联合)。

### 定义结构体

[简单分析C语言中typedef struct 与 struct 的区别](https://zhuanlan.zhihu.com/p/87364251)

使用 `struct` 定义结构体，创建结构体时必须使用struct关键字

​使用 `typedef struct` 定义结构体并且别名，创建结构体时可以使用别名而不加关键字


1. CASE1

    这里，创建了一个变量x，包含两个成员，一个字符a，一个整数b。

    ```C
    struct{
    char a;
    int b;
    }x;

    x.a = 'A';
    x.b = 10;
    ```

2. CASE2

    这里，创建了一个tag标签,为该类型提供了一个STUDENT的名字。

    声明变量，可以通过 struct STUDENT x; 

    ```c
    struct STUDENT{
    char name;
    int age;
    };    

    struct STUDENT x;
    ```

3. CASE3

    这里的作用和CASE2基本一致，STUDENT现在是一个数据类型的名字。

    声明变量，就可以直接写为 STUDENT x;
    ```c
    typedef struct{
    char name;
    int age;
    }STUDENT;

    STUDENT x;
    ```


4. CASE4

    这是创建链表节点的一种常见写法，可以分为两步

    ```c
    typedef struct NODE{
    int data;
    struct NODE* next;
    }node;
    //①创建了一个叫NODE的结构类型，
    //②typedef NODE node;   把 NODE 这种数据类型命名为 node
    ```

### 结构体指针

```c
typedef struct {
    Elemtype member1;
    Elemtype member2;
    ...
​}struct_type;

struct_type* type_ptr;
```

- 访问结构体成员

    结构体变量用 . 来访问结构体的成员

    ```c
    typedef struct {
        Elemtype member1;
        Elemtype member2;
    ...
    ​}struct_type;
    ​
    struct_type s1;

    s1.member1;
    s1.member2;
    ```

- 访问结构体指针的结构体成员

    指向结构体的结构体指针，要用->来访问指向的结构体成员

    ```c
    typedef struct {
        Elemtype member1;
        Elemtype member2;
    ...
    ​}struct_type;
    ​
    struct_type s1;
    struct_type* struct_ptr = s1; 

    struct_ptr->var1; 
    struct_ptr->var2;
    ```

- 访问结构体指针的结构体成员，该结构体嵌套的内部结构体

    一个指针指向结构体1，而结构体1成员有一个结构体2，通过该指针访问结构体2的成员

    如：结构体1->结构体1内的结构体2`->`结构体2的成员，第二个`->`换成`.`

    ```c
    typedef struct {
        char member1;
        int member2;
        another_struct member3;
    ...
    ​}struct_type;
    ​
    struct_type s1;
    struct_type* struct_ptr = s1; 

    struct_ptr->member1; 
    struct_ptr->member3.member;
    ```

### 结构体面向对象

(C结构体)和(C++类)在使用上很类似，结构体甚至可以用面向对象的思想来形容一类对象。结构体具备着面向对象思想中封装的特性，但是它不具备继承和多态的特性，因此大大减少了它的使用频率

[C语言结构体的“继承”](https://www.cnblogs.com/lknlfy/archive/2013/01/06/2848348.html)

### 位域

位域（bit-field）是一种特殊的结构体成员，允许我们按位对成员进行定义，指定其占用的位数

在本质上就是一种结构类型，不过其成员是按二进位分配的。

```c
typedef struct{
    int status:1;
    int  :2;    /* 该 2 位不能使用 */
    int mode:3;
    int temp:2;
}Device;

//上述结构体的值就是status为低1位，隔两位为mode占3位，最高两位temp
Device d; 
d.status = 1;//1
d.mode=7;//111 
d.temp=0;//00

//测试
prinf("%x",d) //0x39=00111001
```

## 修饰变量关键字

### extern 关键字

在 C 语言中，使用 extern 关键字声明的变量是外部变量，表示该变量在其他文件中定义。

extern 关键字在 C 语言中用于声明外部变量或函数，使得它们可以在多个文件中共享和使用。

extern 主要用于实现模块化编程和代码的分离。 extern 变量的声明和定义通常放在不同的文件中。

```c
//////////////////file1.c//////////////////
#include <stdio.h>

int var_file1 = 0xff;

void printMessage() {
    printf("Hello from printMessage!\n");
}

//////////////////main.c//////////////////
#include <stdio.h>

// 声明外部变量
extern int var_file1;
extern printMessage();

int main() {
    // 使用外部变量
    printf("var_file1: %d\n", var_file1);
   
    // 调用其他文件中定义的函数
    printMessage();
   
    return 0;
}

```

### const 关键字

```c
//修饰MAX_VALUE变量为常量，且永远为100不可修改
const int MAX_VALUE = 100;
```

!!! tip "\#define 与 const 区别"

    \#define 与 const 这两种方式都可以用来定义常量，选择哪种方式取决于具体的需求和编程习惯。通常情况下，建议使用 const 关键字来定义常量，因为它具有类型检查和作用域的优势，而 #define 仅进行简单的文本替换，可能会导致一些意外的问题。

    - 替换机制：#define 是进行简单的文本替换，而 const 是声明一个具有类型的常量。#define 定义的常量在编译时会被直接替换为其对应的值，而 const 定义的常量在程序运行时会分配内存，并且具有类型信息。

    - 类型检查：#define 不进行类型检查，因为它只是进行简单的文本替换。而 const 定义的常量具有类型信息，编译器可以对其进行类型检查。这可以帮助捕获一些潜在的类型错误。

    - 作用域：#define 定义的常量没有作用域限制，它在定义之后的整个代码中都有效。而 const 定义的常量具有块级作用域，只在其定义所在的作用域内有效。

    - 调试和符号表：使用 #define 定义的常量在符号表中不会有相应的条目，因为它只是进行文本替换。而使用 const 定义的常量会在符号表中有相应的条目，有助于调试和可读性。

### static 关键字

static 修饰的变量存放在全局数据区的静态变量区，包括全局静态变量和局部静态变量，都在全局数据区分配内存。初始化的时候自动初始化为 0。


- static 修饰局部变量：只执行初始化一次（若不指定初始值，则默认为0）。该修饰延长了局部变量的生命周期，也就是退出当前函数时，下一次进入函数时该变量不被初始化保持上次退出的值。

    ```c
    //example
    uint8_t func(void)
    {
        static uint8_t count_into_func = 0;
        //统计访问次数并且返回
        count_into_func++;
        return count_into_func;
    }
    ```

- static 修饰全局变量：这个全局变量只能在本文件中访问，不能在其它文件中访问，即便是 extern 外部声明也无法访问。

- static 修饰一个函数：则这个函数的只能在本文件中调用，不能被其他文件调用。

    ```c
    static void key_event_handle(void)
    {
        //function_action
    }

    static void display_event_handle(void)
    {
        //function_action
    }

    static void timer_event_handle(void)
    {
        //function_action
    }
    ```

### volatile 关键字

volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改。

volatile 用于告诉编译器，​变量的值可能会在程序执行期间被意外地改变，​因此编译器不应该对该变量进行优化，从而可以提供对特殊地址的稳定访问。当要使用 volatile 声明的变量的值的时候，系统总是重新从它所在的内存读取数据，而且读取的数据立刻被保存。（即使它前面的指令刚刚从该处读取过数据）

- 多线程同步

    ```c
    volatile int i=10;
    int a = i;
    ...
    // 其他代码，不含对i的操作。
    // 故编译器明确此处未对i进行过操作。对其进行优化。
    // 但i有可能被外部程序更改，例如中断，操作系统，多线程等等。
    int b = i;
    ```

    上述 volatile 指出 i 是随时可能发生变化的，每次使用它的时候必须从 i 的地址中读取，因而编译器生成的汇编代码会重新从i的地址读取数据放在 b 中。

    而编译器优化做法是，由于编译器发现两次从 i读数据的代码之间的代码没有对 i 进行过操作，它会自动把上次读的数据放在 b 中。而不是重新从 i 里面读。这样以来，如果 i是一个寄存器变量或者表示一个端口数据就容易出错，所以说 volatile 可以保证对特殊地址的稳定访问。

- 访问硬件寄存器

    在嵌入式系统编程中，我们经常需要访问硬件寄存器。由于硬件寄存器的值可能会被意外地改变，因此我们需要使用 volatile 关键字来告诉编译器，变量的值可能会被意外地改变。

!!! tip "volatile与原子操作区别"

    原子性操作具有不可分割性。非原子操作都会存在线程安全问题，需要我们使用同步技术（sychronized）来让它变成一个原子操作。一个操作是原子操作，那么我们称它具有原子性。

    volatile关键字能保证可见性没有错，但是上面的程序错在没能保证原子性。可见性只能保证每次读取的是最新的值，但是volatile没办法保证对变量的操作的原子性。

    volatile保证数据实时性，但非稳定。原子操作保证数据的稳定性

    [Volatile关键字与原子性操作](https://blog.csdn.net/daijingxin/article/details/113767433)