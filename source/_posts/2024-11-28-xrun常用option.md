---
title: xrun常用option
date: 2024-11-28 11:12:18
tags:
---



```bash
-default_ext verilog
Functional Description：
xrun generates an error if it encounters a file with an extension that it does not recognize. Use the -default_ext option to specify the file type for files with unrecognized file extensions. 

-libext <ext>   
Functional Description：
Specify extensions to be used for the -y search

-nowarn BIDANA:LIBNOU:PMBMDV:ILIOPT:RMVANAL
关闭指定的warning信息


-xmlibdirname
Functional Description：
This option specifies an alternative name for the directory in which the snapshot, libraries, and other generated objects are stored after invoking a simulation with xrun. By default, xrun uses the directory name xcelium.d for storing this information.


-allowredefinition         
Functional Description：
Allow multiple files to define the same object

-nontcglitch
Functional Description：
In a design that contains negative timing checks, the default negative timing check algorithm will calculate, when needed, delays with two values (rise and fall) for the different nets. When a delay with two values is calculated, there is the possibility that an event on the input net may cancel a scheduled event on the internal signal driven by the delay. This is called glitch suppression. Because glitch suppression can hide input events from a timing check's input, the simulator generates a glitch suppression timing violation if an event on a delayed signal is canceled.
Glitch suppression messages are displayed by default. 
Use the xmsim -nontcglitch option if you want to suppress these messages.

-loadpli1
Functional Description：
If a PLI application has already been compiled into a dynamic shared library, you can use -loadpli1 to load the library and to register the system tasks defined in the application at run time.
The argument to this option is the name or full path of the shared library that contains the PLI application followed by the name of the function that registers the new system tasks. This function, called the bootstrap function, is part of the PLI application, and is defined in the shared library.
You can load any number of applications in the same statement by separating the names of the bootstrap functions with a comma. No spaces are allowed in the argument.
The file extension of the shared library is optional. The elaborator appends a suffix that is consistent with the OS that you are running. For example, if you are running on the SUN4v platform and enter the following command, the elaborator searches for a library called SSI.so .
% xmelab -loadpli1 SSI:ssi_boot top
For PLI1.0 applications, the simulator always loads the shared library and executes any bootstrap function(s) that is passed to the elaborator.
通过 -loadpli1 指定共享库的同时，必须提供该引导函数的名称

-wreal  <wreal1driver|wreal4state|wrealmin|wrealmax|wrealsum|wrealavg> 
Enable wreal AMS datatype

-wreal_resolution <default|4state|sum|avg|max|min> 
Set the global wreal resolution function

```



>-loadpli1选项说明：
>
>背景知识
>
>`PLI（Programming Language Interface）` 是 Verilog 仿真中的一种扩展机制，允许用户通过 C 语言编写自定义的系统任务或函数。这些任务或函数可以在仿真运行时被调用，用于执行仿真器本身无法直接实现的特殊功能。
>
>------
>
>关键点解析
>
>1. **动态共享库加载**：
>   - 如果一个 PLI 应用程序已经被编译为动态共享库（比如 `.so` 或 `.dll` 文件），可以通过 `-loadpli1` 选项在运行时加载这个共享库。
>   - 加载的同时，这个选项还会注册 PLI 应用程序中定义的新系统任务。
>2. **引导函数（Bootstrap Function）**：
>   - 每个 PLI 应用程序共享库中包含一个**引导函数**，负责将该应用程序中的新系统任务注册到仿真器。
>   - 通过 `-loadpli1` 指定共享库的同时，必须提供该引导函数的名称。
>   - 格式：`库名称:引导函数名`。
>3. **支持多个共享库**：
>   - 可以一次性加载多个 PLI 应用程序。
>   - 用逗号分隔多个引导函数名，**注意**：逗号之间不能有空格。
>4. **文件扩展名处理**：
>   - 动态共享库的文件扩展名（如 `.so` 或 `.dll`）可以省略。
>   - 仿真器会根据操作系统自动补充扩展名。例如：
>     - 在 Linux 或 Solaris 上扩展为 `.so`。
>     - 在 Windows 上扩展为 `.dll`。
>5. **PLI1.0 应用**：
>   - 对于 PLI 1.0 应用，仿真器总是按照上述流程加载共享库并执行提供的引导函数。



---

参考博客：

1. [RUN无法识别$fsdbDumpfile函数的解决办法](https://blog.csdn.net/m0_48731721/article/details/127773239)

