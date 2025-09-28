
!!! abstract "[FreeRTOS](https://www.freertos.org/zh-cn-cmn-s)"

    - FreeRTOS 是 开源 的实时操作系统（Real-time operating system, RTOS）
    
    - FreeRTOS 简单、小巧、易用，通常情况下内核占用 4k-9k 字节的空间  

## 零、移植FreeRTOS内核

[MDK移植：STM32F103移植FreeRTOS完整过程](https://blog.csdn.net/qq_36973838/article/details/121754908?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522171239934216800182125629%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=171239934216800182125629&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-121754908-null-null.142^v100^pc_search_result_base7&utm_term=stm32%E7%A7%BB%E6%A4%8Dfreertos&spm=1018.2226.3001.4187)

[IAR移植：IAR移植FreeRTOS——笔记](https://blog.csdn.net/qq_39764337/article/details/101511484)

- 拷贝源码到工程文件夹

        1在工程目录下创建FreeRTOS文件夹并创建子文件夹src和port
        2将Source/include文件夹拷贝到FreeRTOS下
        3将Source中的C文件拷贝到FreeRTOS/src下
        4将Source/portable下的MemMang和RVDS文件夹拷贝到FreeRTOS/port下
        （其中MemMang下的源文件是用于堆栈管理的，RVDS下的源文件是不同内核相关的接口文件）
        5拷贝FreeRTOSv9.0.0\FreeRTOS\Demo 对应工程下的FreeRTOSConfig.h到工程

- 添加源码到工程

        1新建FreeRTOS/src和FreeRTOS/port组
        2FreeRTOS/src组中把FreeRTOS/src文件夹中的源文件全部添加
        3FreeRTOS/port组添加FreeRTOS\port\MemMang中的heap4.c

        和FreeRTOS\port\RVDS\ARM_CM3（MDK）的port.c
        和FreeRTOS\port\IAR\ARM_CM3（IAR）的port.c

        ​4FreeRTOSConfig.h添加到USER

- 添加头文件路径

        如果是IAR工程，assembler-->preprocesser
        添加FreeRTOSConfig.h头文件的路径，编译portasm.s时需要

- 修改配置

        1 在FreeRTOSConfig.h中添加
            #define xPortPendSVHandler 	PendSV_Handler
            #define vPortSVCHandler 	SVC_Handler
        
        2 在stm32f10x_it.c中屏蔽PendSV_Handler和SVC_Handler中断

        3最关键的一步，修改stm32f10x_it.c中的systick中断服务函数。

    ```c
    extern void xPortSysTickHandler(void);
    void SysTick_Handler(void)
    {
        #if (INCLUDE_xTaskGetSchedulerState  == 1 )
        if (xTaskGetSchedulerState() != taskSCHEDULER_NOT_STARTED)
        {
        #endif  /* INCLUDE_xTaskGetSchedulerState */  
            xPortSysTickHandler();
        #if (INCLUDE_xTaskGetSchedulerState  == 1 )
        }
        #endif  /* INCLUDE_xTaskGetSchedulerState */
    }
    ```

- 验证demo

    ```c
    #include "stm32f10x.h"
    #include "FreeRTOS.h"
    #include "task.h"
    #include "portmacro.h"
    static TaskHandle_t led_task_handle = NULL;
    void LED_GPIO_Config(void)	
    {
    GPIO_InitTypeDef GPIO_InitStructure;
    RCC_APB2PeriphClockCmd( RCC_APB2Periph_GPIOC, ENABLE); // 使能PC端口时钟  
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;	//选择对应的引脚
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;       
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOC, &GPIO_InitStructure);  //初始化PC端口
    GPIO_SetBits(GPIOC, GPIO_Pin_13 );	 // 关闭所有LED
    }
    void led_task(void *arg)
    {
        while(1)                            
        {
            GPIO_SetBits(GPIOC, GPIO_Pin_13 );	 // 关闭所有LED
            vTaskDelay(1000/portTICK_PERIOD_MS);
            GPIO_ResetBits(GPIOC, GPIO_Pin_13 );	 // 关闭所有LED
            vTaskDelay(1000/portTICK_PERIOD_MS);
        }
    }
    int main(void)
    {
            LED_GPIO_Config();
            xTaskCreate(led_task, "led_task", 1024, NULL, 20, &led_task_handle);
            // 开启调度
            vTaskStartScheduler();
            while(1);
    }
    ```

## 一、任务 Tasks

任务类似线程，分别对应TCB/PCB，实时 RTOS 调度器负责确保任务调入时的处理器上下文（寄存器值、堆栈内容等）与任务调出时的处理器上下文完全相同。为实现这一点，每个任务都分配有自己的堆栈。

- 优点：操作简单。没有使用限制。支持完全抢占式机制。完全按优先顺序排列。

- 缺点：每个任务都保留自己的堆栈，从而提高 RAM 使用率。
  
### 任务调度

<mark>FreeRTOS 默认使用固定优先级的抢占式调度策略，对同等优先级的任务执行时间片轮询调度

### 任务状态

- 就绪态：任务创建完成后，即进入就绪态，等待调度器调度。就绪任务指那些能够执行（它们不处于阻塞或挂起状态）， 但目前没有执行的任务， 因为同等或更高优先级的不同任务已经处于运行状态。

- 运行态：当任务实际执行时，它被称为处于运行状态。 任务当前正在使用处理器。 如果运行 RTOS 的处理器只有一个内核， 那么在任何给定时间内都只能有一个任务处于运行状态。

- 阻塞态：如果任务当前正在等待时间或外部事件，则该任务被认为处于阻塞状态。 例如，如果一个任务调用vTaskDelay()，它将被阻塞（被置于阻塞状态）， 直到延迟结束一个时间事件。
  
    任务也可以通过阻塞来等待队列、信号量、事件组、通知或信号量 事件。

- 挂起态：与“阻塞”状态下的任务一样， “挂起”状态下的任务不能 被选择进入运行状态，但处于挂起状态的任务 没有超时。 相反，任务只有在分别通过 vTaskSuspend() 和 xTaskResume() API 调用明确命令时 才会进入或退出挂起状态。

任务间转换API请查看开发者文档

### 任务优先级

每个任务均被分配了从 0 到 ( configMAX_PRIORITIES - 1 ) 的优先级，其中的 configMAX_PRIORITIES 在 FreeRTOSConfig.h 中定义。低优先级数字表示低优先级任务。 空闲任务的优先级为零 (tskIDLE_PRIORITY)。

任意数量的任务可共用相同的优先级。 如果 configUSE_TIME_SLICING 未经定义， 或者如果 configUSE_TIME_SLICING 设置为 1，则相同优先级的就绪状态任务 将使用时间切片轮询调度方案共享可用的处理时间

### 任务创建应用示例

- 定义任务函数句柄与任务函数
  
```c
//任务函数句柄
static TaskHandle_t Led0_Task_handle = NULL;
//任务实际上就是一个无限循环且不带返回值的 C 函数。
static void Led0_Task(void* parameter)
```

- 创建函数创建任务
  
```c
//xTaskCreate()：创建任务，所需的 RAM 将自动 从 FreeRTOS 堆中分配。
//xTaskCreateStatic()：则 RAM 由应用程序编写者提供，因此可以在编译时进行静态分配。

BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
xReturn = xTaskCreate((TaskFunction_t )Led0_Task, /* 任务入口函数 */
                    (const char*    )"Led0_Task",/* 任务名字 */
                    (uint16_t       )512,   /* 任务栈大小 */
                    (void*          )NULL,/* 任务入口函数参数 */
                    (UBaseType_t    )2,/* 任务的优先级 */
                    (TaskHandle_t*  )&Led0_Task_handle);/* 任务控制块指针 */
                    
if(NULL != xReturn) 
  printf("Create Led0_Task Succeed!\r\n");
```

- 启动任务调度器
  
```c
// 启动任务，开启调度 
vTaskStartScheduler();   
```

## 二、任务管理

### 任务管理API

```c
//任务管理函数，传入任务句柄 TaskHandle_t xTaskToSuspend 即可，一般NULL空句柄指自身任务，任务管理函数没有返回值

//任务挂起函数
vTaskSuspend( TaskHandle_t xTaskToResume )

//任务恢复函数
vTaskResume( TaskHandle_t xTaskToResume )

//任务删除函数
vTaskDelete( TaskHandle_t xTaskToResume )

//任务延时函数
vTaskDelay( const TickType_t xTicksToDelay )

//所有任务挂起函数
vTaskSuspendAll()
```

```c
//vTaskDelay()在我们任务中用得非常之多。要想使用 FreeRTOS 中的 vTaskDelay() 函数必须在 FreeRTOSConfig.h 中把 INCLUDE_vTaskDelay 定义为 1 来使能。

void vTaskDelay(const TickType_t xTicksToDelay);
const TickType_t xDelay = 500 / portTICK_PERIOD_MS;/* Block for 500ms. */
```

### 任务管理应用示例

```c
static TaskHandle_t LED_Task_Handle = NULL;
static TaskHandle_t KEY_Task_Handle = NULL;

static void LED_Task(void* parameter)
{
  while (1)
  {
    LED1_ON;
    printf("LED_Task Running,LED1_ON\r\n");
    vTaskDelay(500);   /* 延时500个tick */
    
    LED1_OFF;     
    printf("LED_Task Running,LED1_OFF\r\n");
    vTaskDelay(500);   /* 延时500个tick */
  }
}


static void KEY_Task(void* parameter)
{
  while (1)
  {
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
    {/* K1 被按下 */
      printf("挂起LED任务！\n");
      vTaskSuspend(LED_Task_Handle);/* 挂起LED任务 */
      printf("挂起LED任务成功！\n");
    } 
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
    {/* K2 被按下 */
      printf("恢复LED任务！\n");
      vTaskResume(LED_Task_Handle);/* 恢复LED任务！ */
      printf("恢复LED任务成功！\n");
    }
    vTaskDelay(20);/* 延时20个tick */
  }
}

int main(void)
{
/* 创建LED_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )LED_Task, /* 任务入口函数 */
                        (const char*    )"LED_Task",/* 任务名字 */
                        (uint16_t       )512,/* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )2,/* 任务的优先级 */
                        (TaskHandle_t*  )&LED_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建LED_Task任务成功!\r\n");

  /* 创建KEY_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )KEY_Task,  /* 任务入口函数 */
                        (const char*    )"KEY_Task",/* 任务名字 */
                        (uint16_t       )512,  /* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )3, /* 任务的优先级 */
                        (TaskHandle_t*  )&KEY_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建KEY_Task任务成功!\r\n");

    vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

## 三、消息队列、队列 Queues

队列是任务间通信的主要形式。 它们可以用于在任务之间以及中断和任务之间发送消息。

队列是为了任务与任务、任务与中断之间的通信而准备的，可以在任务与任务、任务与中断之间传递消息。

`FreeRTOS 中的信号量的也是依据队列实现的！`

### 阻塞队列

队列 API 函数允许指定阻塞时间。如果同一个队列上有多个处于阻塞状态的任务， 那么具有最高优先级的任务将最先解除阻塞。

- 当一个任务试图从一个空队列中读取时，该队列将进入阻塞状态（因此它不会消耗任何 CPU 时间，且其他任务可以运行） 直到队列中的数据变得可用，或者阻塞时间过期。

- 当一个任务试图写入到一个满队列时，该队列将进入阻塞状态（因此它不会消耗任何 CPU 时间，且其他任务可以运行） 直到队列中出现可用空间，或者阻塞时间过期。

### 队列的数据存储方式

通常队列采用FIFO(First in First out)先进先出的存储缓冲机制，也就是往队列发送数据的时候(也叫入队)永远都是发送到队列的尾部，而从队列提取数据的时候(也叫出队)是从队列的头部提取的。但是 也可以使用 LIFO 的存储缓冲，也就是后进先出，FreeRTOS 中的队列也提供了 LIFO 的存储缓 冲机制。 

<mark>数据发送到队列中会导致数据拷贝，也就是将要发送的数据拷贝到队列中，这就意味着在 队列中存储的是数据的原始值，而不是原数据的引用(即只传递数据的指针)，这个也叫做值传递。UCOS 的消息队列采用的是引用传递，传递的是消息指针。

FreeRTOS 中使用队列传递消息的话虽然使用的是数据拷贝，但是也可以使用引用来传递消息啊，直接往队列中发送指向这个消息的地址指针不就可以了！这样当我要发送的消息数据太大的时候就 可以直接发送消息缓冲区的地址指针，比如在网络应用环境中，网络的数据量往往都很大的， 采用数据拷贝的话就不现实。

### 消息队列API

```c
//如果使用 xQueueCreate() 创建队列，则所需的 RAM 将自动 从 FreeRTOS 堆中分配。 
//如果使用 xQueueCreateStatic() 创建队列，则 RAM 由应用程序编写者提供，这会产生更多的参数， 但这样能够在编译时静态分配 RAM 。 

//创建队列
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength,
                             UBaseType_t uxItemSize );

///读队列   
BaseType_t xQueueReceive(
                              QueueHandle_t xQueue,
                              void *pvBuffer,
                              TickType_t xTicksToWait
                          );

//写队列
BaseType_t xQueueSend(
                            QueueHandle_t xQueue,
                            const void * pvItemToQueue,
                            TickType_t xTicksToWait
                         );

//删除队列
void vQueueDelete( QueueHandle_t xQueue );

//复位队列
BaseType_t xQueueReset( QueueHandle_t xQueue );

//返回队列中可用数据的个数
UBaseType_t uxQueueMessagesWaiting( const QueueHandle_t xQueue );
 
//返回队列中可用空间的个数
UBaseType_t uxQueueSpacesAvailable( const QueueHandle_t xQueue );
```

### 消息队列应用示例

```c
QueueHandle_t Test_Queue =NULL;

static TaskHandle_t Receive_Task_Handle = NULL;/* 读消息队列任务句柄 */
static TaskHandle_t Send_Task_Handle = NULL;/* 写消息队列任务句柄 */

static void Receive_Task(void* parameter)
{
  BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdTRUE */
  uint32_t r_queue;/* 定义一个接收消息的变量 */
  while (1)
  {
    xReturn = xQueueReceive( Test_Queue,    /* 消息队列的句柄 */
                            &r_queue,      /* 发送的消息内容 */
                            portMAX_DELAY); /* 等待时间 一直等 */
    if(pdTRUE == xReturn)
      printf("本次接收到的数据是%d\n\n",r_queue);
    else
      printf("数据接收出错,错误代码0x%lx\n",xReturn);
  }
}

static void Send_Task(void* parameter)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  uint32_t send_data1 = 1;
  uint32_t send_data2 = 2;
  while (1)
  {
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
    {/* K1 被按下 */
      printf("发送消息send_data1！\n");
      xReturn = xQueueSend( Test_Queue, /* 消息队列的句柄 */
                            &send_data1,/* 发送的消息内容 */
                            0 );        /* 等待时间 0 */
      if(pdPASS == xReturn)
        printf("消息send_data1发送成功!\n\n");
    } 
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
    {/* K2 被按下 */
      printf("发送消息send_data2！\n");
      xReturn = xQueueSend( Test_Queue, /* 消息队列的句柄 */
                            &send_data2,/* 发送的消息内容 */
                            0 );        /* 等待时间 0 */
      if(pdPASS == xReturn)
        printf("消息send_data2发送成功!\n\n");
    }
    vTaskDelay(20);/* 延时20个tick */
  }
}

