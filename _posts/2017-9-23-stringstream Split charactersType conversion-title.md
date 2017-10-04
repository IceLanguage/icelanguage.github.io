---
layout: post
title: LeetCode stringstream分割字符并完成类型转换
category: c++ LeetCode stringstream SplitcharactersTypeconversion
tag: C++ LeetCode stringstream Split characters Type conversion 分割字符 类型转换
comments: false
---


LeetCode上看到了一个非常牛逼的技巧，可以轻松分割不同的字符，并完成类型转换，那就是stringstream
[原题](https://leetcode.com/problems/complex-number-multiplication/description/)
[原题解答](https://discuss.leetcode.com/topic/84382/c-using-stringstream%20%E5%8E%9F%E9%A2%98%E8%A7%A3%E7%AD%94)

```
Input: "1+1i", "1+1i"
Output: "0+2i"
Explanation: (1 + i) * (1 + i) = 1 + i2 + 2 * i = 2i, and you need convert it to the form of 0+2i.
```
原题是给2个复数和负号组成的字符串，要求返回字符串形态的解答，这种方法极大简化了操作

以下是解答
```
class Solution {
public:
    string complexNumberMultiply(string a, string b) {
        int ra, ia, rb, ib;
        char buff;
        stringstream aa(a), bb(b), ans;
        aa >> ra >> buff >> ia >> buff;//拆分
        bb >> rb >> buff >> ib >> buff;
        ans << ra*rb - ia*ib << "+" << ra*ib + rb*ia << "i";//组合
        return ans.str();
    }
};
```
利用stringstream构造字符串流的时候，空格+-会成为字符串参数的内部分界，例子中对ra,ia对象的输入"赋值"操作证明了这一点，字符串的空格成为了整型数据与符号数据的分解点，利用分界获取的方法我们事实上完成了字符串到整型对象与符号对象的拆分转换过程