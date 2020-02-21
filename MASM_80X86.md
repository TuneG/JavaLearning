## 8086汇编语言编译器--Dosbox&&Masm&&TD

### 安装masm

1. 下载dosbox安装程序：DOSBox0.74-win32-installer.exe

   链接：https://pan.baidu.com/s/1N7DQUhCuHMMgZWuD2siKHQ  提取码：47zj

   百度云盘下载安装。注意它是32位的。我装在了C盘program files(X86)。

2. 下载masm文件。

   链接：https://pan.baidu.com/s/1NKCtzB3rjUgIt569SZCjPQ 
   提取码：ptd9

   masm文件夹内至少要包含这4个文件：masm.exe, link.exe, debug.exe, exe2bin.exe。其中：

   masm.exe：汇编程序，用于汇编源程序(.asm)，得到目标程序(.obj)；

   link.exe：连接程序，用于连接目标程序，得到可执行程序(.exe)；

   debug.exe：调试程序，用于调试可执行程序。

   还可以下载其他的程序。

3. 下载TD文件

   Turbo Debugger(TD)属源代码级调试器，可以调试多种由不同语言编写成的程序。Turbo Debugge 中的重叠式窗口、下拉式和弹出式菜单、热键的使用以及对鼠标的支持等，为用户提供了一个快速、方便和交互式的环境。 此外，联机帮助还可以在操作的每个阶段提供相关的帮助。
    32位段的汇编语言程序与16位段的汇编语言程序在调试时使用的版本是不同的，但它们对用户的操作界面（包括菜单和热键）基本是一样的，我用TD调试16位段的汇编语言程序。

   （上述masm文件中包括了TD.EXE，可直接使用）

### 配置

将dos\masm挂载到dosbox的驱动器下。例如挂载到dosbox的c驱动器下，即虚拟存在的c盘。有两种方法:

1) 运行dosbox，输入Z:\> mount c d:\dos\masm。c是指dosbox的c盘，d:\dos\masm是本机上工作目录dos的位置。

2) 在dosbox的安装文件夹中找到Dosbox 0.74 Options.bat文件，在末尾增加：

mount c d:/dos/masm ; 挂载驱动器

set path=z:\;c:\ ; 添加路径

c: ; 转到c盘


### 使用masm

~~~
.386
DATA  SEGMENT USE16
BUF   DB' HOW ARE YOU ! $'
DATA  ENDS
STACK SEGMENT USE16 STACK
	  DB 200 DUP(0)
STACK ENDS
CODE  SEGMENT USE16
      ASSUME CS:CODE,DS:DATA,SS:STACK
START:MOV  AX,DATA
	  MOV  DS,AX
	  LEA  DX,BUF
	  MOV  AH,9
	  INT  21H
	  MOV  AH,4CH
	  INT  21H
CODE  ENDS
      END  START
~~~

保存为EXAM.asm，保存在d:/dos/masm目录下

运行的简单方式：

1. 汇编源文件，生成目标文件

   MASM EXAM;

2. 连接目标文件，生成可执行文件

   LINK EXAM;

3. 运行可执行文件EXAM.EXE,得到运行结果

   EXAM
   
4. 调试可执行文件EXAM.EXE

   TD EXAM.EXE

   TD调试界面如下：

   ![image-20200210193444568](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200210193444568.png)