int main(void)
{
/* 创建Test_Queue */
  Test_Queue = xQueueCreate((UBaseType_t ) 4,/* 消息队列的长度 */
                            (UBaseType_t ) 4);/* 消息的大小 */
  if(NULL != Test_Queue)
    printf("创建Test_Queue消息队列成功!\r\n");

/* 创建Receive_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Receive_Task, /* 任务入口函数 */
                        (const char*    )"Receive_Task",/* 任务名字 */
                        (uint16_t       )512,/* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )2,/* 任务的优先级 */
                        (TaskHandle_t*  )&Receive_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建Receive_Task任务成功!\r\n");

/* 创建Send_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Send_Task,  /* 任务入口函数 */
                        (const char*    )"Send_Task",/* 任务名字 */
                        (uint16_t       )512,  /* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )3, /* 任务的优先级 */
                        (TaskHandle_t*  )&Send_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建Send_Task任务成功!\n\n");

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

## 四、信号量 Semaphores

### 二值信号量 Binary Semaphores

<mark>在许多情况下， “任务通知”可以提供二进制信号量的轻量级替代方案

与互斥量的区别：二值信号量和互斥信号量（以下使用互斥量表示互斥信号量）非常相似，但是有一些细微差别：互斥量有优先级继承机制，二值信号量则没有这个机制。这使得二值信号量更偏 向应用于同步功能（任务与任务间的同步或任务和中断间同步），而互斥量更偏向应用于 临界资源的访问。

运行机制：（任务/中断）——释放——>（二值信号量）——获取——>（任务）

```c
//创建二值信号量
SemaphoreHandle_t xSemaphoreCreateBinary( void );
```

### 计数信号量 Counting Semaphores

计数信号量通常用于两种情况：

1. 盘点事件。
在此使用方案中，每次事件发生时，事件处理程序将“给出”一个信号量（信号量计数值递增） ，并且 处理程序任务每次处理事件（信号量计数值递减）时“获取”一个信号量。因此，计数值是 已发生的事件数与已处理的事件数之间的差值。在这种情况下， 创建信号量时计数值可以为零。

2. 资源管理。
在此使用情景中，计数值表示可用资源的数量。要获得对资源的控制权，任务必须首先获取 一个信号量——同时递减信号量计数值。当计数值达到零时，表示没有空闲资源可用。当任务使用完资源时， “返还”一个信号量——同时递增信号量计数值。在这种情况下， 创建信号量时计数值可以等于最大计数值。

```c
//创建计数信号量
SemaphoreHandle_t xSemaphoreCreateCounting( UBaseType_t uxMaxCount,
                                            UBaseType_t uxInitialCount);
```

### 互斥信号量 Mutexes

互斥锁是包含优先级继承机制的二进制信号量。 二进制信号量能更好实现实现同步（任务间或任务与中断之间）， 而互斥锁有助于更好实现简单互斥（即相互排斥）。

优先级继承算法是指：<mark>暂时提高某个占有某种资源的低优先级任务的优先级，使之与在所有等待该资源的任务中优先级最高那个任务的优先级相等，而当这个低优先级任务执行完毕释放该资源时，优先级重新回到初始设定值。因此，继承优先级的任务避免了系统资源被任何中 间优先级的任务抢占。（例如：低优先级持有资源，高优先级任务等待资源释放进入阻塞态，中途出现中等优先级唤醒低优先级释放资源，抢夺了高优先级任务的位置）

用于互斥时， 互斥锁就像用于保护资源的令牌。 任务希望访问资源时，必须首先 获取 ('take') 令牌。 使用资源后，必须“返回”令牌，这样其他任务就有机会访问 相同的资源。

不应在中断中使用互斥锁，因为：

- 互斥锁使用的优先级继承机制要求 从任务中（而不是从中断中）拿走和放入互斥锁。
- 中断无法保持阻塞来等待一个被互斥锁保护的资源 由互斥锁保护的资源变为可用

```C
//创建互斥量
SemaphoreHandle_t xSemaphoreCreateMutex( void )
```

### 递归互斥信号量

用户可对一把递归互斥锁重复加锁。只有用户为每个成功的 xSemaphoreTakeRecursive() 请求调用 xSemaphoreGiveRecursive() 后，互斥锁才会重新变为可用。例如，如果一个任务成功“加锁”相同的互斥锁 5 次， 那么任何其他任务都无法使用此互斥锁，直到任务也把这个互斥锁“解锁”5 次。

```c
//互斥量递归获取函数
xSemaphoreTakeRecursive( SemaphoreHandle_t xMutex,
                         TickType_t xTicksToWait );

//互斥量递归释放函数
xSemaphoreGiveRecursive( SemaphoreHandle_t xMutex )
```

### 信号量API

```c
//删除信号量
//包括互斥锁型信号量和递归信号量。请勿删除已有阻塞任务的信号量
void vSemaphoreDelete( SemaphoreHandle_t xSemaphore );

//释放信号量（任务）
xSemaphoreGive( SemaphoreHandle_t xSemaphore );
//释放信号量（中断）
xSemaphoreGiveFromISR/*用于释放一个信号量，带中断保护。被释放的信号量可以是二进制信号量和计数信号量。和普通版本的释放信号量 API 函数有些许不同，它不能释放互斥量，这是因为互斥量 不可以在中断中使用，互斥量的优先级继承机制只能在任务中起作用，而在中断中毫无意义。*/
      (
        SemaphoreHandle_t xSemaphore,
        signed BaseType_t *pxHigherPriorityTaskWoken
      )


//获取信号量（任务）
xSemaphoreTake( SemaphoreHandle_t xSemaphore,
                TickType_t xTicksToWait );
//获取信号量（中断）
xSemaphoreTakeFromISR
      (
        SemaphoreHandle_t xSemaphore,
        signed BaseType_t *pxHigherPriorityTaskWoken
      )

//如果信号量是计数信号量，则返回信号量的当前计数值 。 如果信号量是二进制信号量， 则当信号量可用时，返回 1，当信号量不可用时， 返回 0。
UBaseType_t uxSemaphoreGetCount( SemaphoreHandle_t xSemaphore );
```

### 应用示例

#### 信号量同步示例

```C
SemaphoreHandle_t BinarySem_Handle =NULL;

static TaskHandle_t Receive_Task_Handle = NULL;
static TaskHandle_t Send_Task_Handle = NULL;

static void Receive_Task(void* parameter)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  while (1)
  {
    //获取二值信号量 xSemaphore,没获取到则一直等待
    xReturn = xSemaphoreTake(BinarySem_Handle,
    //二值信号量句柄   xSemaphoreTake获取一个信号量，可以是二值信号量、计数信号量、互斥量。
                              portMAX_DELAY); //等待时间 
    if(pdTRUE == xReturn)
      printf("BinarySem_Handle Receive BinarySem Succeed\n\n");
    LED1_TOGGLE;
  }
}

static void Send_Task(void* parameter)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  while (1)
  {
    /* K1 被按下 */
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
    {
      //给出二值信号量 xSemaphoreGive 释放信号量
      xReturn = xSemaphoreGive( BinarySem_Handle );
      if( xReturn == pdTRUE )
        printf("BinarySem_Handle Send BinarySem Succeed\r\n");
      else
        printf("BinarySem_Handle Send BinarySem Faild\r\n");
    } 
    vTaskDelay(20);
  }
}

int main(void)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区

  /* 创建 BinarySem */
  BinarySem_Handle = xSemaphoreCreateBinary();    /*xSemaphoreCreateBinary  创建信号量*/
  if(NULL != BinarySem_Handle)
    printf("BinarySem_Handle Create BinarySem Succeed\r\n");

  /* 创建Receive_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Receive_Task, /* 任务入口函数 */
                        (const char*    )"Receive_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Receive_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("Create Receive_Task Succeed\r\n");

  /* 创建Send_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Send_Task,  /* 任务入口函数 */
                        (const char*    )"Send_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )3,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Send_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("Create Send_Task Succeed\n\n");

  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}

```

#### 计数信号量示例

```C
SemaphoreHandle_t CountSem_Handle =NULL;

static TaskHandle_t Take_Task_Handle = NULL;
static TaskHandle_t Give_Task_Handle = NULL;

static void Take_Task(void* parameter)
{
  BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
  /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    //如果KEY1被单击
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )       
    {
      /* 获取一个计数信号量 */
      xReturn = xSemaphoreTake(CountSem_Handle,/* 计数信号量句柄 */
                              0);/* 等待时间：0 */
      if ( pdTRUE == xReturn ) 
        printf( "KEY1被按下，成功申请到停车位。\n" );
      else
        printf( "KEY1被按下，不好意思，现在停车场已满！\n" );
    }
    vTaskDelay(20);     //每20ms扫描一次
  }
}

static void Give_Task(void* parameter)
{
  BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
  /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    //如果KEY2被单击
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )       
    {
      /* 获取一个计数信号量 */
      xReturn = xSemaphoreGive(CountSem_Handle);//给出计数信号量                  
      if ( pdTRUE == xReturn ) 
        printf( "KEY2被按下，释放1个停车位。\n" );
      else
        printf( "KEY2被按下，但已无车位可以释放！\n" );
    }
    vTaskDelay(20);     //每20ms扫描一次
  }
}

int main(void)
{
   BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区
  
  /* 创建Test_Queue */
  CountSem_Handle = xSemaphoreCreateCounting(10,10);
  if(NULL != CountSem_Handle)
    printf("CountSem_Handle计数信号量创建成功!\r\n");
 
  /* 创建Take_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Take_Task, /* 任务入口函数 */
                        (const char*    )"Take_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Take_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建Take_Task任务成功!\r\n");
  
  /* 创建Give_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Give_Task,  /* 任务入口函数 */
                        (const char*    )"Give_Task",/* 任务名字 */
                        (uint16_t       )512,  /* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )3, /* 任务的优先级 */
                        (TaskHandle_t*  )&Give_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建Give_Task任务成功!\n\n");
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

#### 互斥信号量示例

```c
/*优先级翻转实验：将本实验互斥量替换为二值信号量即可，实验现象可以观察到优先级翻转。
即高优先级任务等待低优先级释放信号量时，被中等优先级任务抢占运行了*/

//此互斥量实验是基于优先级翻转实验修改的，目的是为了测试互斥量的优先级继承机制是否有效
SemaphoreHandle_t MuxSem_Handle =NULL;

static TaskHandle_t LowPriority_Task_Handle = NULL;/* LowPriority_Task任务句柄 */
static TaskHandle_t MidPriority_Task_Handle = NULL;/* MidPriority_Task任务句柄 */
static TaskHandle_t HighPriority_Task_Handle = NULL;/* HighPriority_Task任务句柄 */


static void LowPriority_Task(void* parameter)
{
  static uint32_t i;
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  while (1)
  {
    printf("LowPriority_Task 获取互斥量\n");
    //获取互斥量 MuxSem,没获取到则一直等待
    xReturn = xSemaphoreTake(MuxSem_Handle,/* 互斥量句柄 */
                              portMAX_DELAY); /* 等待时间 */
    if(pdTRUE == xReturn)
    printf("LowPriority_Task Runing\n\n");
    
    for(i=0;i<2000000;i++)//模拟低优先级任务占用互斥量
    {
      taskYIELD();//发起任务调度
    }
    
    printf("LowPriority_Task 释放互斥量!\r\n");
    xReturn = xSemaphoreGive( MuxSem_Handle );//给出互斥量
      
    LED1_TOGGLE;
    
    vTaskDelay(1000);
  }
}

static void MidPriority_Task(void* parameter)
{ 
  while (1)
  {
   printf("MidPriority_Task Runing\n");
   vTaskDelay(1000);
  }
}

static void HighPriority_Task(void* parameter)
{
  BaseType_t xReturn = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
  while (1)
  {
    printf("HighPriority_Task 获取互斥量\n");
    //获取互斥量 MuxSem,没获取到则一直等待
    xReturn = xSemaphoreTake(MuxSem_Handle,/* 互斥量句柄 */
                              portMAX_DELAY); /* 等待时间 */
    if(pdTRUE == xReturn)
      printf("HighPriority_Task Runing\n");
    LED1_TOGGLE;
    
    printf("HighPriority_Task 释放互斥量!\r\n");
    xReturn = xSemaphoreGive( MuxSem_Handle );//给出互斥量


    vTaskDelay(1000);
  }
}

int main(void)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区
  
  /* 创建MuxSem */
  MuxSem_Handle = xSemaphoreCreateMutex();
  if(NULL != MuxSem_Handle)
    printf("MuxSem_Handle互斥量创建成功!\r\n");
 
  xReturn = xSemaphoreGive( MuxSem_Handle );//给出互斥量
//  if( xReturn == pdTRUE )
//    printf("释放信号量!\r\n");
    
  /* 创建LowPriority_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )LowPriority_Task, /* 任务入口函数 */
                        (const char*    )"LowPriority_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,     /* 任务的优先级 */
                        (TaskHandle_t*  )&LowPriority_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建LowPriority_Task任务成功!\r\n");
  
  /* 创建MidPriority_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )MidPriority_Task,  /* 任务入口函数 */
                        (const char*    )"MidPriority_Task",/* 任务名字 */
                        (uint16_t       )512,  /* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )3, /* 任务的优先级 */
                        (TaskHandle_t*  )&MidPriority_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建MidPriority_Task任务成功!\n");
  
  /* 创建HighPriority_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )HighPriority_Task,  /* 任务入口函数 */
                        (const char*    )"HighPriority_Task",/* 任务名字 */
                        (uint16_t       )512,  /* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )4, /* 任务的优先级 */
                        (TaskHandle_t*  )&HighPriority_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建HighPriority_Task任务成功!\n\n");
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

## 五、事件组 Event Groups

<mark>在许多情况下，“任务通知”可以提供事件组的轻量级替代方案

```c
若是在裸机编程中，用全局变量是最为有效的方法，这点我不否认，但是在操作系统中，使用全局变量就要考虑以下问题了：
如何对全局变量进行保护呢，如何处理多任务同时对它进行访问？
如何让内核对事件进行有效管理呢？使用全局变量的话，就需要在任务中轮询查 看事件是否发送，这简直就是在浪费 CPU 资源啊，还有等待超时机制，使用全局 变量的话需要用户自己去实现。
所以，在操作系统中，还是使用操作系统给我们提供的通信机制就好了，简单方便还 实用。
```

### 事件位（事件标志）

事件位用于指示事件 是否发生。 事件位通常称为事件标志。

- 例如，应用程序可以：
  
    定义一个位（或标志）， 设置为 1 时表示“已收到消息并准备好处理”， 设置为 0 时表示“没有消息等待处理”。

    定义一个位（或标志）， 设置为 1 时表示“应用程序已将准备发送到网络的消息排队”， 设置为 0 时表示 “没有消息需要排队准备发送到网络”。

    定义一个位（或标志）， 设置为 1 时表示“需要向网络发送心跳消息”， 设置为 0 时表示“不需要向网络发送心跳消息”。

### 事件组

事件组就是一组事件位。 事件组中的事件位 通过位编号来引用。

- 同样，以上面列出的三个例子为例：
  
    事件标志组位编号 为 0 表示“已收到消息并准备好处理”。

    事件标志组位编号 为 1 表示“应用程序已将准备发送到网络的消息排队”。

    事件标志组位编号 为 2 表示“需要向网络发送心跳消息”。

### 事件标志组和事件位的数据类型

事件标志组的数据类型为 EventGroupHandle_t，当 configUSE_16_BIT_TICKS 为 1 的时候 事件标志组可以存储 8 个事件位，当 configUSE_16_BIT_TICKS 为 0 的时候事件标志组存储 24 个事件位。

```c
//portmacro.h
#if( configUSE_16_BIT_TICKS == 1 )
   typedef uint16_t TickType_t;
#define portMAX_DELAY ( TickType_t ) 0xffff 
#else
 typedef uint32_t TickType_t;
 #define portMAX_DELAY ( TickType_t ) 0xffffffffUL
 #define portTICK_TYPE_IS_ATOMIC 1
 #endif
```

当 configUSE_16_BIT_TICKS 为 1 的时候 TickType_t 是个 16 位的数据类型，因此 EventBits_t 也是个 16 位的数据类型。<mark>EventBits_t 类型的变量可以存储 8（16-8） 个事件位，高8位其他用途占用。

当 configUSE_16_BIT_TICKS 为 0 的时候 TickType_t 是个 32 位的数据类型，因此 EventBits_t 也是个 32 位的数据类型。<mark>EventBits_t 类型的变量可以存储 24（32-8） 个事件位，高8位其他用途占用。

事件位 0 存放在这个变量的 bit0 上，变量的 bit1 就是事件位 1，以此类推。 对于 STM32 来说一个事件标志组最多可以存储 24 个事件位，如图1所示

### 事件组API

```C
//事件创建
EventGroupHandle_t xEventGroupCreate( void );

//事件删除
void vEventGroupDelete( EventGroupHandle_t xEventGroup )

//事件组置位
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup,
                                const EventBits_t uxBitsToSet );

//等待事件
EventBits_t xEventGroupWaitBits(
                      const EventGroupHandle_t xEventGroup,
                      const EventBits_t uxBitsToWaitFor,
                      const BaseType_t xClearOnExit,
                      const BaseType_t xWaitForAllBits,
                      TickType_t xTicksToWait );
                
//事件组清除位。 无法从中断调用此函数
EventBits_t xEventGroupClearBits(
                                 EventGroupHandle_t xEventGroup,
                                 const EventBits_t uxBitsToClear );

//可以从中断调用的 xEventGroupClearBits() 版本                     
BaseType_t xEventGroupClearBitsFromISR(
                              EventGroupHandle_t xEventGroup,
                              const EventBits_t uxBitsToClear );
```

### 事件组应用示例

```C
//创建了两个任务，一个是设置事件任务，一个是等待事件任务，两个任务独立运行，设置事件任务通过检测按键的按下情况设置不同的事件标志位，等待事件任务则获取这两个事件标志位，并且判断两个事件是否都发生，如果是则输出相应信息，LED 进行翻转。
static EventGroupHandle_t Event_Handle =NULL;
static TaskHandle_t LED_Task_Handle = NULL;
static TaskHandle_t KEY_Task_Handle = NULL;

#define KEY1_EVENT  (0x01 << 0)//设置事件掩码的位0
#define KEY2_EVENT  (0x01 << 1)//设置事件掩码的位1

static void LED_Task(void* parameter)
{
  EventBits_t r_event;  /* 定义一个事件接收变量 */
  /* 任务都是一个无限循环，不能返回 */
  while (1)
  {

    //xClearOnExit设置为pdTRUE，那么在xEventGroupWaitBits()返回之前，如果满足等待条件（如果函数返回的原因不是超时），那么在事件组中设置的uxBitsToWaitFor中的任何位都将被清除。 
    //xClearOnExit设置为pdFALSE，则在调用xEventGroupWaitBits()时，不会更改事件组中设置的位。

    //xWaitForAllBits如果xWaitForAllBits设置为pdTRUE，类似条件与。当uxBitsToWaitFor中的所有位都设置或指定的块时间到期时，xEventGroupWaitBits()才返回。
    //xWaitForAllBits设置为pdFALSE，类似条件或当设置uxBitsToWaitFor中设置的任何一个位置1 或指定的块时间到期时，xEventGroupWaitBits()都会返回。 

    //阻塞时间由xTicksToWait参数指定。          
    r_event = xEventGroupWaitBits(Event_Handle,  /* 事件对象句柄 */
                                  KEY1_EVENT|KEY2_EVENT,/* 接收线程感兴趣的事件 */
                                  pdTRUE,   /* 退出时清除事件位 */
                                  pdTRUE,   /* 满足感兴趣的所有事件 */
                                  portMAX_DELAY);/* 指定超时事件,一直等 */
                        
    if((r_event & (KEY1_EVENT|KEY2_EVENT)) == (KEY1_EVENT|KEY2_EVENT)) 
    {
      /* 如果接收完成并且正确 */
      printf ( "KEY1与KEY2都按下\n");
      LED1_TOGGLE;       //LED1反转
    }
    else
      printf ( "事件错误！\n");
  }
}


static void KEY_Task(void* parameter)
{
  /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
    {
      printf ( "KEY1被按下\n" );
      /* 触发一个事件1 */
      xEventGroupSetBits(Event_Handle,KEY1_EVENT);
    }
    
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )       //如果KEY2被单击
    {
      printf ( "KEY2被按下\n" );
      /* 触发一个事件2 */
      xEventGroupSetBits(Event_Handle,KEY2_EVENT);
    }
    vTaskDelay(20);     //每20ms扫描一次
  }
}

int main(void)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区
  
  /* 创建 Event_Handle */
  Event_Handle = xEventGroupCreate();
  if(NULL != Event_Handle)
    printf("Event_Handle 事件创建成功!\r\n");
    
  /* 创建LED_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )LED_Task, /* 任务入口函数 */
                        (const char*    )"LED_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,     /* 任务的优先级 */
                        (TaskHandle_t*  )&LED_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建LED_Task任务成功!\r\n");
  
  /* 创建KEY_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )KEY_Task,  /* 任务入口函数 */
                        (const char*    )"KEY_Task",/* 任务名字 */
                        (uint16_t       )512,  /* 任务栈大小 */
                        (void*          )NULL,/* 任务入口函数参数 */
                        (UBaseType_t    )3, /* 任务的优先级 */
                        (TaskHandle_t*  )&KEY_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建KEY_Task任务成功!\n");
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

## 六、任务通知 

<mark> 自 FreeRTOS V8.2.0  起可用  自 V10.4.0 起支持单任务多条通知

每个任务都有一个 32 位的通知 值，在大多数情况下，任务通知可以替代二值信号量、计数信号量、事件组，也可以替代长度为 1 的队列（可以保存一个32位整数或指针值）。

相对于以前使用 FreeRTOS 内核通信的资源，必须创建队列、二进制信号量、计数信 号量或事件组的情况，使用任务通知显然更灵活。

使用任务 通知比通过信号量等 ICP 通信方式解除阻塞的任务要快 45%，并且更加省 RAM 内存空间 （使用 GCC 编译器，-o2 优化级别），任务通知的使用无需创建队列。想要使用任务通知， 必须将 FreeRTOSConfig.h 中的宏定义 configUSE_TASK_NOTIFICATIONS 设置为 1，其实 FreeRTOS 默认是为 1 的，所以任务通知是默认使能的。

- FreeRTOS 提供以下几种方式发送通知给任务 ：

    发送通知给任务，如果有通知未读，不覆盖通知值。

    发送通知给任务，直接覆盖通知值。

    发送通知给任务，设置通知值的一个或者多个位，可以当做事件组来使用。

    发送通知给任务，递增通知值，可以当做计数信号量使用。 通过对以上任务通知方式的合理使用，可以在一定场合下替代 FreeRTOS 的信号量， 队列、事件组等。

当然，凡是都有利弊，不然的话 FreeRTOS 还要内核的 IPC 通信机制干嘛，消息通知虽然处理更快，RAM 开销更小，但也有以下限制 ：

- 只能有一个任务接收通知消息，因为必须指定接收通知的任务。

- 只有等待通知的任务可以被阻塞，发送通知的任务，在任何情况下都不会因为发送失败而进入阻塞态。

### 任务通知API

```c
/*xTaskNotifyGive()是一个宏，宏展开是调用函数 xTaskNotify( ( xTaskToNotify ), ( 0 ), eIncrement )，即向一个任务发送通知，并将对方的任务通知值加 1。该函数可以作为二值信号量和计数信号量的一种轻量型的实现，速度更快，
在这种情况下对象任务在等待任务通知的时候应该是使用函数 ulTaskNotifyTake() 而不是 xTaskNotifyWait() 。*/
BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify );

