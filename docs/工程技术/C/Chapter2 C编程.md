
[菜鸟教程 - C 教程](https://www.runoob.com/cprogramming/c-tutorial.html)

## 1.数据类型、变量

- 变量未初始化初值

    C 语言中变量的默认值取决于其类型和作用域。

    全局变量和静态变量的默认值为 0，

    字符型变量的默认值为 \0，

    指针变量的默认值为 NULL，

    局部变量（在函数内部定义的非静态变量）不会自动初始化为默认值，它们的初始值是未定义的（包含垃圾值）。因此，在使用局部变量之前，应该显式地为其赋予一个初始值。

### 数据类型转化

隐式类型转换：隐式类型转换是在表达式中自动发生的，无需进行任何明确的指令或函数调用。它通常是将一种较小的类型自动转换为较大的类型，例如，将int类型转换为long类型或float类型转换为double类型。隐式类型转换也可能会导致数据精度丢失或数据截断。

```c
int i = 10;
float f = 3.14;
double d = i + f; // 隐式将int类型转换为double类型
```

显式类型转换：显式类型转换需要使用强制类型转换运算符（type casting operator），它可以将一个数据类型的值强制转换为另一种数据类型的值。强制类型转换可以使程序员在必要时对数据类型进行更精确的控制，但也可能会导致数据丢失或截断。

```c
double d = 3.14159;
int i = (int)d; // 显式将double类型转换为int类型
```

## 2.运算符

### 三元运算符

`(a>0)?b:c`

判断是否表达式a>0是否为真，true取b，false取c

用于替代简单的if-else判断

## 3.C指针

### 定义指针

指针解址符—— * 

指针取址符—— &

```c
//名为p的int型指针变量，指向了地址a
int * p = &a

//​通过指针传参要使用&传入地址
void func(int *p)
{
    p = 100;
    ...
}
func(&a);
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


### 函数指针与回调函数

[菜鸟教程 - C 语言回调函数详解](https://www.runoob.com/w3cnote/c-callback-function.html)

[博客园 - 回调函数实战](https://www.cnblogs.com/xiaokang-coding/p/18801623)

- 函数指针

    函数指针是指向函数的指针变量。
```c
// 声明一个指向同样参数、返回值的函数指针类型
void Function(int num,int (*func)(void)){ 
    num = func(); 
}
```

- 回调函数 
    
    你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里，你的电话号码就叫回调函数，你把电话留给店员就叫登记回调函数，店里后来有货了叫做触发了回调关联的事件，店员给你打电话叫做调用回调函数，你到店里去取货叫做响应回调事件。
```c
// 事件类型枚举
typedef enum { 
    CLICK,
    MIDDLE_CLICK,
    RIGHT_CLICK,
    KEY_PRESS,
    KEY_RELEASE,
    MOUSE_MOVE,
    SCROLL,
}EventType;

// 事件结构体
typedef struct {
    EventType type;
    int x, y;
    char key;
}Event;

// 定义回调函数类型
//EventCallback 函数指针数据类型，Event 函数变量
typedef void (*EventCallback)(const Event*);

// 事件处理器结构体
typedef struct {
    EventCallback callbacks[10];  // 假设最多10种事件类型，对应最多10个处理函数
}EventHandler;

// 注册事件回调
void registerCallback(EventHandler* handler, EventType type, EventCallback callback) {
    handler->callbacks[type] = callback; //函数指针赋值
}

// 事件分发器
void dispatchEvent(EventHandler* handler, const Event* event) {
    if (handler->callbacks[event->type] != NULL) {
        handler->callbacks[event->type](event);
    }
}

// 点击事件处理函数
void onClickCallback(const Event* event) {
    printf("点击事件触发了！坐标: (%d, %d)\n", event->x, event->y);
}
// 按键事件处理函数
void onKeyPressCallback(const Event* event) {
    printf("按键事件触发了！按下的键是: %c\n", event->key);
}

void main() {
    // 创建并初始化事件处理器
    EventHandler handler = {NULL};
    
    // 注册回调函数
    registerCallback(&handler, CLICK, onClickCallback);
    
    // 模拟点击事件
    Event clickEvent = {CLICK, 100, 200};

    // 事件触发
    dispatchEvent(&handler, &clickEvent);

}
```

## 4.C函数

### 定义函数

### 指针函数

指针函数指的是返回值为

## 5.C动态内存分配

```c
void *calloc(int num, int size);
//在内存中动态地分配 num 个长度为 size 的连续空间，并将每一个字节都初始化为 0。
//所以它的结果是分配了 num*size 个字节长度的内存空间，并且每个字节的值都是 0。
```

```c
void *malloc(int num);//num通常使用sizeof(var)定大小
//在堆区分配一块指定大小的内存空间，用来存放数据。
//这块内存空间在函数执行完成后不会被初始化，它们的值是未知的。
```

```c
void free(void *address);
//该函数释放 address 所指向的内存块,释放的是动态分配的内存空间。
```

```c
void *realloc(void *address, int newsize);
//该函数重新分配内存，把内存扩展到 newsize。
```




