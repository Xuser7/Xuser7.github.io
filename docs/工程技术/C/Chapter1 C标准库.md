## 1.stdio.h

- sprintf

    常用于数字（hex、dec、bin）转字符串ascii输出，可以使用[占位符](https://www.runoob.com/note/26809)格式化输出

    ```c
    //buffer：指向用于存储格式化输出的字符数组的指针
    //format：输出格式，可以使用占位符格式化
    //argument：表示要格式化输出的数据，分别与格式字符对应。
    int sprintf(char *buffer,const char *format[,argument...]);
    ```

## 2.string.h

- memcpy

    ```c
    //destin-- 指向用于存储复制内容的目标数组，类型强制转换为 void* 指针。
    //source-- 指向要复制的数据源，类型强制转换为 void* 指针。
    //n-- 要被复制的字节数
    void *memcpy(void *destin, void *source, unsigned n);
    ```

    !!! tip "strcpy和memcpy的区别"
  
        1、复制的内容不同。strcpy只能复制字符串，而memcpy可以复制任意内容，例如字符数组、整型、结构体、类等。

        2、复制的方法不同。strcpy不需要指定长度，它遇到被复制字符的串结束符"\0"才结束，所以容易溢出。memcpy则是根据其第3个参数决定复制的长度。
        
        3、用途不同。通常在复制字符串时用strcpy，而需要复制其他类型数据时则一般用memcpy

- memset

    ```c
    //str -- 指向要填充的内存区域的指针。
    //c -- 要设置的值，通常是一个无符号字符。
    //n -- 要被设置为该值的字节数。
    void *memset(void *str, int c, size_t n);
    ```

## 3.stdint.h

- stdint.h 的主要目的是：

    提供固定宽度的整数类型（如 int8_t、int16_t、uint8_t、uint16_t 等），确保其大小在不同平台上一致。

    定义与平台无关的整数类型（如 int_least8_t、int_fast16_t 等），用于优化性能和内存使用。

    提供最大宽度整数类型（如 intmax_t、uintmax_t），用于表示最大可能的整数。

## 4.time.h

对于需要更高精度和分辨率的时间获取需求，可以使用clock_gettime函数。clock_gettime函数提供了纳秒级的时间分辨率，适用于更加严格的时间同步和测量场景。
```c
#include <stdio.h>
#include <time.h>
int main() 
{   
    struct timespec ts;    
    clock_gettime(CLOCK_REALTIME, &ts);    
    printf("Seconds: %ldn", ts.tv_sec);//自1970年1月1日以来的秒数
    printf("Nanoseconds: %ldn", ts.tv_nsec);//当前秒内的纳秒数
    return 0;
}
```