//可在中断服务程序 (ISR) 中使用的 xTaskNotifyGive() 版本。
void vTaskNotifyGiveFromISR( TaskHandle_t xTaskToNotify,
                            BaseType_t *pxHigherPriorityTaskWoken );

//等待任务通知 ulTaskNotifyTake()作为二值信号量和计数信号量的一种轻量级实现，速度更快。如果 FreeRTOS 中使用函数 xSemaphoreTake() 来获取信号量，这个时候则可以试试使用函数 ulTaskNotifyTake()来代替。 
uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit,
                            TickType_t xTicksToWait );





/* FreeRTOS 每个任务都有一个 32 位的变量用于实现任务通知，在任务创建的时候初始化为 0。这个 32 位的通知值在任务控制块 TCB 里面定义

xTaskNotify()用于在任务中直接向另外一个任务发送一个事件，接收到该任务通知的任务有可能解锁。
xTaskNotify()函数在发送任务 通知的时候会指定一个通知值，并且用户可以指定通知值发送的方式。

如果你想使用任务通知来实现二值信号量和计数信号量，那么应该使用更加简单的函数 xTaskNotifyGive() ，而不是使用 xTaskNotify()。*/
BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify,
                         uint32_t ulValue,
                         eNotifyAction eAction );

//可在中断服务程序 (ISR) 中使用的 xTaskNotify() 版本。
BaseType_t xTaskNotifyFromISR( TaskHandle_t xTaskToNotify,
                                uint32_t ulValue,
                                eNotifyAction eAction,
                                BaseType_t *pxHigherPriorityTaskWoken );

