---
layout: page
title: 计算机基础篇-bomblab(一)
category: 
    - blogs
---

#csapp bomb lab
标签（空格分隔）： 计算机基础
---
layout: page
title: 计算机基础篇-bomb lab(一）1,2,3
category: 
    - blogs
---

----------
##The first
根据http://csapp.cs.cmu.edu/3e/bomblab.pdf获取信息
利用objdump -d ./bomb获得各函数的反汇编代码
发现了phase_1，phase_2，phase_3，phase_4，phase_5，phase_6
刚好和需要验证的字符串数相同
代码也提示了
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
所以phase_1就是第一个爆炸点了
```
0000000000400ee0 <phase_1>:
  400ee0:       48 83 ec 08             sub    $0x8,%rsp
  400ee4:       be 00 24 40 00          mov    $0x402400,%esi
  400ee9:       e8 4a 04 00 00          callq  401338 <strings_not_equal>
  400eee:       85 c0                   test   %eax,%eax
  400ef0:       74 05                   je     400ef7 <phase_1+0x17>
  400ef2:       e8 43 05 00 00          callq  40143a <explode_bomb>
  400ef7:       48 83 c4 08             add    $0x8,%rsp
  400efb:       c3                      retq
```

通过下面2行汇编代码了解到
strings_not_equal 使用了2个参数用来验证字符串
那么不出意外$0x402400这个地址就是第一个字符串,%esi就是我们的输入
至于explode_bomb从名字看便是爆炸

```
  400ee4:       be 00 24 40 00          mov    $0x402400,%esi
  400ee9:       e8 4a 04 00 00          callq  401338 <strings_not_equal>
```
官方提示必须gdb看到寄存器存储的值，那么就gdb吧
gdb ./bomb
然后打断点
b explode_bomb
b  phase_1
```
Admin@DESKTOP-4CJQTHR:/mnt/c/Windows/System32/ubuntu/csapp/bomb$ gdb ./bomb
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./bomb...done.
(gdb) b explode_bomb
Breakpoint 1 at 0x40143a
(gdb) b  phase_1
Breakpoint 2 at 0x400ee0
```
再run,随便输入一个数，比如nice
```
(gdb) run
Starting program: /mnt/c/Windows/System32/ubuntu/csapp/bomb/bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
nice

Breakpoint 2, 0x0000000000400ee0 in phase_1 ()
```
接着利用 disas 来看看对应的汇编代码，和之前看的一模一样
```
(gdb) disas
Dump of assembler code for function phase_1:
=> 0x0000000000400ee0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:     mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:     callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    retq
End of assembler dump.
```
然后info register查看寄存器里的内容
```
(gdb) info register
rax            0x603780 6305664
rbx            0x0      0
rcx            0x4      4
rdx            0x1      1
rsi            0x603780 6305664
rdi            0x603780 6305664
rbp            0x402210 0x402210 <__libc_csu_init>
rsp            0x7ffffffde2c8   0x7ffffffde2c8
r8             0x604225 6308389
r9             0x7fffff7c0700   140737479706368
r10            0x519    1305
r11            0x7fffff05e0f0   140737471963376
r12            0x400c90 4197520
r13            0x7ffffffde3b0   140737488217008
r14            0x0      0
r15            0x0      0
rip            0x400ee0 0x400ee0 <phase_1>
eflags         0x202    [ IF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
(gdb)
```
试试sepi,可以看到汇编代码在逐步执行

```
(gdb) stepi
0x0000000000400ee4 in phase_1 ()
(gdb) disas
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:     sub    $0x8,%rsp
=> 0x0000000000400ee4 <+4>:     mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:     callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    retq
End of assembler dump.
(gdb)
```
```
(gdb) disas
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:     mov    $0x402400,%esi
=> 0x0000000000400ee9 <+9>:     callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    retq
End of assembler dump.
(gdb) print $esi
$1 = 4203520
```
试试p/s (char*)0x402400
```
(gdb) p/s (char*)0x402400
$16 = 0x402400 "Border relations with Canada have never been better."
(gdb)
```
直接获得了第一个字符串
"Border relations with Canada have never been better."
？？？我不是加拿大的
测试答案
打好断点
```
(gdb) b explode_bomb
Breakpoint 1 at 0x40143a
(gdb) b phase_2
Breakpoint 2 at 0x400efc
```
测试是否通过phase_1
```
(gdb) r
Starting program: /mnt/c/Windows/System32/ubuntu/csapp/bomb/bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
```
成功了
##The Second
这里我整个人爆炸了，看https://www.zhihu.com/question/40720890才理解了lea，mov
lea是计算（地址+偏移）。将计算结果（地址）存储到另一个寄存器
mov是计算各地址上值后将值存储的到另一个寄存器
我可能描述不好，不理解没关系，继续往下看
接下来开始第二个
```
0000000000400efc <phase_2>:
  400efc:       55                      push   %rbp
  400efd:       53                      push   %rbx
  400efe:       48 83 ec 28             sub    $0x28,%rsp
  400f02:       48 89 e6                mov    %rsp,%rsi
  400f05:       e8 52 05 00 00          callq  40145c <read_six_numbers>
  400f0a:       83 3c 24 01             cmpl   $0x1,(%rsp)
  400f0e:       74 20                   je     400f30 <phase_2+0x34>
  400f10:       e8 25 05 00 00          callq  40143a <explode_bomb>
  400f15:       eb 19                   jmp    400f30 <phase_2+0x34>
  400f17:       8b 43 fc                mov    -0x4(%rbx),%eax
  400f1a:       01 c0                   add    %eax,%eax
  400f1c:       39 03                   cmp    %eax,(%rbx)
  400f1e:       74 05                   je     400f25 <phase_2+0x29>
  400f20:       e8 15 05 00 00          callq  40143a <explode_bomb>
  400f25:       48 83 c3 04             add    $0x4,%rbx
  400f29:       48 39 eb                cmp    %rbp,%rbx
  400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>
  400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>
  400f30:       48 8d 5c 24 04          lea    0x4(%rsp),%rbx
  400f35:       48 8d 6c 24 18          lea    0x18(%rsp),%rbp
  400f3a:       eb db                   jmp    400f17 <phase_2+0x1b>
  400f3c:       48 83 c4 28             add    $0x28,%rsp
  400f40:       5b                      pop    %rbx
  400f41:       5d                      pop    %rbp
  400f42:       c3                      retq
```
从read_six_numbers>字面意思看这里读取了6个数字
```
  400f0a:       83 3c 24 01             cmpl   $0x1,(%rsp)
  400f0e:       74 20                   je     400f30 <phase_2+0x34>
  400f10:       e8 25 05 00 00          callq  40143a <explode_bomb>
```
之后比较了0x1 和(%rsp)，通过je汇编指令可以确认rsp=0x01,否则就爆炸
所以读取的第一个数就是1
之后je
注意：注意%rsp是栈指针,(%rsp)指针指向的值，也就是我们读取的第一个数
```
400f30:       48 8d 5c 24 04          lea    0x4(%rsp),%rbx
400f35:       48 8d 6c 24 18          lea    0x18(%rsp),%rbp
400f3a:       eb db                   jmp    400f17 <phase_2+0x1b>
```
接下来我们就可以lea指令，我被这个指令可是折磨了好久
  lea    0x4(%rsp),%rbx  x就是rbx=&(*rsp+4）//第二个数的地址
rbp=&(*rsp+24）//基指针地址
```
400f17:       8b 43 fc                mov    -0x4(%rbx),%eax
400f1a:       01 c0                   add    %eax,%eax
400f1c:       39 03                   cmp    %eax,(%rbx)
400f1e:       74 05                   je     400f25 <phase_2+0x29>
400f20:       e8 15 05 00 00          callq  40143a <explode_bomb>
```
mov    -0x4(%rbx),%eax 则是 eax =*(rbx-4)//得到第一个数的值，也就是1

接下来就简单了，eax*2= （%rbx）
rbx=2
第二个数是2
```
400f25:       48 83 c3 04             add    $0x4,%rbx
400f29:       48 39 eb                cmp    %rbp,%rbx
400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>
400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>
```
继续破解第三个数
rbx=4
rbx+=4
24=rbp!=rax=8
可以发现这实际上是个循环，遍历这6个数
```
400f25:       48 83 c3 04             add    $0x4,%rbx
400f29:       48 39 eb                cmp    %rbp,%rbx
400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>
400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>
```
eax=*(rbx-4)//前一个数
eax*2=*rbx=第三个数的值
这样一看答案很明显了，这是个等比数列
```
400f17:       8b 43 fc                mov    -0x4(%rbx),%eax
400f1a:       01 c0                   add    %eax,%eax
400f1c:       39 03                   cmp    %eax,(%rbx)
400f1e:       74 05                   je     400f25 <phase_2+0x29>
400f20:       e8 15 05 00 00          callq  40143a <explode_bomb>
```
这个字符串答案是 "1 2 4 8 16 32"

##The Third
```
0000000000400f43 <phase_3>:
400f43:       48 83 ec 18             sub    $0x18,%rsp
400f47:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
400f4c:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
400f51:       be cf 25 40 00          mov    $0x4025cf,%esi
400f56:       b8 00 00 00 00          mov    $0x0,%eax
400f5b:       e8 90 fc ff ff          callq  400bf0 <__isoc99_sscanf@plt>
400f60:       83 f8 01                cmp    $0x1,%eax
400f63:       7f 05                   jg     400f6a <phase_3+0x27>
400f65:       e8 d0 04 00 00          callq  40143a <explode_bomb>
400f6a:       83 7c 24 08 07          cmpl   $0x7,0x8(%rsp)
400f6f:       77 3c                   ja     400fad <phase_3+0x6a>
400f71:       8b 44 24 08             mov    0x8(%rsp),%eax
400f75:       ff 24 c5 70 24 40 00    jmpq   *0x402470(,%rax,8)
400f7c:       b8 cf 00 00 00          mov    $0xcf,%eax
400f81:       eb 3b                   jmp    400fbe <phase_3+0x7b>
400f83:       b8 c3 02 00 00          mov    $0x2c3,%eax
400f88:       eb 34                   jmp    400fbe <phase_3+0x7b>
400f8a:       b8 00 01 00 00          mov    $0x100,%eax
400f8f:       eb 2d                   jmp    400fbe <phase_3+0x7b>
400f91:       b8 85 01 00 00          mov    $0x185,%eax
400f96:       eb 26                   jmp    400fbe <phase_3+0x7b>
400f98:       b8 ce 00 00 00          mov    $0xce,%eax
400f9d:       eb 1f                   jmp    400fbe <phase_3+0x7b>
400f9f:       b8 aa 02 00 00          mov    $0x2aa,%eax
400fa4:       eb 18                   jmp    400fbe <phase_3+0x7b>
400fa6:       b8 47 01 00 00          mov    $0x147,%eax
400fab:       eb 11                   jmp    400fbe <phase_3+0x7b>
400fad:       e8 88 04 00 00          callq  40143a <explode_bomb>
400fb2:       b8 00 00 00 00          mov    $0x0,%eax
400fb7:       eb 05                   jmp    400fbe <phase_3+0x7b>
400fb9:       b8 37 01 00 00          mov    $0x137,%eax
400fbe:       3b 44 24 0c             cmp    0xc(%rsp),%eax
400fc2:       74 05                   je     400fc9 <phase_3+0x86>
400fc4:       e8 71 04 00 00          callq  40143a <explode_bomb>
400fc9:       48 83 c4 18             add    $0x18,%rsp
400fcd:       c3                      retq
```
gdb 查看0x4025cf
```
(gdb) p/s (char*)0x4025cf
$3 = 0x4025cf "%d %d"
(gdb)
```
这是格式化字符串
那<__isoc99_sscanf@plt>很可能是scanf了
```
400f60:       83 f8 01                cmp    $0x1,%eax
400f63:       7f 05                   jg     400f6a <phase_3+0x27>
400f65:       e8 d0 04 00 00          callq  40143a <explode_bomb>
```
上面这段汇编很容易判断0x1>%eax,即%eax=0
```
400f6a:       83 7c 24 08 07          cmpl   $0x7,0x8(%rsp)
400f6f:       77 3c                   ja     400fad <phase_3+0x6a>
400f71:       8b 44 24 08             mov    0x8(%rsp),%eax
400f75:       ff 24 c5 70 24 40 00    jmpq   *0x402470(,%rax,8)
```
```
400fad:       e8 88 04 00 00          callq  40143a <explode_bomb>
```
接下来可以从上面2段汇编判断0x7<=%rsp+8，%rsp>=-1
继续执行
eax=*rsp+8
执行 *0x402470(,%rax,8)？？
跳转0x402470+8*%rax？
在看下面一长串move jump，这是switch？
每一个跳转的分支最后都执行
```
400fbe:       3b 44 24 0c             cmp    0xc(%rsp),%eax
  400fc2:       74 05                   je     400fc9 <phase_3+0x86>
```
这是验证是否相等
那么正确答案就挺多了
我随便挑一个,这个好算点
取%rax=3    jump     0x402470+0x24              
400f8a:       b8 00 01 00 00          mov    $0x100,%eax
400f8f:       eb 2d                   jmp    400fbe <phase_3+0x7b>
得rax=3，eax=256
答案就是3 256
唯一奇怪的是偏移，不能理解0x402470+8*%rax，会多出4偏移