!!! tip "脚本语言"

    系统批处理脚本: batch file

    推荐其他脚本语言：python

## Windows 使用记录

### 文件夹网络共享

在同一局域网内，可以通过文件夹网络共享分别地在设备间传递文件。

1. 右键文件夹-属性-共享-高级共享（共享此文件夹）
2. 共享-共享-添加everyone或特定用户，并且添加 读取/写入权限
3. 若想文件夹不需要密码访问，控制面板\网络和 Internet\网络和共享中心-更改高级共享设置（密码保护的共享）
4. 手机访问电脑的共享文件夹——下载带网络访问的文件管理器（CX，ES）等等，找到网络选项即可

### 自动任务

#### 自动任务——设置Windows开机启动项

- 1.win+R调出运行 输入`shell:startup`，进入开机启动项
- 2.将你的exe快捷方式放在这里
- 3.打开任务管理器-启动，找到你的程序，以确认添加成功(此处也可禁用启动程序)  

#### 自动任务——设置Windows定时自动任务

- 1.win+R调出运行 输入compmgmt.msc/或者右键左下角Windows LOGO打开计算机管理
- 2.系统工具-任务计划程序-创建基本任务,（添加任务名称，添加触发，添加启动程序、脚本，完成）

## CMD 指令

ipconfig

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

### 添加指令到环境变量

[百度-为什么开发程序要配置环境变量](https://baijiahao.baidu.com/s?id=1739693765201054630&wfr=spider&for=pc)

- 一、系统属性 -> 环境变量 -> PATH新建变量（用户变量 or 系统变量）

    需要注意的是：系统变量是作用系统全局的，用户变量只作用于当前用户

- 二、cmd测试指令是否执行

### Git Bash 添加 make 指令

[GIT BASH支持make](https://www.cnblogs.com/WLCYSYS/p/16715366.html)

git的bash实际上也就是一个mingw，是可以支持部分linux指令的，但是只有少部分。在编译代码的时候经常会使用make命令反而在bash下默认是不支持的。

当然是有办法可以解决的：

到 https://sourceforge.net/projects/ezwinports/files/ 去下载 make-4.1-2-without-guile-w32-bin.zip 文件。
把该文件进行解压把出来的文件全部拷贝的git的安装目录下： . \Program Files\Git\mingw64\ ,把文件夹进行合并，如果跳出来需要替换的文件要选择不替换

这样在git bash窗口下就可以执行make了（编译的时候编译器还是需要另外安装的）

## 批处理脚本batch .bat

在DOS,OS/2和微软Windows操作系统中，批处理文件(batch file)是包含一系列命令的文本文件，由命令解释器解释执行。批处理文件是一种简单的程序，可以通过条件语句(if)和流程控制语句(goto)来控制命令运行的流程，在批处理中也可以使用循环语句(for)来循环执行一条命令。

- **@echo off表示执行了这条命令后关闭所有命令(包括本身这条命令)的回显**
- **如果需要显示中文，请让.bat以ASNI编码保存**
- **自动息屏，可以设置时间 timeout /t < time >**

```bash
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
```