//xTaskNotifyAndQuery()与 xTaskNotify()很像，不同的是多了一个附加的参数 pulPreviousNotifyValue 用于回传接收任务的上一个通知值
BaseType_t xTaskNotifyAndQuery( TaskHandle_t xTaskToNotify,
                                 uint32_t ulValue,
                                 eNotifyAction eAction,
                                 uint32_t *pulPreviousNotifyValue );

//可在中断服务程序 (ISR) 中使用的 xTaskNotifyAndQuery() 版本。
BaseType_t xTaskNotifyAndQueryFromISR( 
                      TaskHandle_t xTaskToNotify,
                      uint32_t ulValue,
                      eNotifyAction eAction,
                      uint32_t *pulPreviousNotifyValue,
                      BaseType_t *pxHigherPriorityTaskWoken );

//用于实现全功能版的等待任务通知，根据用户指定的参数的不同，可以灵活的用于实现轻量级的消息队列队列、二值信号量、计数信号量和事件组功能，并带有超时等待。
BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry,
                            uint32_t ulBitsToClearOnExit,
                            uint32_t *pulNotificationValue,
                            TickType_t xTicksToWait );
```

### 任务通知应用示例

#### 替代二值信号量实验

```c
static TaskHandle_t Receive1_Task_Handle = NULL;
static TaskHandle_t Receive2_Task_Handle = NULL;
static TaskHandle_t Send_Task_Handle = NULL;


