---
title: Linux 程序调试
date: 2018-08-24 11:04:34
categories: Linux
tags: [Linux]
---

> GDB是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。

<!-- more -->


# 进程调试

## gdb 程序交互调试

GDB是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。

GDB中的命令固然很多，但我们只需掌握其中十个左右的命令，就大致可以完成日常的基本的程序调试工作。

以下从一个完整的调试过程简单说明最基本的几个命令;

```
    $gdb programmer     # 启动gdb
    >break main         # 设置断点
    >run                # 运行调试程序
    >next               # 单步调试
    >print var1         # 在调试过程中，我们需要查看当前某个变量值的时候，使用print 命令打印该值
    >list               # 显示当前调试处的源代码
    >info b             # 显示当前断点设置情况
```

同时，你需要更高效的调试：常用的调试命令都会有单字符的缩写，使用缩写更方便；同时，直接敲回车表示重复执行上一步命令；这在单步调试时非常有用；

## pstack 跟踪栈空间

> `pstack`是一个脚本工具，可显示每个进程的栈跟踪。`pstack` 命令必须由相应进程的属主或 root 运行。其核心实现就是使用了`gdb`以及`thread apply all bt`命令;

### 语法

```
    pstrack <program-pid>
```

### 示例:

```
    [root@caoxl ~]# pstack 777
    #0  0x00007fde797c117c in waitpid () from /usr/lib64/libc.so.6
    #1  0x0000000000440b24 in waitchld.isra.10 ()
    #2  0x0000000000441ddc in wait_for ()
    #3  0x0000000000433aae in execute_command_internal ()
    #4  0x000000000046f6e4 in parse_and_execute ()
    #5  0x000000000042f27b in execute_builtin.isra.2 ()
    #6  0x00000000004313d9 in execute_simple_command ()
    #7  0x00000000004322e3 in execute_command_internal ()
    #8  0x0000000000432953 in execute_command_internal ()
    #9  0x000000000043283f in execute_command_internal ()
    #10 0x0000000000434e2f in execute_function.isra.7 ()
    #11 0x00000000004316f1 in execute_simple_command ()
    #12 0x00000000004322e3 in execute_command_internal ()
    #13 0x0000000000432953 in execute_command_internal ()
    #14 0x0000000000433cce in execute_command ()
    #15 0x0000000000432917 in execute_command_internal ()
    #16 0x0000000000433cce in execute_command ()
    #17 0x0000000000432917 in execute_command_internal ()
    #18 0x0000000000433cce in execute_command ()
    #19 0x0000000000432917 in execute_command_internal ()
    #20 0x0000000000433cce in execute_command ()
    #21 0x0000000000432917 in execute_command_internal ()
    #22 0x0000000000433cce in execute_command ()
    #23 0x0000000000432917 in execute_command_internal ()
    #24 0x0000000000433cce in execute_command ()
    #25 0x0000000000432917 in execute_command_internal ()
    #26 0x0000000000433cce in execute_command ()
    #27 0x0000000000432917 in execute_command_internal ()
    #28 0x0000000000433cce in execute_command ()
    #29 0x0000000000432917 in execute_command_internal ()
    #30 0x0000000000433cce in execute_command ()
    #31 0x0000000000433da5 in execute_while_or_until ()
    #32 0x0000000000432410 in execute_command_internal ()
    #33 0x0000000000433cce in execute_command ()
    #34 0x000000000041e305 in reader_loop ()
    #35 0x000000000041c96e in main ()
```

## strace 分析系统调用

> strace常用来跟踪进程执行时的系统调用和所接收的信号。在Linux世界，进程不能直接访问硬件设备，当进程需要访问硬件设备(比如读取磁盘文件，接收网络数据等等)时，必须由用户态模式切换至内核态模式，通过系统调用访问硬件设备。strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。

完整程序:

```
    strace -o output.txt -T -tt -e trace=all -p 777
```

跟踪777进程的所有系统调用（-e trace=all），并统计系统调用的花费时间，以及开始时间（以可视化的时分秒格式显示），最后将记录结果存在output.txt文件里面。

查看进程正在做什么(实时输出进程执行系统调用的情况):

```
    [root@caoxl ~]# strace -p 777
    strace: Process 777 attached
    wait4(-1, 
```

# 目标文件分析

## nm

nm用来列出目标文件的符号清单。

