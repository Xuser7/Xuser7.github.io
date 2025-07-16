
## 1 编码规范

### 1.1 命名规范
|类型|模板|示例|
|---|---|---|
|文件|小写+横杠（功能/模块 功能_模块）| disp_driver.c|
|函数|小写+下划线（模块_动词_名词）|disp_draw_line()|
|变量|小写+下划线|interrupt_count|
|宏定义|大写+下划线|QUEUE_BUF_NUM|

### 1.2 优秀编码码准则

1. 维护性：一般需要解耦合度，降低各功能代码块间的耦合度
2. 复用性：需要将各功能代码块进行封装，用到时直接调用
3. 扩展性：需要应用类的可继承性。或者配合使用工厂模式，让工厂根据不同的情形实例化不同功能的对象。
4. 灵活性：需要满足以上三个特性，然后考虑实现跨平台ARM x86，可移植性等。
5. 健壮性：考虑各异常情况，尽量使代码任何时候都能工作，否则抛出异常
6. 可读性：例如if else 嵌套不要超过三层。语法不要复杂嵌套，例如多级指针

### 1.3 NASA 安全代码规范

1. 将所有代码限制为非常简单的控制流结构，不要使用goto语句、setjmp 或 longjmp 构造以及直接或间接的递归调用
2. 所有循环都必须有一个固定的上界。
3. 初始化后不要使用动态内存分配。
4. 任何函数都都不应超过可以打印在单张论文纸上的长度，每条语句一行，每条语句一行声明。通常，这意味着每个函数不超过 60 行代码。
5. 每个函数的断言，至少要有两个。
6. 数据对象必须在尽可能小的范围内声明
7. 函数的返回值和函数的形参有效性必须做检测。
8. 预定义宏限用于包含的头文件和简单的宏定义。
9. 指针的使用应该受到限制。具体来说，不超过一级指针。指针解引用操作不能隐藏在宏中定义或在 typedef 声明中。不允许使用函数指针（使用函数指针后，分析功能可能无法检测是否有递归问题）。
10. 从编写代码的第1天开始，编译器的所有编译警告设置要全开，所有的代码编译后必须零警告，并且使用静态分析工具分析后，也必须保证零警告。

## 2 C常识

### static关键字

static 修饰的变量存放在全局数据区的静态变量区，包括全局静态变量和局部静态变量，都在全局数据区分配内存。初始化的时候自动初始化为 0。


1. static 修饰局部变量：只执行初始化一次（若不指定初始值，则默认为0）。该修饰延长了局部变量的生命周期，也就是退出当前函数时，下一次进入函数时该变量不被初始化保持上次退出的值。
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
2. static 修饰全局变量：这个全局变量只能在本文件中访问，不能在其它文件中访问，即便是 extern 外部声明也无法访问。

3. static 修饰一个函数：则这个函数的只能在本文件中调用，不能被其他文件调用。
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

### volatile关键字

volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改。

比如：操作系统、硬件或者其它线程等。遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。

当要求使用 volatile 声明的变量的值的时候，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据。而且读取的数据立刻被保存。
```c
volatile int i=10;
int a = i;
...
// 其他代码，不含对i的操作。
// 故编译器明确此处未对i进行过操作。对其进行优化。
// 但i有可能被外部程序更改，例如中断，操作系统，多线程等等。
int b = i;
```
volatile 指出 i 是随时可能发生变化的，每次使用它的时候必须从 i的地址中读取，因而编译器生成的汇编代码会重新从i的地址读取数据放在 b 中。

而编译器优化做法是，由于编译器发现两次从 i读数据的代码之间的代码没有对 i 进行过操作，它会自动把上次读的数据放在 b 中。而不是重新从 i 里面读。这样以来，如果 i是一个寄存器变量或者表示一个端口数据就容易出错，所以说 volatile 可以保证对特殊地址的稳定访问。

!!! tip "volatile与原子操作"

    原子性操作具有不可分割性。非原子操作都会存在线程安全问题，需要我们使用同步技术（sychronized）来让它变成一个原子操作。一个操作是原子操作，那么我们称它具有原子性。

    volatile关键字能保证可见性没有错，但是上面的程序错在没能保证原子性。可见性只能保证每次读取的是最新的值，但是volatile没办法保证对变量的操作的原子性。

    volatile保证数据实时性，但非稳定。原子操作保证数据的稳定性

    [Volatile关键字与原子性操作](https://blog.csdn.net/daijingxin/article/details/113767433)

## 3 经验累积

### 使用枚举类型，增加代码可读性

```C
//状态机状态枚举
enum
{
    uint8 STATE1 = 1,//递增
    uint8 STATE2,
    uint8 STATE3,
    uint8 STATE4,
    uint8 STATE5,
}State_Type;
```

### 使用结构体，描述对象

例如描述ModbusRTU的协议帧

```C
typedef struct
{
    uint8 device_addr;//从机地址域
    uint8 func_code;//功能码
    uint8 data[];//数据域
    uint8 CRC_L;
    uint8 CRC_H;
}ModbusRTU_Frame;
```

### 使用MAP思想映射，记录数据

例如，通过ascii值记录出现过的字母,以及出现过的频次

```C

char str[];
int ascii_map[200]={0};

for(int i=0;i<strlen(str);i++)
{
    ascii_map[str[i]]+=1;
}
```

### 使用MASK思想，记录标志位

例如，一个16位的数据类型，uint16。可以使用位操作对16个位赋予1/0状态。用于标记状态位

```C

uint16 MASK;

for(int i=0;i<16;i++)
{
    if(1 = (MASK>>i)&0x01)
    {
        //action...
    }
    else
    {
        //action...
    }
}
```

## C数据结构

### 链表

```C
//链表节点结构体
typedef struct ListNode {
    int val;
    struct ListNode* next;
}LNode,*Linklist;
```

以下代码均以带头节点的链表为例子

```C
//链表初始化
LinkList InitList()   
{
    LinkList L=(LNode *)malloc(sizeof(LNode));   //为头结点动态分配内存
    L->next=NULL;  //初始化时头结点指向空
    return L;
}
```

```C
//插入

//头插法：插入数据逻辑简单，但是插入数据与在链表中的排序顺序相反
void HeadInsertList(LinkList L,int data)  
{
     scanf("%d",&data);
     LNode *p=(LinkList)malloc(sizeof(LNode));
     p->val=data;
     p->next=L->next;
     L->next=p; 
}
//尾插法：相对于头插法来说，数据的插入顺序与在链表中的数据顺序时一致的
void EndInsertList(LinkList L,int data)
{
    LNode *End;
    End=L;    
    while(End->next)
    {      
         End=End->next;   	
    }    
    LNode *p=(LinkList)malloc(sizeof(LNode));
    p->val=data;
    End->next=p;
    p->next=NULL;
}
```

```C
//遍历
void  TraverList(LinkList L)
{
    LNode *p=L->next;
    while(p)   //由于最后的节点指向空，可以通过这一特点结束遍历
    {
        printf("%d ",p->val);
        p=p->next;
    }
}
```

### 树

```C
//二叉树节点结构体
typedef struct BTNode{
        char data;
        struct BTNode* left;
        struct BTNode* right;
};
```