static void Receive1_Task(void* parameter)
{
  while (1)
  {

    /* xClearCountOnExit：
     * pdTRUE 在退出函数的时候任务任务通知值清零，类似二值信号量
     * pdFALSE 在退出函数ulTaskNotifyTakeO的时候任务通知值减一，类似计数型信号量。*/
    //获取任务通知 ,没获取到则一直等待
    ulTaskNotifyTake(pdTRUE,portMAX_DELAY);
    
    printf("Receive1_Task 任务通知获取成功!\n\n");
    
    LED1_TOGGLE;
  }
}

static void Receive2_Task(void* parameter)
{
  while (1)
  {
    //获取任务通知 ,没获取到则一直等待
    ulTaskNotifyTake(pdTRUE,portMAX_DELAY);
    
    printf("Receive2_Task 任务通知获取成功!\n\n");
    
    LED2_TOGGLE;
  }
}

static void Send_Task(void* parameter)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  while (1)
  {
    /* KEY1 被按下 */
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
    {
      /* 原型:BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify ); */
      xReturn = xTaskNotifyGive(Receive1_Task_Handle);
      /* 此函数只会返回pdPASS */
      if( xReturn == pdTRUE )
        printf("Receive1_Task_Handle 任务通知发送成功!\r\n");
    } 
    /* KEY2 被按下 */
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
    {
      xReturn = xTaskNotifyGive(Receive2_Task_Handle);
      /* 此函数只会返回pdPASS */
      if( xReturn == pdPASS )
        printf("Receive2_Task_Handle 任务通知发送成功!\r\n");
    }
    vTaskDelay(20);
  }
}

