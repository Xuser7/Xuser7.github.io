GUI (Graphic User Interface)

## LVGL 

LVGL (Light and Versatile Graphics Library) is the most popular free and open-source embedded graphics library to create beautiful UIs for any MCU, MPU and display type.

[LVGL](https://lvgl.io/)  

[LVGL-Docs](https://docs.lvgl.io/master/index.html)

[LVGL-examples](https://docs.lvgl.io/master/examples.html#)


### LVGL模拟器

[模拟器Github仓库](https://docs.lvgl.io/master/integration/ide/pc-simulator.html)

[Simulator on PC](https://docs.lvgl.io/master/details/integration/ide/pc-simulator.html)

- VisualStudio: For Windows

    1.选择仓库源码release/v8.3，并且下载，此时获取的源码是没有配置的

    2.还需要下载LVGL.Simulator下freetype lv_drivers lvgl三个仓库文件，然后添加到你的本地文件下即可

    3.VisualStudio打开LVGL.Simulator.sln，生成解决方案后运行。

### LVGL移植

#### 00移植所需工程

1. [LVGL源码](https://github.com/lvgl/lvgl)

2. 硬件驱动工程，包含`屏幕显示`，`屏幕触摸`，`定时器`。确保工程部件可以正确运行
  
    1 需要显示驱动下的颜色填充函数，实现`disp_flush()`

    2 需要触摸驱动下的触摸扫描函数与获取坐标函数，实现`touchpad_is_pressed()`与`touchpad_get_xy()`

    3 需要提供一个时基，可以是普通定时器，可以是滴答定时器，实现`lv_tick_inc()`

#### 01选取源码中需要的文件

选取文件如下：（lv_conf_template.h修改名称为lv_conf.h，并且打开宏定义）

```
├─demos [ALL]
├─example
│  └─porting
├─src [ALL]
├─lv_conf.h
└─lvgl.h
```

- demos 为官方演示例程，如果不需要可以不作保留
- example 下只保留porting文件夹，负责对接LVGL接口与硬件驱动接口
- src 为LVGL的内核代码，全部保留
- lv_conf.h 为LVGL宏配置

#### 02移动到驱动工程文件夹

新建文件夹Middlewares，并且将选取后的文件，移动至Middlewares_LVGL_lvgl下;

文件目录应当如下结构：

```
Middlewares
└─LVGL
   └─lvgl
       ├─demos [ALL]
       ├─example
       │  └─porting
       ├─src [ALL]
       ├─lv_conf.h
       └─lvgl.h
```

<mark>这是为了保证代码头文件引用路径一致。例如../../lvgl.h这种相对路径。

当然，不嫌麻烦的话，可以自己定义文件结构，但是需要修改头文件引用路径。

- Middlewares 文件夹：目的在于收录所有中间层组件，例如可以向其添加FreeRTOS源码，配合LVGL

    ```c
    Middlewares
    ├─LVGL
    │   └─lvgl
    │       ├─demos [ALL]
    │       ├─example
    │       │  └─porting
    │       ├─src [ALL]
    │       ├─lv_conf.h
    │       └─lvgl.h
    │
    └─FreeRTOS
        ├─FreeRTOS_CORE
        └─FreeRTOS_PORT
        
    ```

#### 03驱动工程添加LVGL源码

keil工程内项目管理添加以下分组，并且向各个分组添加lvgl源码

```
Middlewares/lvgl/example/porting    ————添加 examples/porting文件夹下的lv_port_disp_template.c 和 lv_port_indev_template.c 文件

Middlewares/lvgl/src/core           ————添加 src/core 文件夹下的全部.c 文件

Middlewares/lvgl/src/draw           ————添加 draw 文件夹下除 nxp_pxp、nxp_vglite、sdl 和stm32_dma2d 文件夹之外的全部.c 文件

Middlewares/lvgl/src/extra          ————添加 extra 文件夹下的全部.c 文件，子文件夹较多，需要耐心

Middlewares/lvgl/src/font           ————添加 font 文件夹下的全部.c 文件

Middlewares/lvgl/src/gpu            ————添加 draw/stm32_dma2d 和 draw/sdl 文件夹下的全部.c 文件

Middlewares/lvgl/src/hal            ————添加 hal 文件夹下的全部.c 文件

Middlewares/lvgl/src/misc           ————添加 misc 文件夹下的全部.c 文件

Middlewares/lvgl/src/widgets        ————添加 widgets 文件夹下的全部.c 文件

```

#### 04驱动工程添加头文件路径

添加以下头文件路径

```
..\..\Middlewares\LVGL
..\..\Middlewares\LVGL\lvgl
..\..\Middlewares\LVGL\lvgl\src
..\..\Middlewares\LVGL\lvgl\examples\porting

```

编译器开启C99模式，编译，查看是否有error。正确的话是没有报错的

#### 05配置输出（屏幕显示）lv_port_disp_template

1. 打开`lv_port_disp_template.c`与`.h`的条件编译
2. 在c文件中包含屏幕硬件驱动的头文件（例`#include "lcd.h"`）
3. 实现显示初始化，`disp_init(void)`，并且设置显示方向

    ```c
    static void disp_init(void)
    {
        /*You code here*/
            LCD_Init();
            LCD_Display_Dir(1);//横屏
    }
    ```

4. 配置图形数据缓冲模式 `lv_port_disp_init(void)`

    ```c
    //三种方案，缓冲区越大，对RAM要求越大，选择一个注释剩余
    //1.十行的缓冲区
    //2.两份十行的缓冲区
    //3.两份全屏缓冲区
    
    /* Example for 1) */
    static lv_disp_draw_buf_t draw_buf_dsc_1;
    static lv_color_t buf_1[MY_DISP_HOR_RES * 10];                          /*A buffer for 10 rows*/
    lv_disp_draw_buf_init(&draw_buf_dsc_1, buf_1, NULL, MY_DISP_HOR_RES * 10);   /*Initialize the display buffer*/

    /* Example for 2) */
    static lv_disp_draw_buf_t draw_buf_dsc_2;
    static lv_color_t buf_2_1[MY_DISP_HOR_RES * 10];                        /*A buffer for 10 rows*/
    static lv_color_t buf_2_2[MY_DISP_HOR_RES * 10];                        /*An other buffer for 10 rows*/
    lv_disp_draw_buf_init(&draw_buf_dsc_2, buf_2_1, buf_2_2, MY_DISP_HOR_RES * 10);   /*Initialize the display buffer*/

    /* Example for 3) also set disp_drv.full_refresh = 1 below*/
    static lv_disp_draw_buf_t draw_buf_dsc_3;
    static lv_color_t buf_3_1[MY_DISP_HOR_RES * MY_DISP_VER_RES];            /*A screen sized buffer*/
    static lv_color_t buf_3_2[MY_DISP_HOR_RES * MY_DISP_VER_RES];            /*Another screen sized buffer*/
    lv_disp_draw_buf_init(&draw_buf_dsc_3, buf_3_1, buf_3_2, MY_DISP_VER_RES * LV_VER_RES_MAX);   /*Initialize the display buffer*/
    ```

5. 设置屏幕尺寸（默认横屏）

    `disp_drv.hor_res = lcddev.width;`

    `disp_drv.ver_res = lcddev.height;`

6. 配置像素点颜色填充输出`disp_flush()`
  
    ```c
    static void disp_flush(lv_disp_drv_t * disp_drv, const lv_area_t * area, lv_color_t * color_p)
    {
        //    /*The most simple case (but also the slowest) to put all pixels to the screen one-by-one*/

        //    int32_t x;
        //    int32_t y;
        //    for(y = area->y1; y <= area->y2; y++) {
        //        for(x = area->x1; x <= area->x2; x++) {
        //            /*Put a pixel to the display. For example:*/
        //            /*put_px(x, y, *color_p)*/
        //            color_p++;
        //        }
        //    }
            
        LCD_Color_Fill(area->x1,area->y1,area->x2,area->y2,(uint16_t*) color_p);
        /*IMPORTANT!!!
            *Inform the graphics library that you are ready with the flushing*/
        lv_disp_flush_ready(disp_drv);
    }
    ```

#### 06配置输入（触摸、鼠标、键盘、编码器、按钮）lv_port_indev_template

1. 打开`lv_port_indev_template.c`与`.h`的条件编译
2. 选择并且裁剪输出设备
  
    ```c
    //选择你的输入方式，删除或注释剩余的输入方式的函数定义与声明。以保持c文件阅读性
    static void touchpad_init(void);
    static void touchpad_read(lv_indev_drv_t * indev_drv, lv_indev_data_t * data);
    static bool touchpad_is_pressed(void);
    static void touchpad_get_xy(lv_coord_t * x, lv_coord_t * y);

    static void mouse_init(void);
    static void mouse_read(lv_indev_drv_t * indev_drv, lv_indev_data_t * data);
    static bool mouse_is_pressed(void);
    static void mouse_get_xy(lv_coord_t * x, lv_coord_t * y);

    static void keypad_init(void);
    static void keypad_read(lv_indev_drv_t * indev_drv, lv_indev_data_t * data);
    static uint32_t keypad_get_key(void);

    static void encoder_init(void);
    static void encoder_read(lv_indev_drv_t * indev_drv, lv_indev_data_t * data);
    static void encoder_handler(void);

    static void button_init(void);
    static void button_read(lv_indev_drv_t * indev_drv, lv_indev_data_t * data);
    static int8_t button_get_pressed_id(void);
    static bool button_is_pressed(uint8_t id);
    ```

3. 在c文件中包含输入设备硬件驱动的头文件（例`#include "touch.h"`）
4. 实现`touchpad_init()`
  
    ```c
    static void touchpad_init(void)
    {
        /*Your code comes here*/
            tp_dev.init();
    }
    ```

5. 配置触摸检测函数`touchpad_is_pressed(void)`

    ```c
    static bool touchpad_is_pressed(void)
    {
        /*Your code comes here*/
            tp_dev.scan(0);
            if(tp_dev.sta & TP_PRES_DOWN)
            {
                return true;
            }

        return false;
    }
    ```

6. 配置坐标获取函数`touchpad_get_xy()`

    ```c
    static void touchpad_get_xy(lv_coord_t * x, lv_coord_t * y)
    {
        /*Your code comes here*/
        //横屏
        (*x) = lcddev.width-tp_dev.y[0];
        (*y) = tp_dev.x[0];
    }
    ```
  
#### 07提供时基

- 硬件定时器提供时基

  1. 添加定时器驱动
  2. 在定时器驱动.c文件中包含：`#include "lvgl.h"`
  3. 在定时器中断函数（回调）中调用：`lv_tick_inc(x);`
  4. 初始化定时器时，需保证：进入中断的时间间隔 = x 毫秒

#### main创建一个switch测试运行

```c
#include "sys.h"
#include "delay.h"
#include "lcd.h"
#include "touch.h"
#include "timer.h"

#include "lvgl.h"
#include "lv_port_disp_template.h"
#include "lv_port_indev_template.h"
int main(void)
{
    delay_init();
    VIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//设置中断优先级分组为组2：2位抢占优先级，2位响应优先级
    TIM3_Int_Init(10-1, 9000-1);//时基
        
    lv_init(); 
    lv_port_disp_init(); //输出初始化
    lv_port_indev_init();//输入初始化

    //创建一个switch
    lv_obj_t* switch_obj = lv_switch_create(lv_scr_act());
    lv_obj_set_size(switch_obj, 120, 60);
    lv_obj_align(switch_obj, LV_ALIGN_CENTER, 0, 0);

    while(1)
    {
        delay_ms(5);
        lv_timer_handler();
    }
}
```

#### 测试官方demo(lv_demo_widgets)

1. 工程新建分组 Middlewares/lvgl/demos，添加`lv_demo_widgets.c`以及它的所需文件

    `img_clothes.c`

    `img_demo_widgets_avatar.c`

    `img_lvgl_logo.c`

2. 工程添加头文件路径

    ..\Middlewares\LVGL\lvgl\demos\widgets
    ..\Middlewares\LVGL\lvgl\demos\widgets\assets

3. 打开 lv_conf.h 文件，宏定义 LV_USE_DEMO_WIDGETS 设置为 1

4. main.c文件里包含头文件：`#include "lv_demo_widgets.h"`

5. 使用demo`lv_demo_widgets();`
