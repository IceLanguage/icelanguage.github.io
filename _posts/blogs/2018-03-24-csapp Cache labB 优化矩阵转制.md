---
layout: page
title: 计算机基础篇-Cache labB 优化矩阵转制
category: 
    - blogs
---

## 题目要求和任务

### 要求

#### 测试的矩阵尺寸：

32 x 32
64 x 64
61 x 67

#### 缓存的指标：

有 1KB 大小的缓存

是 directly mapped，也就是 E=1

Block size 为 32 字节，也就是 b=5

一共有 32 个 set，也就是 s=5

### 任务

在trans.c的transpose_submit

## 原理阐述
cache与主存之间的数据传输和和cache与cache之间的数据传输都是以块为最小单位，块里面保存着多个相邻地址的数据

因为块保存着主存相邻的数据，按相邻的顺序调用cache，就能提高cache的命中率，减少时钟周期。
比如遍历一个int[4][4]，按b[0][1]、b[0][2]、b[0][3]顺序遍历，效率要比按b[0][1]、b[1][1]、b[2][1]遍历效率高

以这个思路我们要在transpose_submit中尽可能优化程序，提高cache的命中率
## 32*32
由于block能存8个int,题目还限制了变量数量，所以数据块最好是以它为单位的，这样能尽可能利用block
```
int i,j,tmp,index;
    int row_Block,col_Block;
    if(M==32)
    {
        for(row_Block = 0;row_Block < N ;row_Block+=8){
            for(col_Block =0 ;col_Block < M; col_Block+=8){
                for(i=row_Block ; i<row_Block+8;i++){
                    for(j=col_Block;j<col_Block+8;j++){
                        if(i!=j){
                            //处理对角线
                            B[j][i] = A[i][j];
                        }else{
                            tmp = A[i][j];                 
                            index = i;
                        }
                    }
                    if(col_Block == row_Block){
                        //处理对角             
                        B[index][index] = tmp;
                    }
                }
            }
        }
    }
```
## 61*67
61*67也很简单，加个限制条件就能解决问题
```
 for(row_Block = 0;row_Block < N ;row_Block+=16){
            for(col_Block =0 ;col_Block < M; col_Block+=16){
                for(i=row_Block ; i<row_Block+16&& (i<N);i++){
                    for(j=col_Block;j<col_Block+16&&(j<M);j++){
                        if(i!=j){
                            B[j][i] = A[i][j];
                        }else{
                            tmp = A[i][j];                 
                            index = i;
                        }
                    }
                    if(col_Block == row_Block){             
                        B[index][index] = tmp;
                    }
                }
            }
        }
```

## 64*64
真正麻烦的是64*64，想不到要求那么高

64 × 64 (M = 64, N = 64)
此时，数组一行有64个int，即8个block，所以每四行就会填满一个cache，即两个元素相差四行就会发生冲突

如果我们使用4*4的blocking，的确可以成功，但拿不到满分
```
for(row_Block = 0;row_Block < N ;row_Block+=4){
            for(col_Block =0 ;col_Block < M; col_Block+=4){
                for(i=row_Block ; i<row_Block+4;i++){
                    for(j=col_Block;j<col_Block+4;j++){
                        if(i!=j){
                            B[j][i] = A[i][j];
                        }else{
                            tmp = A[i][j];                  
                            index = i;
                        }
                    }
                    if(col_Block == row_Block){             
                        B[index][index] = tmp;
                    }
                }
            }
        }
```
满分参考这位大神https://www.cnblogs.com/liqiuhao/p/8026100.html?utm_source=debugrun&utm_medium=referral