int main(void)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区
 
  /* 创建Receive1_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Receive1_Task, /* 任务入口函数 */
                        (const char*    )"Receive1_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Receive1_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建Receive1_Task任务成功!\r\n");
  
  /* 创建Receive2_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Receive2_Task, /* 任务入口函数 */
                        (const char*    )"Receive2_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )3,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Receive2_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建Receive2_Task任务成功!\r\n");
  
  /* 创建Send_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Send_Task,  /* 任务入口函数 */
                        (const char*    )"Send_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )4,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Send_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建Send_Task任务成功!\n\n");
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

#### 替代计数信号量实验

```c
static TaskHandle_t Take_Task_Handle = NULL;
static TaskHandle_t Give_Task_Handle = NULL;


static void Take_Task(void* parameter)
{
  uint32_t take_num = pdTRUE;/* 定义一个创建信息返回值，默认为pdPASS */
  /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    //如果KEY1被单击
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )       
    {
      //获取任务通知 ,没获取到则不等待
      take_num=ulTaskNotifyTake(pdFALSE,0);//
      if(take_num > 0)
        printf( "KEY1被按下，成功申请到停车位，当前车位为 %d \n", take_num - 1);
      else
        printf( "KEY1被按下，车位已经没有了，请按KEY2释放车位\n" );  
    }
    vTaskDelay(20);     //每20ms扫描一次
  }
}

static void Give_Task(void* parameter)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  /* 任务都是一个无限循环，不能返回 */
  while (1)
  {
    //如果KEY2被单击
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )       
    {
      /* 原型:BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify ); */
      /* 释放一个任务通知 */
      xTaskNotifyGive(Take_Task_Handle);//发送任务通知
      /* 此函数只会返回pdPASS */
      if ( pdPASS == xReturn ) 
        printf( "KEY2被按下，释放1个停车位。\n" );
    }
    vTaskDelay(20);     //每20ms扫描一次
  }
}

int main(void)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区
 
  /* 创建Take_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Take_Task, /* 任务入口函数 */
                        (const char*    )"Take_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Take_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建Take_Task任务成功!\r\n");
  
  /* 创建Give_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Give_Task,  /* 任务入口函数 */
                        (const char*    )"Give_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )3,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Give_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建Give_Task任务成功!\n\n");
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

#### 替代消息队列实验

比较长，请跳转至CSDN博客查看

[FreeRTOS 任务通知替代](https://blog.csdn.net/qq_61672347/article/details/125651530?spm=1001.2014.3001.5502)

## 七、软件定时器

使用软件定时器时候要注意以下几点：

- 软件定时器的回调函数中应快进快出，绝对不允许使用任何可能引软件定时器起 任务挂起或者阻塞的 API 接口，在回调函数中也绝对不允许出现死循环。

- 软件定时器使用了系统的一个队列和一个任务资源，软件定时器任务的优先级默认为 configTIMER_TASK_PRIORITY，为了更好响应，该优先级应设置为所有任务中最高的优先级。

- 定时器任务的堆栈大小默认为 configTIMER_TASK_STACK_DEPTH 个字节。
  
### 硬件定时器与软件定时器

- 硬件定时器

  硬件定时器是芯片本身提供的定时功能。一般是由外部晶振提供给芯片输入时钟，芯片向软件模块提供一组配置寄存器，接受控制输入，到达设定时间值后芯片中断控制器产生时钟中断。硬件定时器的精度一般很高，可以达到纳秒级别，并且是中断触发方式。

  使用硬件定时器时，每次在定时时间到达之后就会自动触发一个中断，用户在中断中处理信息；

- 软件定时器

  软件定时器是由操作系统提供的一类系统接口，它构建在硬件定时器基础之上，使系统能够提供不受硬件定时器资源限制的定时器服务，它实现的功能与硬件定时器也是类似的。但是定时器精度以OS心跳为整数基准，所以不够高。

  使用软件定时器时，需要我们在创建软件定时器时指定时间到达后要调用的函数（也称超时函数/回调函数，为了统一，下文均用回调函数描述），在回调函数中处理信息。

### 软件定时器应用场景

软件定时器更适用于对时间精度要求不高的任务，一些辅助型的任务。

硬件定时器受硬件的限制，数量上不足以满足用户的实际需求，无法提供更多的定时器，那么可以采用软件定时器来完成，由软件定时器代替硬件定时器任务。

但需要注意的是软件定时器的精度是无法和硬件定时器相比，而且在软件定时器的定时过程中是极有可能被其它中断所打断，因为软件定时器的执行上下文环境是任务。

### 软件定时器的精度

在操作系统中，通常软件定时器以系统节拍周期为计时单位。系统节拍是系统的心跳节拍，表示系统时钟的频率，就类似人的心跳，1s 能跳动多少下，系统节拍配置为 configTICK_RATE_HZ，该宏在 FreeRTOSConfig.h 中有定义，默认是 1000。那么系统的时钟节拍周期就为 1ms（1s跳动 1000 下，每一下就为 1ms）。

软件定时器的所定时数值必须 是这个节拍周期的整数倍，例如节拍周期是 10ms，那么上层软件定时器定时数值只能是 10ms，20ms，100ms 等，而不能取值为 15ms。

由于节拍定义了系统中定时器能够分辨的精确度，系统可以根据实际系统 CPU 的处理能力和实时性需求设置合适的数值，系统节拍周期的值越小，精度越高，但是系统开销也将越大，因为这代表在 1 秒中系统进入时钟中断的次数也就越多。

### 软件定时器API

```c
//软件定时器创建函数
TimerHandle_t xTimerCreate
                 ( const char * const pcTimerName,
                   const TickType_t xTimerPeriod,
                   const UBaseType_t uxAutoReload,
                   void * const pvTimerID,
                   TimerCallbackFunction_t pxCallbackFunction );

//软件定时器启动函数
BaseType_t xTimerStart( TimerHandle_t xTimer,
                            TickType_t xBlockTime );
BaseType_t xTimerStartFromISR//中断版本ISR
               (
                  TimerHandle_t xTimer,
                  BaseType_t *pxHigherPriorityTaskWoken
               );

