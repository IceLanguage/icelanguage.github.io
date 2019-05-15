---
layout: page
title: 《Fluid Engine Development》 学习笔记1-求解线性方程组
category: 
    - blogs


---
我个人对基于物理的动画很感兴趣，最近在尝试阅读《Fluid Engine Development》，由于内容涉及太多的数学问题，而单纯学习数学又过于枯燥，难以坚持学习（我中途放弃好多次了），打算尝试通过编写博客总结知识的学习方法来学习。

在计算数值问题时，我们经常遇到线性方程，比如基于网格的流体模拟在求解扩散和压强，需要求解线性方程组。

## 线性方程组

线性方程组 

$ \begin{matrix}2 * x - y =3 \\\-x  + 2 * y = 6\end{matrix}​$​

可以转换成形如 $A \times x=b​$ 的矩阵矢量形式，转换结果如下

$\left[
\begin{matrix}
2 & -1 \\\
-1 & 2 
\end{matrix}
\right] \left[
\begin{matrix}
x \\\
y 
\end{matrix}
\right]  = \left[
\begin{matrix}
3 \\\
6
\end{matrix}
\right] ​$

其中A是系统矩阵，x 为所求解，b是一个矢量，也就是线性方程的常数项

至于如何求解x，通常我们是通过 是让等式两边乘以系统矩阵的逆矩阵

## 高斯消元法

求解线性方程组我们通常使用[高斯消元法](https://en.wikipedia.org/wiki/Gaussian_elimination)来求解逆矩阵，这种方法虽然足够直接，然而把高斯消元法作为一种算法来看待，这种算法的时间复杂度达到了惊人的 $O(n^3)​$,n是矩阵的尺寸，由此我们可以明确高斯消元法并不适合有着许多数值问题的较大系统。

如果不选用高斯消元法直接求解逆矩阵，如果不通过高斯消元法，我们又该通过什么方法计算逆矩阵呢？《Fluid Engine Development》给出了就4种不那么直接的方法。通过对解的猜测和多次迭代得到近似解

## 雅可比方法

[雅克比方法（Jacobi方法）](https://en.wikipedia.org/wiki/Jacobi_method)是用于确定对角占优的解的迭代算法。求解每个对角元素，并插入近似值。然后迭代该过程直到它收敛

使用雅克比方法，需要将$A \times x=b​$ 转换成 $（D + R） \times x=b​$ （矩阵A被拆成对角矩阵D和矩阵R）

所以易得解${x}^{(k+1)} = D^{-1} (\mathbf{b} - R \mathbf{x}^{(k)})$ k是迭代的次数

同样作为矢量的解的每一个元素可以通过$ x^{(k+1)}_i  = \frac{1}{a_{ii}} \left(b_i -\sum_{j\ne i}a_{ij}x^{(k)}_j\right),\quad i=1,2,\ldots,n.​$计算得到

## Gauss-Seidel法

Gauss-Seidel方法和jacobi方法有些像

将$A \times x=b​$ 转换成 $（L + U） \times x=b​$ （矩阵A被拆成包含对角部分的下三角形和又矩阵A上三角形部分构成的矩阵U）

具体如下

$\left[
\begin{matrix}
2 & -1 \\\
-1 & 2 
\end{matrix}
\right]= \left[
\begin{matrix}
2 & 0 \\\
-1 & 2
\end{matrix}
\right]+ \left[
\begin{matrix}0 & -1 \\\
0 & 0
\end{matrix}
\right] ​$

通过这种形式的转换我们可以轻易的发现解的第一个元素能轻易的被计算出来

$  x^{(k+1)}_1  = \frac{1}{a_{11}} ) $

将上式代入第二行，我们可以得到

$x^{(k+1)}_2  = \frac{1}{a_{22}} \left(b_i -a_{21}x_{1} ^{k+1}-\sum_{j>1}a_{1j}x^{(k)}_j\right)$

然后可以得到同样的迭代解 

$x^{(k+1)}_i  = \frac{1}{a_{ii}} \left(b_i - \sum_{j=1}^{i-1}a_{ij}x^{(k+1)}_j - \sum_{j=i+1}^{n}a_{ij}x^{(k)}_j \right),\quad i=1,2,\dots,n.​$

## 梯度下降法

梯度下降法将线性方程组求解问题转换成求解最小值问题

 $F(x) =(Ax - b)^{2}​$

如果x有解，则F(x) = 0,如果无解，则可以一直迭代直到x有解，

如果从x1开始，沿着梯度下降，逐步迭代来靠近零点

![1557900682959](https://raw.githubusercontent.com/IceLanguage/icelanguage.github.io/master/images/FoxitPhantomPDF_2019-05-15_14-11-18.png)

## 共轭梯度法

相对梯度下降法，[共轭梯度法](https://en.wikipedia.org/wiki/Conjugate_gradient_method)不使用梯度方向迭代，而是使用共轭方向

何为共轭，如果 $a  *( A b) = 0$ 则我们称向量a，b关于矩阵A 共轭

## 

## 预处理共轭梯度法

在此基础上，我们可以进一步加速： 这个方法称为预处理共轭梯度法。大体的思想是，引入一个preconditioning filter到系统中:

$M^{-1}Ax = M^{-1}b​$

具体算法实现如下

![](<https://raw.githubusercontent.com/IceLanguage/icelanguage.github.io/master/images/FoxitPhantomPDF_2019-05-15_17-15-59.png>)

d就是新的共轭方向，alpha的推导参考[链接](https://en.wikipedia.org/wiki/Conjugate_gradient_method)