```
    $nm myProgrammer
    08049f28 d _DYNAMIC
    08049ff4 d _GLOBAL_OFFSET_TABLE_
    080484dc R _IO_stdin_used
             w _Jv_RegisterClasses
    08049f18 d __CTOR_END__
    08049f14 d __CTOR_LIST__
    08049f20 D __DTOR_END__
    08049f1c d __DTOR_LIST__
    080485e0 r __FRAME_END__
    08049f24 d __JCR_END__
    08049f24 d __JCR_LIST__
    0804a014 A __bss_start
    0804a00c D __data_start
    08048490 t __do_global_ctors_aux
    08048360 t __do_global_dtors_aux
    0804a010 D __dso_handle
             w __gmon_start__
    08048482 T __i686.get_pc_thunk.bx
    08049f14 d __init_array_end
    08049f14 d __init_array_start
    08048480 T __libc_csu_fini
    08048410 T __libc_csu_init
             U __libc_start_main@@GLIBC_2.0
    0804a014 A _edata
    0804a01c A _end
    080484bc T _fini
    080484d8 R _fp_hw
    080482b4 T _init
    08048330 T _start
    0804a014 b completed.6086
    0804a00c W data_start
    0804a018 b dtor_idx.6088
    080483c0 t frame_dummy
    080483e4 T main
             U printf@@GLIBC_2.0
```

```
    [root@caoxl lib64]# nm -X 32_64 /usr/lib64/liby.a
    
    main.o:
    0000000000000000 T main
                     U setlocale
                     U yyparse
    
    yyerror.o:
                     U fputc
                     U fputs
                     U stderr
    0000000000000000 T yyerror
```

这些包含可执行代码的段称为正文段。同样地，数据段包含了不可执行的信息或数据。另一种类型的段，称为 BSS 段，它包含以符号数据开头的块。对于 nm 命令列出的每个符号，它们的值使用十六进制来表示（缺省行为），并且在该符号前面加上了一个表示符号类型的编码字符。

常见的各种编码包括：

- A 表示绝对 (absolute)，这意味着不能将该值更改为其他的连接；
- B 表示 BSS 段中的符号；
- C 表示引用未初始化的数据的一般符号。

可以将目标文件中所包含的不同的部分划分为段。段可以包含可执行代码、符号名称、初始数据值和许多其他类型的数据。有关这些类型的数据的详细信息，可以阅读 UNIX 中 nm 的 man 页面，其中按照该命令输出中的字符编码分别对每种类型进行了描述。

在目标文件阶段，即使是一个简单的 Hello World 程序，其中也包含了大量的细节信息。nm 程序可用于列举符号及其类型和值，但是，要更仔细地研究目标文件中这些命名段的内容，需要使用功能更强大的工具。

## objdump

ogjdump工具用来显示二进制文件的信息，就是以一种可阅读的格式让你更多地了解二进制文件可能带有的附加信息。

```
    [root@caoxl lib64]# objdump -d liby.a
    In archive liby.a:
    
    main.o:     file format elf64-x86-64
    
    
    Disassembly of section .text.startup:
    
    0000000000000000 <main>:
       0:	48 83 ec 08          	sub    $0x8,%rsp
       4:	be 00 00 00 00       	mov    $0x0,%esi
       9:	bf 06 00 00 00       	mov    $0x6,%edi
       e:	e8 00 00 00 00       	callq  13 <main+0x13>
      13:	48 83 c4 08          	add    $0x8,%rsp
      17:	e9 00 00 00 00       	jmpq   1c <main+0x1c>
    
    yyerror.o:     file format elf64-x86-64
    
    
    Disassembly of section .text:
    
    0000000000000000 <yyerror>:
       0:	48 83 ec 08          	sub    $0x8,%rsp
       4:	48 8b 35 00 00 00 00 	mov    0x0(%rip),%rsi        # b <yyerror+0xb>
       b:	e8 00 00 00 00       	callq  10 <yyerror+0x10>
      10:	48 8b 35 00 00 00 00 	mov    0x0(%rip),%rsi        # 17 <yyerror+0x17>
      17:	bf 0a 00 00 00       	mov    $0xa,%edi
      1c:	e8 00 00 00 00       	callq  21 <yyerror+0x21>
      21:	31 c0                	xor    %eax,%eax
      23:	48 83 c4 08          	add    $0x8,%rsp
      27:	c3                   	retq   
```

