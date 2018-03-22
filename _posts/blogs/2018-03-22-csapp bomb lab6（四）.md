---
layout: page
title: 计算机基础篇-bomb lab6(四）
category: 
    - blogs
---
最后一个bomb lab了，go！go！

汇编
```
00000000004010f4 <phase_6>:
  4010f4:       41 56                   push   %r14
  4010f6:       41 55                   push   %r13
  4010f8:       41 54                   push   %r12
  4010fa:       55                      push   %rbp
  4010fb:       53                      push   %rbx
  4010fc:       48 83 ec 50             sub    $0x50,%rsp
  401100:       49 89 e5                mov    %rsp,%r13
  401103:       48 89 e6                mov    %rsp,%rsi
  401106:       e8 51 03 00 00          callq  40145c <read_six_numbers>
  40110b:       49 89 e6                mov    %rsp,%r14
  40110e:       41 bc 00 00 00 00       mov    $0x0,%r12d
  401114:       4c 89 ed                mov    %r13,%rbp
  401117:       41 8b 45 00             mov    0x0(%r13),%eax
  40111b:       83 e8 01                sub    $0x1,%eax
  40111e:       83 f8 05                cmp    $0x5,%eax
  401121:       76 05                   jbe    401128 <phase_6+0x34>
  401123:       e8 12 03 00 00          callq  40143a <explode_bomb>
  401128:       41 83 c4 01             add    $0x1,%r12d
  40112c:       41 83 fc 06             cmp    $0x6,%r12d
  401130:       74 21                   je     401153 <phase_6+0x5f>
  401132:       44 89 e3                mov    %r12d,%ebx
  401135:       48 63 c3                movslq %ebx,%rax
  401138:       8b 04 84                mov    (%rsp,%rax,4),%eax
  40113b:       39 45 00                cmp    %eax,0x0(%rbp)
  40113e:       75 05                   jne    401145 <phase_6+0x51>
  401140:       e8 f5 02 00 00          callq  40143a <explode_bomb>
  401145:       83 c3 01                add    $0x1,%ebx
  401148:       83 fb 05                cmp    $0x5,%ebx
  40114b:       7e e8                   jle    401135 <phase_6+0x41>
  40114d:       49 83 c5 04             add    $0x4,%r13
  401151:       eb c1                   jmp    401114 <phase_6+0x20>
  401153:       48 8d 74 24 18          lea    0x18(%rsp),%rsi
  401158:       4c 89 f0                mov    %r14,%rax
  40115b:       b9 07 00 00 00          mov    $0x7,%ecx
  401160:       89 ca                   mov    %ecx,%edx
  401162:       2b 10                   sub    (%rax),%edx
  401164:       89 10                   mov    %edx,(%rax)
  401166:       48 83 c0 04             add    $0x4,%rax
  40116a:       48 39 f0                cmp    %rsi,%rax
  40116d:       75 f1                   jne    401160 <phase_6+0x6c>
  40116f:       be 00 00 00 00          mov    $0x0,%esi
  401174:       eb 21                   jmp    401197 <phase_6+0xa3>
  401176:       48 8b 52 08             mov    0x8(%rdx),%rdx
  40117a:       83 c0 01                add    $0x1,%eax
  40117d:       39 c8                   cmp    %ecx,%eax
  40117f:       75 f5                   jne    401176 <phase_6+0x82>
  401181:       eb 05                   jmp    401188 <phase_6+0x94>
  401183:       ba d0 32 60 00          mov    $0x6032d0,%edx
  401188:       48 89 54 74 20          mov    %rdx,0x20(%rsp,%rsi,2)
  40118d:       48 83 c6 04             add    $0x4,%rsi
  401191:       48 83 fe 18             cmp    $0x18,%rsi
  401195:       74 14                   je     4011ab <phase_6+0xb7>
  401197:       8b 0c 34                mov    (%rsp,%rsi,1),%ecx
  40119a:       83 f9 01                cmp    $0x1,%ecx
  40119d:       7e e4                   jle    401183 <phase_6+0x8f>
  40119f:       b8 01 00 00 00          mov    $0x1,%eax
  4011a4:       ba d0 32 60 00          mov    $0x6032d0,%edx
  4011a9:       eb cb                   jmp    401176 <phase_6+0x82>
  4011ab:       48 8b 5c 24 20          mov    0x20(%rsp),%rbx
  4011b0:       48 8d 44 24 28          lea    0x28(%rsp),%rax
  4011b5:       48 8d 74 24 50          lea    0x50(%rsp),%rsi
  4011ba:       48 89 d9                mov    %rbx,%rcx
  4011bd:       48 8b 10                mov    (%rax),%rdx
  4011c0:       48 89 51 08             mov    %rdx,0x8(%rcx)
  4011c4:       48 83 c0 08             add    $0x8,%rax
  4011c8:       48 39 f0                cmp    %rsi,%rax
  4011cb:       74 05                   je     4011d2 <phase_6+0xde>
  4011cd:       48 89 d1                mov    %rdx,%rcx
  4011d0:       eb eb                   jmp    4011bd <phase_6+0xc9>
  4011d2:       48 c7 42 08 00 00 00    movq   $0x0,0x8(%rdx)
  4011d9:       00
  4011da:       bd 05 00 00 00          mov    $0x5,%ebp
  4011df:       48 8b 43 08             mov    0x8(%rbx),%rax
  4011e3:       8b 00                   mov    (%rax),%eax
  4011e5:       39 03                   cmp    %eax,(%rbx)
  4011e7:       7d 05                   jge    4011ee <phase_6+0xfa>
  4011e9:       e8 4c 02 00 00          callq  40143a <explode_bomb>
  4011ee:       48 8b 5b 08             mov    0x8(%rbx),%rbx
  4011f2:       83 ed 01                sub    $0x1,%ebp
  4011f5:       75 e8                   jne    4011df <phase_6+0xeb>
  4011f7:       48 83 c4 50             add    $0x50,%rsp
  4011fb:       5b                      pop    %rbx
  4011fc:       5d                      pop    %rbp
  4011fd:       41 5c                   pop    %r12
  4011ff:       41 5d                   pop    %r13
  401201:       41 5e                   pop    %r14
  401203:       c3                      retq
```
不愧是压轴题，真的长
r13=rsp
rsi=rsp

read_six_numbers获取6个数
这个函数在phase_2中在见过，作用是将输入的6个数字存储在从rsi所存储地址开始的连续24个字节中，这里由于rsi已经被赋予了rsp的值，我们输入的6个数字被存储在rsp标记的连续24个字节中

r14=rsp
r12d=0
rbp=r13=rsp
eax=r13
eax+=1//eax=r14

```
40111e:       83 f8 05                cmp    $0x5,%eax
401121:       76 05                   jbe    401128 <phase_6+0x34>
401123:       e8 12 03 00 00          callq  40143a <explode_bomb>
```
然后比较5和eax的值

eax需要小于等于5
```
401128:       41 83 c4 01             add    $0x1,%r12d
40112c:       41 83 fc 06             cmp    $0x6,%r12d
401130:       74 21                   je     401153 <phase_6+0x5f>
401132:       44 89 e3                mov    %r12d,%ebx
401135:       48 63 c3                movslq %ebx,%rax
401138:       8b 04 84                mov    (%rsp,%rax,4),%eax
40113b:       39 45 00                cmp    %eax,0x0(%rbp)
40113e:       75 05                   jne    401145 <phase_6+0x51>
401140:       e8 f5 02 00 00          callq  40143a <explode_bomb>
```
%r12d+=1

比较6和%r12d

假设不相等

ebx=r12d
rax=ebx
eax=*(esp+4*rax)
比较eax和rbp+0
必须不相等
```
401145:       83 c3 01                add    $0x1,%ebx
401148:       83 fb 05                cmp    $0x5,%ebx
40114b:       7e e8                   jle    401135 <phase_6+0x41>
```
ebx+=1
比较ebx和5
哎，发现是循环不相等就继续加

ebx==5后
```
40114d:       49 83 c5 04             add    $0x4,%r13
401151:       eb c1                   jmp    401114 <phase_6+0x20>
```
r13+=4
然后又回去，这是个循环嵌套

这个循环嵌套做了什么，只是让r11，r16指向了第一到第六个数

然后下一段
```
401153:       48 8d 74 24 18          lea    0x18(%rsp),%rsi
401158:       4c 89 f0                mov    %r14,%rax
40115b:       b9 07 00 00 00          mov    $0x7,%ecx
401160:       89 ca                   mov    %ecx,%edx
401162:       2b 10                   sub    (%rax),%edx
401164:       89 10                   mov    %edx,(%rax)
401166:       48 83 c0 04             add    $0x4,%rax
40116a:       48 39 f0                cmp    %rsi,%rax
40116d:       75 f1                   jne    401160 <phase_6+0x6c>
```
rsi=*rsp+24;
rax=r14;
ecx=7;
edx=ecx=7;
edx-=eax;
rax=edx;
rax+=4;
比较rsi,rax;

不相等，又回去了，这又是个循环，只是将输入的值num变成7-num

下一段
```
40116f:       be 00 00 00 00          mov    $0x0,%esi
401174:       eb 21                   jmp    401197 <phase_6+0xa3>
401176:       48 8b 52 08             mov    0x8(%rdx),%rdx
40117a:       83 c0 01                add    $0x1,%eax
40117d:       39 c8                   cmp    %ecx,%eax
40117f:       75 f5                   jne    401176 <phase_6+0x82>
401181:       eb 05                   jmp    401188 <phase_6+0x94>
401183:       ba d0 32 60 00          mov    $0x6032d0,%edx
401188:       48 89 54 74 20          mov    %rdx,0x20(%rsp,%rsi,2)
40118d:       48 83 c6 04             add    $0x4,%rsi
401191:       48 83 fe 18             cmp    $0x18,%rsi
401195:       74 14                   je     4011ab <phase_6+0xb7>
401197:       8b 0c 34                mov    (%rsp,%rsi,1),%ecx
40119a:       83 f9 01                cmp    $0x1,%ecx
40119d:       7e e4                   jle    401183 <phase_6+0x8f>
40119f:       b8 01 00 00 00          mov    $0x1,%eax
4011a4:       ba d0 32 60 00          mov    $0x6032d0,%edx
4011a9:       eb cb                   jmp    401176 <phase_6+0x82>
```
esi=0；
rcx=rsp+rsi；
然后1和ecx比较；

小于等于1则跳到401183；
edx=*(0x6032d0);//node1?
*(rsp+2*rsi+32)=rdx;
rsi+=4;
rsi和24比较；
如果相等则跳出这个循环，否则将ecx=*（rsp+rsi）继续与1比较

0x6032d0 <node1>:    0x10000014c    0x6032e0 <node2>
0x6032e0 <node2>:    0x2000000a8    0x6032f0 <node3>
0x6032f0 <node3>:    0x30000039c    0x603300 <node4>
0x603300 <node4>:    0x4000002b3    0x603310 <node5>
0x603310 <node5>:    0x5000001dd    0x603320 <node6>
0x603320 <node6>:    0x6000001bb    0x0 
链表？
循环的作用就是，在rsp+32到rsp+72这一段地址上存储6个节点的地址

接下来又是一个循环
```
4011ab: 48 8b 5c 24 20          mov    0x20(%rsp),%rbx  
4011b0: 48 8d 44 24 28          lea    0x28(%rsp),%rax  
4011b5: 48 8d 74 24 50          lea    0x50(%rsp),%rsi  
4011ba: 48 89 d9                mov    %rbx,%rcx  
4011bd: 48 8b 10                mov    (%rax),%rdx #starting a loop  
4011c0: 48 89 51 08             mov    %rdx,0x8(%rcx) # retail the linked list (set every node's tail pointer to the new next node)  
4011c4: 48 83 c0 08             add    $0x8,%rax  
4011c8: 48 39 f0                cmp    %rsi,%rax  
4011cb: 74 05                   je     4011d2 <phase_6+0xde> #exit the loop  
4011cd: 48 89 d1                mov    %rdx,%rcx  
4011d0: eb eb                   jmp    4011bd <phase_6+0xc9>   

```
（好烦啊，汇编的循环看着真反人类）

在4011c0处可以看出，这个循环让新排好的node序列中每个node的指针域指向新链表中的下一个node

```
4011d2: 48 c7 42 08 00 00 00    movq   $0x0,0x8(%rdx)  
4011d9: 00   
4011da: bd 05 00 00 00          mov    $0x5,%ebp  
4011df: 48 8b 43 08             mov    0x8(%rbx),%rax # the start of a for loop  
4011e3: 8b 00                   mov    (%rax),%eax  
4011e5: 39 03                   cmp    %eax,(%rbx)  
4011e7: 7d 05                   jge    4011ee <phase_6+0xfa>  
4011e9: e8 4c 02 00 00          callq  40143a <explode_bomb>  
4011ee: 48 8b 5b 08             mov    0x8(%rbx),%rbx  
4011f2: 83 ed 01                sub    $0x1,%ebp  
4011f5: 75 e8                   jne    4011df <phase_6+0xeb> # the end of a for loop  
4011f7: 48 83 c4 50             add    $0x50,%rsp  
4011fb: 5b                      pop    %rbx  
4011fc: 5d                      pop    %rbp  
4011fd: 41 5c                   pop    %r12  
4011ff: 41 5d                   pop    %r13  
401201: 41 5e                   pop    %r14  
401203: c3                      retq     
```
*(rdx+8)=0；
再一次循环；
rax=*(rbx+8)//下一个节点的地址；
rax=eax；
比较eax和ebx；
在4011e7可以看出，前一个节点的数值必须更大，否则炸了；
然后rbx加上12变成下一个节点的地址，ebp减去1；
这个循环告诉我们，新的链表中每个节点的值都要大于下一个的；

回到回到0x6032d0那片链表;

发现链表重排了，顺序: node3 node4 node5 node6 node1 node2;

所以是4 3 2 1 6 5;
over
```
Starting program: /mnt/c/Windows/System32/ubuntu/csapp/bomb/bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.
Phase 1 defused. How about the next one?
1 2 4 8 16 32
That's number 2.  Keep going!
3 256
Halfway there!
7 0
So you got that one.  Try this one.
9on567
Good work!  On to the next...
4 3 2 1 6 5
Congratulations! You've defused the bomb!
[Inferior 1 (process 24) exited normally]
```

总结：
lab1让我感兴趣了，感觉自己踏向了新世界，感觉自己以后能吹一波逆向工程；
不过lab2差点让我奔溃了，人家老师课设的好，让我深刻理解了lea和mov等汇编指令和自己的菜；
lab4让我纠结好一阵，可能脑子转进死胡同了；
lab6很烦，不过独立做完lab的感觉还是挺好的；
相比datalab这个lab做的还是很爽的