---
layout: page
title: 计算机基础篇-bomb lab(二）
category: 
    - blogs
---
这个递归算了好久，都要放弃了，想不到吃完饭回来一下子做出来，hh

----------
The fourth
```
000000000040100c <phase_4>:
40100c:       48 83 ec 18             sub    $0x18,%rsp
401010:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
401015:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
40101a:       be cf 25 40 00          mov    $0x4025cf,%esi
40101f:       b8 00 00 00 00          mov    $0x0,%eax
401024:       e8 c7 fb ff ff          callq  400bf0 <__isoc99_sscanf@plt>
401029:       83 f8 02                cmp    $0x2,%eax
40102c:       75 07                   jne    401035 <phase_4+0x29>
40102e:       83 7c 24 08 0e          cmpl   $0xe,0x8(%rsp)
401033:       76 05                   jbe    40103a <phase_4+0x2e>
401035:       e8 00 04 00 00          callq  40143a <explode_bomb>
40103a:       ba 0e 00 00 00          mov    $0xe,%edx
40103f:       be 00 00 00 00          mov    $0x0,%esi
401044:       8b 7c 24 08             mov    0x8(%rsp),%edi
401048:       e8 81 ff ff ff          callq  400fce <func4>
40104d:       85 c0                   test   %eax,%eax
40104f:       75 07                   jne    401058 <phase_4+0x4c>
401051:       83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)
401056:       74 05                   je     40105d <phase_4+0x51>
401058:       e8 dd 03 00 00          callq  40143a <explode_bomb>
40105d:       48 83 c4 18             add    $0x18,%rsp
401061:       c3                      retq
```

```
(gdb) p/s (char*) 0x4025cf
$1 = 0x4025cf "%d %d"
(gdb)
```
和lab3一样可以看到格式化字符串和scanf

```
401029:       83 f8 02                cmp    $0x2,%eax
40102c:       75 07                   jne    401035 <phase_4+0x29>
```
```
401035:       e8 00 04 00 00          callq  40143a <explode_bomb>
```
由上面2段汇编可以获得限制条件
%eax=2
```
40102e:       83 7c 24 08 0e          cmpl   $0xe,0x8(%rsp)
401033:       76 05                   jbe    40103a <phase_4+0x2e>
401035:       e8 00 04 00 00          callq  40143a <explode_bomb>
```
同理限制条件0xe<=8+*rsp 第一个数字
即
```
40103a:       ba 0e 00 00 00          mov    $0xe,%edx
40103f:       be 00 00 00 00          mov    $0x0,%esi
401044:       8b 7c 24 08             mov    0x8(%rsp),%edi
401048:       e8 81 ff ff ff          callq  400fce <func4>
```
edx=14
esi=0
edi=*rsp+8
14>=8+*rsp第一个数字
执行函数func4
```
0000000000400fce <func4>:
400fce:       48 83 ec 08             sub    $0x8,%rsp
400fd2:       89 d0                   mov    %edx,%eax
400fd4:       29 f0                   sub    %esi,%eax
400fd6:       89 c1                   mov    %eax,%ecx
400fd8:       c1 e9 1f                shr    $0x1f,%ecx
400fdb:       01 c8                   add    %ecx,%eax
400fdd:       d1 f8                   sar    %eax
400fdf:       8d 0c 30                lea    (%rax,%rsi,1),%ecx
400fe2:       39 f9                   cmp    %edi,%ecx
400fe4:       7e 0c                   jle    400ff2 <func4+0x24>
400fe6:       8d 51 ff                lea    -0x1(%rcx),%edx
400fe9:       e8 e0 ff ff ff          callq  400fce <func4>
400fee:       01 c0                   add    %eax,%eax
400ff0:       eb 15                   jmp    401007 <func4+0x39>
400ff2:       b8 00 00 00 00          mov    $0x0,%eax
400ff7:       39 f9                   cmp    %edi,%ecx
400ff9:       7d 0c                   jge    401007 <func4+0x39>
400ffb:       8d 71 01                lea    0x1(%rcx),%esi
400ffe:       e8 cb ff ff ff          callq  400fce <func4>
401003:       8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
401007:       48 83 c4 08             add    $0x8,%rsp
40100b:       c3                      retq
```
将edx赋值给eax，eax减去esi，然后将eax赋值给ecx，ecx逻辑右移31位（也就是说，它在eax小于0时变为1，否则为0）
eax=14
ecx=0
接着将eax加上ecx，然后eax除以2，将eax+esi的值赋给ecx
然后比较edi,ecx=21
eax=7
ecx=7
如果edi<=ecx
eax=0
比较edi ecx
因为edi<=ecx
esi=ecx+1
再次执行func4
eax+=eax
如果edi>ecx
ecx=esi+eax
再比较edi，ecx
if edi>=ecx
    return *rsp+8
else  edi< ecx
    eax=0
    esi=ecx+1   
可以看出是个递归
尝试用伪c代码来解释
```
func4(edx,esi,edi)
{
    eax=edx/2
    ecx=eax+esi
    if(edi<=ecx)
    {
        eax=0
        if(edi==ecx)
        {
            return eax
        }
        esi=ecx+1
        eax=fun4(edx,edx/2+esi+1,edi)
        eax=2*eax+1
        return eax
    }
   
    edx=ecx-1
    eax=func4(edx/2+esi-1,esi,edi)
    eax=eax*2
    return eax
}
```
发现必须edi==ecx才能正常返回，否则会无止境递归
那么edi=14/2就行了
或者7/2=3，或者3/2=1或者1/2=0
然后由phase_4中下一段汇编可得第二个数是0
```
401051:       83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)
401056:       74 05                   je     40105d <phase_4+0x51>
401058:       e8 dd 03 00 00          callq  40143a <explode_bomb>
```
所以答案是 
7 0或3 0或1 0或0 0
