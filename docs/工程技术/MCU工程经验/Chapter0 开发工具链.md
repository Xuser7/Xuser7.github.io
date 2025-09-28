## 1 Compiler

[ARM 主流编译器（armcc、iar、gcc for arm、LLVM(clang)）](https://cloud.tencent.com/developer/article/2032451)

- ARMCC

- IAR

- GCC for ARM

## 2 IDE

### IAR Embedded Workbench

[对 IAR Embedded Workbench 文件进行版本控制](https://mypages.iar.com/s/article/IAR-Embedded-Workbench-files-to-be-version-controlled?language=zh_CN)

- icf文件（链接文件）

    icf主要作用就是定义内存位置、内存大小和堆栈大小。每个芯片开发商都会针对每款芯片来编写一个.icf文件即链接文件。对于基本的应用，这个.icf文件足以满足你的工程需要。但有时也会需要改动，比如当你的项目要添加外部RAM时、修改app存放地址时。都要修改icf。

- map文件

    通过map文件也可以查看代码的的存储信息，如变量大小存放位置、函数大小存放位置等等。以及查看固件的存储信息，ROM占用和RAM占用

## 3 Debug Tool

### SEGGER JLink

- JLink RTT 

    [调试备忘录-J-Link RTT的使用（原理 + 教程 + 应用 + 代码）](https://www.cnblogs.com/snowsad/p/12076740.html)
  
    RTT(Real Time Transfer)是一种用于嵌入式中与用户进行交互的技术，它结合了SWO和半主机的优点，具有极高的性能。

    使用RTT可以从MCU非常快速输出调试信息和数据，且不影响MCU实时性。这个功能可以用于很多支持J-Link的设备和MCU，兼容性强。

- JScope 

    [调试神器---＞JScope](https://blog.csdn.net/qq_23852045/article/details/108837881)

    J-Scope、可以在目标MCU运行时，实时分析数据并图形化显示的软件。它不需要SWO或目标上的任何额外引脚等功能，但使用可用的标准调试端口。J-Scope可以以类似示波器的方式显示多个变量的值。它读取elf或axf文件并允许选择多个变量进行可视化。

    ```
    HSS模式是通过采样周期定时从内存文件中读取变量的值，所以采样周期和可执行文件是必须的，
    为了更加准确有效的采集到对的数据，最好用volatile声明变量。

    优点：随时可连接MCU，不影响MCU正常运行，因为不需添加任何代码，所以也不会占用MCU紧张的资源。
    缺点：速度慢，采样速率基本固定在1khz左右，因此仅仅适合采样变量变化速率低于1khz的情况，
    因为数据是根据采样率来的，所以实时性不是太准，不够低速率下影响不大。
    ```

    ```
    RTT模式下，所有的数据和时间戳均是有MCU来提供。

    优点：比HSS更高的数据吞吐量，最高可达2MB/S，不过这个是由MCU上使用的缓冲区大小决定，
    即使只有512字节的小缓冲区也可以达到1MB/S，够用了；J-Scope数据采集与MCU的应用程序执行同步，
    因为应用程序决定何时以及如何采样数据；不需要知道变量位置，
    RTT缓冲区的位置由J-Scope自动检测；时间戳等数据可以被添加到数据样本中。
    缺点：稍微比较麻烦，需要移植RTT代码，占用mcu资源，大概需要1.4kb左右的flash，1KB左右的RAM。
    ```

### lks scope

一款国产的上位机跟踪软件，和JScope一样的功能，但是更方便、界面更完善，推荐使用

[南京凌鸥创芯](https://www.lksmcu.com/index.php/Tools/)