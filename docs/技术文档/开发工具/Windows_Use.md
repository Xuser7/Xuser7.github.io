# Windows

- **系统批处理脚本 batch file**

- **推荐其他脚本语言：python**

## 添加指令到环境变量（使指令可执行）

[百度-为什么开发程序要配置环境变量](https://baijiahao.baidu.com/s?id=1739693765201054630&wfr=spider&for=pc)

- 一、系统属性 -> 环境变量 -> PATH新建变量（用户变量 or 系统变量）

    需要注意的是：系统变量是作用系统全局的，用户变量只作用于当前用户
- 二、cmd测试指令是否执行

## cmd指令

- ipconfig：

        ipconfig - 查看类目下可用指令
        ipconfig -all 显示完整详细信息（控制面板可禁用网卡）

- ping：

        ping -a +ip地址 将地址解析成主机名。（获取IP主机的主机名）

- tree
  
        打印当前cmd窗口路径下的文件树

- arp -a查看局域网内所有设备
  
        （物理地址为 ff-ff-ff-ff-ff-ff 的条目是广播地址，不指向真实设备。）
        （224.0.0.0 到 239.255.255.255 的 IP 地址是组播地址，也不是物理设备）

- pip：(python第三方包管理)

        pip install xxx-i https://pypi.tuna.tsinghua.edu.cn/simple 清华软件源
        pip unistall xxx xxx xxx ... xxx 移除多个库
        pip freeze <requirement.txt> 查询第三方库(导出)
        pip uninstall -r <requirements.txt> -y (根据导出卸载)

## 批处理脚本batch .bat

在DOS,OS/2和微软Windows操作系统中，批处理文件(batch file)是包含一系列命令的文本文件，由命令解释器解释执行。批处理文件是一种简单的程序，可以通过条件语句(if)和流程控制语句(goto)来控制命令运行的流程，在批处理中也可以使用循环语句(for)来循环执行一条命令。

- **@echo off表示执行了这条命令后关闭所有命令(包括本身这条命令)的回显**
- **如果需要显示中文，请让.bat以ASNI编码保存**
- **自动息屏，可以设置时间 timeout /t < time >**

        @echo off
        echo 1分钟后自动熄灭屏幕

        ::显示时间
        echo %time%

        ::延时
        timeout /t 60

        ::显示时间
        echo %time%

        ::息屏
        start  C:\Windows\System32\rundll32.exe powrprof.dll,SetSuspendState Hibemate

## 设置Windows开机启动项 WIN10

- 1.win+R调出运行 输入`shell:startup`，进入开机启动项
- 2.将你的exe快捷方式放在这里
- 3.打开任务管理器-启动，找到你的程序，以确认添加成功(此处也可禁用启动程序)  

## 设置Windows定时自动任务 WIN10

- 1.win+R调出运行 输入compmgmt.msc/或者右键左下角Windows LOGO打开计算机管理
- 2.系统工具-任务计划程序-创建基本任务

    （添加任务名称，添加触发，添加启动程序、脚本，完成）

## Windows Tools

- Tesseract-OCR：(WIN教程)

    由HP实验室开发 由Google维护的开源的​光学字符识别（OCR）引擎，可以在 Apache 2.0 许可下获得。

    ​它可以直接使用，或者（对于程序员）使用 API 从图像中​提取输入，包括手写的或打印的文本。

    1.安装Tesseract——github UB-Mannheim
        需要勾选additional script 的han

    2.Tesseract-OCR添加到环境变量，添加TESSDATA_PREFIX，路径tessdata
        tesseract -v查看版本
        tesseract --list-langs查看语言包

    3.python开发
        pip install pytesseract
        进入python安装包的路径：D:\Program Files\Python37\Lib\site-packages\pytesseract
        编辑文件：pytesseract.py
        修改tesseract_cmd = ‘D:\Program Files\Tesseract-OCR/tesseract.exe’，修改后在python中运行就不会报错了。