//软件定时器停止函数
BaseType_t xTimerStop( TimerHandle_t xTimer,
                           TickType_t xBlockTime );
BaseType_t xTimerStopFromISR//中断版本ISR
             (
                 TimerHandle_t xTimer,
                 BaseType_t *pxHigherPriorityTaskWoken
             );

//软件定时器删除函数
BaseType_t xTimerDelete( TimerHandle_t xTimer,
                             TickType_t xBlockTime );

```

### 软件定时器应用示例

```c
static TimerHandle_t Swtmr1_Handle =NULL; 
static TimerHandle_t Swtmr2_Handle =NULL;  

static uint32_t TmrCb_Count1 = 0; /* 记录软件定时器1回调函数执行次数 */
static uint32_t TmrCb_Count2 = 0; /* 记录软件定时器2回调函数执行次数 */

//软件定时器1 回调函数，打印回调函数信息&当前系统时间
//软件定时器请不要调用阻塞函数，也不要进行死循环，应快进快出
static void Swtmr1_Callback(void* parameter)
{
  TickType_t tick_num1;
 
  TmrCb_Count1++;                   /* 每回调一次加一 */
 
  tick_num1 = xTaskGetTickCount();  /* 获取滴答定时器的计数值 */
  
  LED1_TOGGLE;
  
  printf("Swtmr1_Callback函数执行 %d 次\n", TmrCb_Count1);
  printf("滴答定时器数值=%d\n", tick_num1);
}
//软件定时器2 回调函数，打印回调函数信息&当前系统时间
//软件定时器请不要调用阻塞函数，也不要进行死循环，应快进快出
static void Swtmr2_Callback(void* parameter)
{
  TickType_t tick_num2;
 
  TmrCb_Count2++;                   /* 每回调一次加一 */
 
  tick_num2 = xTaskGetTickCount();  /* 获取滴答定时器的计数值 */
 
  printf("Swtmr2_Callback函数执行 %d 次\n", TmrCb_Count2);
  printf("滴答定时器数值=%d\n", tick_num2);
}

