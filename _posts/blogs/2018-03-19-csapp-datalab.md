---
layout: page
title: 计算机基础篇-datalab
category: 
    - blogs
---

----------
有好几道没吃透，尤其是浮点数，先放着。
做完csapp的datalab后我的唯一感受是：智商被碾压，大家都是怎么想出来的，搞不起搞不起。
```c
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {

  return ~((~x)|(~y));
}
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int getByte(int x, int n) {

  return 0xFF&(x>>(n<<3));

}
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {

  int tmp = ~(((1<<31)>>n)<<1); 
  //获取掩码0xfff..

  return (x >> n)&tmp;
  //利用掩码消除右移导致多出的符号位
}
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
  int tmp=(((0x01<<8|0x01)<<8|0x01)<<8|0x01);
  //获取0x01010101

  int val=x&tmp;
  val+=(x>>1)&tmp;
  val+=(x>>2)&tmp;
  val+=(x>>3)&tmp;
  val+=(x>>4)&tmp;
  val+=(x>>5)&tmp;
  val+=(x>>6)&tmp;
  val+=(x>>7)&tmp;
  //检测出现过的1，并全部加到0,8,16,24上

  val+=val>>16;
  val+=val>>8;
  //继续将高位的1加到低位上

  return val&0xff;
}
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {
  //只有0和-0的二进制中，最高位都不是1
  return ((~(x|(~x+1)))>>31)&1;
}
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1<<31;
}
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
//求x是否在n位补码的范围内
int fitsBits(int x, int n) {
    int tmp=33+~n;//32-n

    //正数满足第n位到32位全为0，负数满足第n位到32位全为1
    return !(((x<<tmp)>>tmp)^x);
}
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) {
    //只考虑负数，正数直接右移n位

    //获取符号位
    int signx=x>>31;  


    //int mask=(1<<n)+(-1);  
    int mask=(1<<n)+(~0);  
    int bias=signx&mask;  

    //负数就加一
    return (x+bias)>>n;  
}
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return 1+~x;
}
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
  //x>>31获取符号位
  //!x判断是否为0
  return !((x>>31)|!x);
}
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  //获取符号位
  int signx = (x>>31)&1;
  int signy = (y>>31)&1;

  //符号位不同 x符号位为1，y符号位为0符号条件
  int signdif = (!signy)&signx;

  //x,y符号位相同，x-y符号位为1
  int signsam = (!(signx^signy))&(((x+(~y))>>31)&1);
  return signdif|signsam;
}
/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
//x二进制里最高位的1在哪里
int ilog2(int x) {
  //参考https://www.zhihu.com/people/ma-shao-nan-89/posts
  int ans = 0;
  ans = ans + ((!!(x>>(16 + ans)))<<4);
  ans = ans + ((!!(x>>(8 + ans)))<<3);
  ans = ans + ((!!(x>>(4 + ans)))<<2);
  ans = ans + ((!!(x>>(2 + ans)))<<1);
  ans = ans + ((!!(x>>(1 + ans)))<<0);

  return ans;
}
/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
   //参考https://www.zhihu.com/people/ma-shao-nan-89/posts和浮点数标准
    unsigned tmp = uf&(0x7fffffff);
    unsigned result = uf^(0x80000000);
    if(tmp>0x7f800000) result=uf;//uf是NaN 返回NaN
    return result; 
}
/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
//先放着，没吃透，只理解了符号位和一知半解的M
//int to float
unsigned float_i2f(int x) {\
  //来源http://blog.csdn.net/u014124795/article/details/38471797
  unsigned sign=0,shiftleft=0,flag=0,tmp;  
  unsigned absx=x;  
  if( x==0 ) return 0;  
  if( x<0 ){  
     sign=0x80000000;  
     absx=-x;  
  }  
  while(1){//Shift until the highest bit equal to 1 in order to normalize the floating-point number  
     tmp=absx;  
     absx<<=1;  
     shiftleft++;  
     if(tmp&0x80000000 ) break;  
  }  

  //我看不懂了
  if( (absx & 0x01ff) > 0x0100 ) flag=1;//***Rounding 1  
  if( (absx & 0x03ff) == 0x0300 ) flag=1;//***Rounding 2  
    
  return sign+(absx>>9)+((127+32-shiftleft)<<23)+flag;  
}
/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
//不理解
unsigned float_twice(unsigned uf) {
   //来源https://www.zhihu.com/people/ma-shao-nan-89/
  unsigned tmp = uf;

  //非规格化的
  if((tmp&0x7f800000)==0){
    tmp = (tmp&0x80000000)|((tmp&0x007fffff)<<1);
  }

  //规格化的
  else if((tmp&0x7f800000)!=0x7f800000){
    tmp+=(1<<23);
    if((tmp&0x7f800000)==0x7f800000){
      tmp=(tmp>>23)<<23;
    }
  }
  return tmp;
}

```