每个可执行代码段将在需要特定的事件时执行，这些事件包括库的初始化和该程序本身主入口点。

对于那些着迷于底层编程细节的程序员来说，这是一个功能非常强大的工具，可用于研究编译器和汇编器的输出。细节信息，比如这段代码中所显示的这些信息，可以揭示有关本地处理器本身运行方式的很多内容。对该处理器制造商提供的技术文档进行深入的研究，您可以收集关于一些有价值的信息，通过这些信息可以深入地了解内部的运行机制，因为功能程序提供了清晰的输出。

> 我的功力还没有那么深厚, 这些都是转载而来, 等我努力努力再探讨这些.

## readelf

这个工具和objdump命令提供的功能类似，但是它显示的信息更为具体，并且它不依赖BFD库(BFD库是一个GNU项目，它的目标就是希望通过一种统一的接口来处理不同的目标文件）；

```
    [root@caoxl lib64]# readelf -all liby.a
    
    File: liby.a(main.o)
    ELF Header:
      Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
      Class:                             ELF64
      Data:                              2's complement, little endian
      Version:                           1 (current)
      OS/ABI:                            UNIX - System V
      ABI Version:                       0
      Type:                              REL (Relocatable file)
      Machine:                           Advanced Micro Devices X86-64
      Version:                           0x1
      Entry point address:               0x0
      Start of program headers:          0 (bytes into file)
      Start of section headers:          736 (bytes into file)
      Flags:                             0x0
      Size of this header:               64 (bytes)
      Size of program headers:           0 (bytes)
      Number of program headers:         0
      Size of section headers:           64 (bytes)
      Number of section headers:         14
      Section header string table index: 11
    ...
```

ELF Header 为该文件中所有段入口显示了详细的摘要。在列举出这些 Header 中的内容之前，您可以看到 Header 的具体数目。在研究一个较大的目标文件时，该信息可能非常有用。

除了所有这些段之外，编译器可以将调试信息放入到目标文件中，并且还可以显示这些信息。输入下面的命令，仔细分析编译器的输出（假设您扮演了调试程序的角色）:

```
    readelf --debug-dump a.out | more
```

调试工具，如 GDB，可以读取这些调试信息，并且当程序在调试器中运行的同时，您可以使用该工具显示更具描述性的标记，而不是对代码进行反汇编时的原始地址值。


## size 查看程序内存占用

size这个工具用来查看程序运行时各个段的实际内存占用:

```
    [root@caoxl lib64]# size liby.a
       text	   data	    bss	    dec	    hex	filename
         77	      0	      0	     77	     4d	main.o (ex liby.a)
         88	      0	      0	     88	     58	yyerror.o (ex liby.a)
```

## file 文件类型查询

这个工具用于查看文件的类型；

比如我们在64位机器上发现了一个32位的库，链接不上，这就有问题了：

```
    [root@caoxl lib64]# file liby.a
    liby.a: current ar archive
```

也可以查看Core文件是由哪个程序生成:

```
    [root@caoxl lib64]# file core.22355
```

## strings 查询数据中的文本信息

一个文件中包含二进制数据和文本数据，如果只需要查看其文本信息，使用这个命令就很方便；过滤掉非字符数据，将文本信息输出:

```
    strings <objfile>
    
    # 例
    [root@caoxl ~]# strings test/test.php
    <?php
    echo 1024;
```

## fuser 显示文件使用者

显示所有正在使用着指定的file, file system 或者 sockets的进程信息;

```
    [root@caoxl ~]# fuser -m -u test/test.php
    /root/test/test.php:     1rce(root)     2rc(root)     3rc(root)     5rc(root)     6rc(root)     7rc(root)     8rc(root)     9rc(root)    10rc(root)    11rc(root)    14rc(root)    15rc(root)    16rc(root)    17rc(root)    18rc(root)    19rc(root)    20rc(root)    21rc(root)    22rc(root)    23rc(root)    29rc(root)    30rc(root)    31rc(root)    32rc(root)    40rc(root)    42rc(root)    43rc(root)    44rc(root)    45rc(root)    58rc(root)    89rc(root)   219rc(root)   229rc(root)   230rc(root)   231rc(root)   232rc(root)   235rc(root)   242rc(root)   254rc(root)   255rc(root)   322rce(root)   354rce(root)   431rce(root)   460rce(polkitd)   462rce(dbus)   468rce(ntp)   470rce(root)   471rce(root)   472rce(root)   483rce(root)   711rce(root)   716rce(root)   777rce(root)  1358rce(mysql)  1448rce(root)  1476rce(root)  1508rce(root)  2277rce(root)  2278rce(www)  3114rce(root)  3121rce(root)  5457rce(root)  5459rce(root)  5628rc(root)  5643rc(root)  5652rc(root) 14571rc(root) 19436rce(root) 25899rce(root) 26119rce(root) 26120rce(www) 26121rce(www) 26828rce(www) 26829rce(www) 26830rce(www) 26831rce(www) 26832rce(www) 26833rce(www) 26834rce(www) 26835rce(www) 26841rce(www) 26842rce(www) 26863rce(www) 26864rce(www) 26865rce(www) 26866rce(www) 26867rce(www) 26982rce(www) 26983rce(www)
```

使用了`-m`和`-u`选项，用来查找所有正在使用test.php的所有进程的PID以及该进程的OWNER；

fuser通常被用在诊断系统的”resource busy”问题。如果你希望kill所有正在使用某一指定的file, file system or sockets的进程的时候，你可以使用-k选项:

```
    fuser –k /path/to/your/filename
```

## xxd 十六进制显示数据

```
    $xxd a.out
    0000000: 7f45 4c46 0101 0100 0000 0000 0000 0000  .ELF............
    0000010: 0200 0300 0100 0000 3083 0408 3400 0000  ........0...4...
    0000020: 3c11 0000 0000 0000 3400 2000 0900 2800  <.......4. ...(.
    0000030: 1e00 1b00 0600 0000 3400 0000 3480 0408  ........4...4...
    0000040: 3480 0408 2001 0000 2001 0000 0500 0000  4... ... .......
    0000050: 0400 0000 0300 0000 5401 0000 5481 0408  ........T...T...
    ...
```


## od 查看特殊格式的文件内容

通常使用od命令查看特殊格式的文件内容。通过指定该命令的不同选项可以以十进制、八进制、十六进制和ASCII码来显示文件。

参数说明:

- `-A` 指定地址基数，包括：
  - `d` 十进制
  - `o` 八进制（系统默认值）
  - `x` 十六进制
  - `n` 不打印位移值
  
- `-t` 指定数据的显示格式，主要的参数有：
  - `c` ASCII字符或反斜杠序列
  - `d` 有符号十进制数
  - `f` 浮点数
  - `o` 八进制（系统默认值为02）
  - `u` 无符号十进制数
  - `x` 十六进制数
  
除了选项c以外的其他选项后面都可以跟一个十进制数n，指定每个显示值所包含的字节数。

> 说明：od命令系统默认的显示方式是八进制，这也是该命令的名称由来（`Octal Dump`）。但这不是最有用的显示方式，用ASCII码和十六进制组合的方式能提供更有价值的信息输出。

- 以十六进制和字符同时显示:

```
    $od -Ax -tcx4 a.c
    000000   #   i   n   c   l   u   d   e       <   s   t   d   i   o   .
                  636e6923        6564756c        74733c20        2e6f6964
    000010   h   >  \n  \n   v   o   i   d       m   a   i   n   (   )  \n
                  0a0a3e68        64696f76        69616d20        0a29286e
    000020   {  \n  \t   i   n   t       i       =       5   ;  \n  \t   p
                  69090a7b        6920746e        35203d20        70090a3b
    000030   r   i   n   t   f   (   "   h   e   l   l   o   ,   %   d   "
                  746e6972        68222866        6f6c6c65        2264252c
    000040   ,   i   )   ;  \n   }  \n
                  3b29692c        000a7d0a
    000047
```

- 以字符方式显示:

```
    $od -c a.c
    0000000   #   i   n   c   l   u   d   e       <   s   t   d   i   o   .
    0000020   h   >  \n  \n   v   o   i   d       m   a   i   n   (   )  \n
    0000040   {  \n  \t   i   n   t       i       =       5   ;  \n  \t   p
    0000060   r   i   n   t   f   (   "   h   e   l   l   o   ,   %   d   "
    0000100   ,   i   )   ;  \n   }  \n
    0000107
```