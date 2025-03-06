# MCU开发

## 软件开发流程

### 分层设计思想

- 应用层

- 设备模块层

- 硬件驱动层

### 固件更新

- IAP(In Application Program) / OTA(Over The Air)

  Bootloader(引导程序)+FlashArea1(APP1应用程序段)+FlashArea2(APP2应用程序备份段)

  为了防止刷新失败，应该有刷新回滚功能，即更新备份段覆盖程序段

      主要在于Bootloader的程序编写，次要在于APP程序中断向量表偏移

      Bootloader：通信UART/CAN，来接收APP的bin文件，并且写入片内Flash/SRAM
      
      APP：由于Boot程序已经占用了Flash的一些空间，所以需要根据占用来偏移APP程序的所在空间，并且调整中断向量表的偏移

### 软件测试

## 开发经验

### UART 异步串口RX TX

### KEY 按键扫描

### 使用矩阵检测按键

#### 使用ADC检测多个按键 

#### 硬件消抖

- RS触发器

- 滤波电容滤波

#### 软件消抖

- 延时 消耗CPU资源

- 定时器 不消耗CPU资源

### FLASH

### INT interrupt

### IIC（SDA,CLK,GND）

半双工，带有时钟线CLK，同步信号

### SPI（MISO,MOSI,CLK,GND）

全双工 带有时钟线CLK 同步信号

### UART(RX,TX)

全双工，无CLK，异步信号

### CAN（CANH CANL）

半双工，双绞线，无CLK，异步信号

- CAN 2.0协议
  
    8byte 数据域

- CAN FD协议
  
    64byte 数据域

- CANopen协议（应用层）

    常用于机器人，医疗器械，工业控制领域

- UDS诊断协议（应用层）

    用于汽车电子领域

## 嵌入式GUI开发

### 上位机

- C#

- QT

### 下位机

- LVGL（MCU）

- emwin（MCU）

- QT（ARM）
