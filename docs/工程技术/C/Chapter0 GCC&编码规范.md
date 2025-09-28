!!! tip "编程环境 [vscode-cpp](https://code.visualstudio.com/docs/cpp/config-mingw)"

    Windows:

        编辑器：VSCode
        gcc编译器：MSYS2 
        gcc添加至环境变量

    Linux:

        编辑器：VSCode
        gcc编译器：sudo apt install gcc
        查询gcc环境：gcc -v

## GCC编译器

[GCC, the GNU Compiler Collection](https://gcc.gnu.org/)

The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, Go, D, Modula-2, and COBOL as well as libraries for these languages (libstdc++,...). GCC was originally written as the compiler for the GNU operating system. The GNU system was developed to be 100% free software, free in the sense that it respects the user's freedom.

### 编译过程

预处理（#预处理指令） —— 编译（转为汇编码）—— 汇编（转为机器码）—— 链接（组合创建可执行文件）

编译器的主要任务是将高级编程语言（如C语言）的源代码转换为机器码或中间表示，以便计算机可以执行。在这一过程中，某些编译器确实会生成汇编语言作为中间步骤，但这并不是所有编译过程的必要部分。

- 预处理阶段：在这个阶段，预处理器处理所有的预处理指令（例如 #include 和 #define），并将宏替换为实际的代码。

- 编译阶段：接下来，编译器将预处理后的代码翻译成低级的汇编语言代码。对于一些编译器（如GCC），你可以通过指定特定的选项（例如 -S）来让编译器只进行到这一步并输出汇编文件，而不继续生成机器码。

- 汇编阶段：如果编译器生成了汇编代码，那么接下来的步骤就是使用汇编器将这些汇编代码转换成机器码，即目标代码（object code）。这个阶段产生的文件通常具有 .o 或 .obj 扩展名，并且包含可以直接由操作系统加载和执行的二进制数据。

- 链接阶段：最后，链接器将一个或多个目标文件与库文件组合在一起，解决符号引用问题，创建最终的可执行文件。

### 编译器特性

_attribute__实际上是GCC的一种编译器命令，用来指示编译器执行实现某些高级操作。__attribute__可以设置函数属性（Function Attribute）、变量属性（Variable Attribute）和类型属性（Type Attribute）。

## 1.命名规范

|类型|模板|示例|
|---|---|---|
|文件|小写+下划线（功能_模块）| disp_driver.c|
|函数|小写+下划线（模块_动词_名词）|disp_draw_line()|
|变量|小写+下划线|irq_count|
|全局变量|小写+前缀g_|g_systick|
|指针变量|小写+后缀_ptr|buffer_ptr|
|宏定义|大写+下划线|QUEUE_BUF_NUM|

- state与status命名

    [程序代码中，怎么区分status和state？](https://www.zhihu.com/question/21994784)

    在程序代码中似乎很好区分:因为状态机(state machine)、状态迁移图 (state transition diagram)都是明确的state,所以如果「状态」的有效值之间可以儿搞出类似状态迁移图之类的东西,就命名为state;否则就用status。

    比如TCP状态之间是有迁移关系的,所以是TCPstate;HHTTP状态码+由于没有互相迁移的关系,所以是HTTPstatuscode。

## 2.优秀编码准则

1. 维护性：一般需要解耦合度，降低各功能代码块间的耦合度
2. 复用性：需要将各功能代码块进行封装，用到时直接调用
3. 扩展性：需要应用类的可继承性。或者配合使用工厂模式，让工厂根据不同的情形实例化不同功能的对象。
4. 灵活性：需要满足以上三个特性，然后考虑实现跨平台ARM x86，可移植性等。
5. 健壮性：考虑各异常情况，尽量使代码任何时候都能工作，否则抛出异常
6. 可读性：例如if else 嵌套不要超过三层。语法不要复杂嵌套，例如多级指针

!!! note "为什么需要优雅地写代码？"

    合理的，逻辑清晰的软件架构可以节省开发时间，节省调试时间，以及减少程序的bug。

    代码运行正确性，可维护性都会提高，而且个人编程思维长进也藏在其中

## 3.注释规范

注释多不一定好，太多了会阻碍阅读的流畅性，首要（写优雅的代码，可读性强的代码，结构好的代码），次要（特殊行的注释，快速引导的注释，例如函数引导）