int main(void)
{
  taskENTER_CRITICAL();           //进入临界区
    
   //单次定时器，周期(1000个时钟节拍)，周期模式
  Swtmr1_Handle=xTimerCreate((const char*   )"AutoReloadTimer",
                            (TickType_t     )1000,/* 定时器周期 1000(tick) */
                            (UBaseType_t    )pdTRUE,/* 周期模式 */
                            (void*          )1,/* 为每个计时器分配一个索引的唯一ID */
                            (TimerCallbackFunction_t)Swtmr1_Callback); 
  if(Swtmr1_Handle != NULL)                          
  {
    xTimerStart(Swtmr1_Handle,0);//开启周期定时器
  }                            

   //单次定时器，周期(5000个时钟节拍)，单次模式
  Swtmr2_Handle=xTimerCreate((const char*   )"OneShotTimer",
                             (TickType_t    )5000,/* 定时器周期 5000(tick) */
                             (UBaseType_t   )pdFALSE,/* 单次模式 */
                             (void*         )2,/* 为每个计时器分配一个索引的唯一ID */
                             (TimerCallbackFunction_t)Swtmr2_Callback); 
  if(Swtmr2_Handle != NULL)
  {
    xTimerStart(Swtmr2_Handle,0);//开启周期定时器
  } 
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

## 八、内存管理

FreeRTOS操作系统将内核与内存管理分开实现，操作系统内核仅规定了必要的内存管理函数原型，而不关心这些内存管理函数是如何实现的（src内核文件夹 port架构内存管理文件夹）

每次创建任务、队列、互斥锁、软件定时器、信号量或事件组时，RTOS 内核都需要 RAM ， RAM 可以 可以从 RTOS API 对象创建函数内的 RTOS 堆自动动态分配， 或者由应用程序编写者提供。

FreeRTOS\Source\portable\MemMang\ 目录下 有五种内存管理方案(分别是 heap_1.c、 heap_2.c、heap_3.c、heap_4.c 和 heap_5.c)

- 五种内存管理方案：
  
  heap_1 —— 最简单，不允许释放内存。

  heap_2—— 允许释放内存，但不会合并相邻的空闲块。

  heap_3 —— 简单包装了标准 malloc() 和 free()，以保证线程安全。

  heap_4 —— 合并相邻的空闲块以避免碎片化。 包含绝对地址放置选项。

  heap_5 —— 如同 heap_4，能够跨越多个不相邻内存区域的堆。

  注意：

  heap_1 不太有用，因为 FreeRTOS 添加了静态分配支持。

  heap_2 现在被视为旧版，因为较新的 heap_4 实现是首选。

为什么不直接使用 C 标准库中的内存管理函数呢？在电脑中我们可以 用 malloc()和 free()这两个函数动态的分配内存和释放内存。但是，在嵌入式实时操作系统 中，调用 malloc()和 free()却是危险的，原因有以下几点：

1. 这些函数在小型嵌入式系统中并不总是可用的，小型嵌入式设备中的 RAM 不足。
2. 他们不是线程安全的。
3. 它们并不是确定的，每次调用这些函数执行的时间可能都不一样。

    实时操作系统中，对内存的分配时间要求更为苛刻，分配内存的时间必须是确定的。一般内存管理算法是根据需要存储的数据的长度在内存中去寻找一个与这段数据相适应的空闲内存块，然后将数据存储在里面。而寻找这样一个空闲内存块所耗费的时间是不确定的，因此对于实时系统来说，这就是不可接受的，实时系统必须要保证内 存块的分配过程在可预测的确定时间内完成，否则实时任务对外部事件的响应也将变得不可确定。

4. 它们有可能产生碎片。

    嵌入式系统中，内存是十分有限而且是十分珍贵的，用一块内存就少了一块内存， 而在分配中随着内存不断被分配和释放，整个系统内存区域会产生越来越多的碎片，因为在使用过程中，申请了一些内存，其中一些释放了，导致内存空间中存在一些小的内存块， 它们地址不连续，不能够作为一整块的大内存分配出去，所以一定会在某个时间，系统已经无法分配到合适的内存了，导致系统瘫痪。其实系统中实际是还有内存的，但是因为小块的内存的地址不连续，导致无法分配成功，所以我们需要一个优良的内存分配算法来避 免这种情况的出现。

5. 如果允许堆空间的生长方向覆盖其他变量占据的内存，它们会成为 debug 的灾难。

### 动态内存分配vs静态内存分配

在嵌入式程序设计中内存分配应该是根据所设计系统的特点来决定选择使用`动态内存分配`还是`静态内存分配`算法。

一些可靠性要求非常高的系统应选择使用静态的，而普通的业务系统可以使用动态来提高内存使用效率。静态可以保证设备的可靠性但是需要考虑内存上限，内存使用效率低，而动态则是相反。

### 内存管理应用场景

内存管理的主要工作是动态划分并管理用户分配好的内存区间，主要是在用户需要使用大小不等的内存块的场景中使用，当用户需要分配内存时，可以通过操作系统的内存申请函数索取指定大小内存块，一旦使用完毕，通过动态内存释放函数归还所占用内存。

- 例如我们需要定义一个 float 型数组：floatArr[];

  但是，在使用数组的时候，总有一个问题困扰着我们：数组应该有多大？在很多的情 况下，你并不能确定要使用多大的数组，可能为了避免发生错误你就需要把数组定义得足够大。即使你知道想利用的空间大小，但是如果因为某种特殊原因空间利用的大小有增加或者减少，你又必须重新去修改程序，扩大数组的存储范围。这种分配固定大小的内存分配方法称之为静态内存分配。这种内存分配的方法存在比较严重的缺陷，在大多数情况下会浪费大量的内存空间，在少数情况下，当你定义的数组不够大时，可能引起下标越界错误，甚至导致严重后果。

  我们用动态内存分配就可以解决上面的问题。所谓动态内存分配就是指在程序执行的过程中动态地分配或者回收存储空间的分配内存的方法。动态内存分配不象数组等静态内 存分配方法那样需要预先分配存储空间，而是由系统根据程序的需要即时分配，且分配的大小就是程序要求的大小。

### heap_4.c

heap_4.c 方案的空闲内存块也是以单链表的形式连接起来的，BlockLink_t 类型的局部静态变量 xStart 表示链表头，但 heap_4.c 内存管理方案的链表尾部则保存在内存堆空间最后位置，并使用 BlockLink_t 指针类型局部静态变量 pxEnd 指向这个区域

heap_4.c 内存管理方案的空闲块链表不是以内存块大小进行排序的，而是以内存块起始地址大小排序，内存地址小的在前，地址大的在后，因为 heap_4.c 方案还有一个内存合 并算法，在释放内存的时候，假如相邻的两个空闲内存块在地址上是连续的，那么就可以 合并为一个内存块，这也是为了适应合并算法而作的改变。

- heap_4.c 方案具有以下特点：

    1. 可用于重复删除任务、队列、信号量、互斥量等的应用程序
    2. 可用于分配和释放随机字节内存的应用程序，但并不像 heap2.c 那样产生严重的内 存碎片。
    3. 具有不确定性，但是效率比标准 C 库中的 malloc 函数高得多。

heap_4.c 对于想在应用程序代码中直接使用可移植层内存分配方案的应用程序特别有用 （而不是 通过调用 API 函数 pvPortMalloc() 和 vPortFree() 来间接使用）。

- 内存申请函数（请查看函数源码） pvPortMalloc()
- 内存释放函数（请查看函数源码） vPortFree()

### 内存申请测试示例

```C
static TaskHandle_t LED_Task_Handle = NULL;
static TaskHandle_t Test_Task_Handle = NULL;

uint8_t *Test_Ptr = NULL;

//LED_TASK仅表示处于运行状态
static void LED_Task(void* parameter)
{
  while (1)
  {
    LED1_TOGGLE;
    vTaskDelay(1000);/* 延时1000个tick */
  }
}


static void Test_Task(void* parameter)
{
  uint32_t g_memsize;
  while (1)
  {
    if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON )
    {
      /* KEY1 被按下 */
      if(NULL == Test_Ptr)
      {
        /* 获取当前内存大小 */
        g_memsize = xPortGetFreeHeapSize();
        printf("系统当前内存大小为 %d 字节，开始申请内存\n",g_memsize);
        Test_Ptr = pvPortMalloc(1024);
        if(NULL != Test_Ptr)
        {
          printf("内存申请成功\n");
          printf("申请到的内存地址为%#x\n",(int)Test_Ptr);
 
          /* 获取当前内剩余存大小 */
          g_memsize = xPortGetFreeHeapSize();
          printf("系统当前内存剩余存大小为 %d 字节\n",g_memsize);
                  
          //向Test_Ptr中写入当数据:当前系统时间
          sprintf((char*)Test_Ptr,"当前系统TickCount = %d \n",xTaskGetTickCount());
          printf("写入的数据是 %s \n",(char*)Test_Ptr);
        }
      }
      else
      {
        printf("请先按下KEY2释放内存再申请\n");
      }
    } 
    if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON )
    {
      /* KEY2 被按下 */
      if(NULL != Test_Ptr)
      {
        printf("释放内存\n");
        vPortFree(Test_Ptr);//释放内存
        Test_Ptr=NULL;
        /* 获取当前内剩余存大小 */
        g_memsize = xPortGetFreeHeapSize();
        printf("系统当前内存大小为 %d 字节，内存释放完成\n",g_memsize);
      }
      else
      {
        printf("请先按下KEY1申请内存再释放\n");
      }
    }
    vTaskDelay(20);/* 延时20个tick */
  }
}

int main(void)
{
  BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
  
  taskENTER_CRITICAL();           //进入临界区
 
  /* 创建LED_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )LED_Task, /* 任务入口函数 */
                        (const char*    )"LED_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )2,   /* 任务的优先级 */
                        (TaskHandle_t*  )&LED_Task_Handle);/* 任务控制块指针 */
  if(pdPASS == xReturn)
    printf("创建LED_Task任务成功\n");
  
  /* 创建Test_Task任务 */
  xReturn = xTaskCreate((TaskFunction_t )Test_Task,  /* 任务入口函数 */
                        (const char*    )"Test_Task",/* 任务名字 */
                        (uint16_t       )512,   /* 任务栈大小 */
                        (void*          )NULL,  /* 任务入口函数参数 */
                        (UBaseType_t    )3,     /* 任务的优先级 */
                        (TaskHandle_t*  )&Test_Task_Handle);/* 任务控制块指针 */ 
  if(pdPASS == xReturn)
    printf("创建Test_Task任务成功\n\n");
  
  taskEXIT_CRITICAL();            //退出临界区

  vTaskStartScheduler();   /* 启动任务，开启调度 */
}
```

## 九、中断管理

与中断相关的硬件可以划分为三类：外设、中断控制器、CPU 本身。

- 外设：当外设需要请求 CPU 时，产生一个中断信号，该信号连接至中断控制器。

- 中断控制器：中断控制器是 CPU 众多外设中的一个，它一方面接收其他外设中断信号的输入，另一方面，它会发出中断信号给 CPU。可以通过对中断控制器编程实现对中断源的优先级、触发方式、打开和关闭源等设置操作。在 Cortex-M 系列控制器中常用的中断控制器是 NVIC（内嵌向量中断控制器 Nested Vectored Interrupt Controller）。

- CPU：CPU 会响应中断源的请求，中断当前正在执行的任务，转而执行中断处理程序。 NVIC 最多支持 240个中断，每个中断最多 256 个优先级。

### 中断相关名词

中断号：每个中断请求信号都会有特定的标志，使得计算机能够判断是哪个设备提出 的中断请求，这个标志就是中断号。

- 中断请求：“紧急事件”需向 CPU 提出申请，要求 CPU 暂停当前执行的任务，转而处理该“紧急事件”，这一申请过程称为中断请求。

- 中断优先级：为使系统能够及时响应并处理所有中断，系统根据中断时间的重要性和紧迫程度，将中断源分为若干个级别，称作中断优先级。

- 中断处理程序：当外设产生中断请求后，CPU 暂停当前的任务，转而响应中断申请，即执行中断处理程序。

- 中断触发：中断源发出并送给 CPU 控制信号，将中断触发器置“1”，表明该中断源 产生了中断，要求 CPU 去响应该中断，CPU 暂停当前任务，执行相应的中断处理程序。 中断触发类型：外部中断申请通过一个物理信号发送到 NVIC，可以是电平触发或边沿触发。

- 中断向量：中断服务程序的入口地址。

- 中断向量表：存储中断向量的存储区，中断向量与中断号对应，中断向量在中断向量表中按照中断号顺序存储。

- 临界段：代码的临界段也称为临界区，一旦这部分代码开始执行，则不允许任何中断打断。为确保临界段代码的执行不被中断，在进入临界段之前须关中断，而临界段代码执行完毕后，要立即开中断。

### 中断运行机制

当中断产生时，处理机将按如下的顺序执行：

1. 保存当前处理机状态信息

2. 载入异常或中断处理函数到 PC寄存器

3. 把控制权转交给处理函数并开始执行

4. 当处理函数执行完成时，恢复处理器状态信息

5. 从异常或中断中返回到前一个程序执行点
  
中断发生的环境有两种情况：在任务的上下文中，在中断服务函数处理上下文中。

### 中断管理

ARM Cortex-M 系列内核的中断是由硬件管理的，而 FreeRTOS 是软件，它并不接管由硬件管理的相关中断（接管简单来说就是，所有的中断都由 RTOS 的软件管理，硬件来了 中断时，由软件决定是否响应，可以挂起中断，延迟响应或者不响应），只支持简单的开关中断等。

<mark>所以 FreeRTOS 中的中断使用其实跟裸机差不多的，需要我们自己配置中断，并且使能中断，编写中断服务函数，在中断服务函数中使用内核 IPC 通信机制，一般建议使用信号量、消息或事件标志组等标志事件的发生，将事件发布给处理任务，等退出中断 后再由相关处理任务具体处理中断

- 中断管理实验

  和裸机一样，可以定义GPIO，或者其他外设的触发中断方式，其触发的中断服务函数也与裸机一样。
  
  区别只是在中断服务函数中使用了FreeRTOS的通信机制，即调用中断版本的信号量API，通过信号量传递给任务，实现中断。具体代码可以跳转博客

## 参考资料

1. [FreeRTOS教程CSDN](https://blog.csdn.net/qq_61672347/article/details/125748646?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522171318107516800182763903%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=171318107516800182763903&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-125748646-null-null.142^v100^pc_search_result_base7&utm_term=freertos&spm=1018.2226.3001.4187)