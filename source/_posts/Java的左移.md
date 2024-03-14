---
title: Java的左移(<<)，右移(>>)，无符号右移(>>>)，或与非(&|^~)
tags: Java
comments: false
date: 2021-11-13 15:01:48
---
不常用，所以给自己记一下，做二进制计算的时候很好用。
<!--more-->
```java
public class Demo {

    public static void main(String[] args) {

        /**
         * << 左移运算符，num << 1,相当于num乘以2
         *
         */

        //左移： 10 << 1 = 1010 变成 20
        System.out.println("左移： 10 << 1 = " + Integer.toBinaryString(10) + " 变成 " + (10 << 1));

        //左移： 1 << 10 = 1 变成 1024
        System.out.println("左移： 1 << 10 = " + Integer.toBinaryString(1) + " 变成 " + (1 << 10));

        /**
         * >> 右移运算符，num >> 1,相当于num除以2
         *
         */

        //右移： -4 >> 2 = 11111111111111111111111111111100 变成 -1
        System.out.println("右移： -4 >> 2 = " + Integer.toBinaryString(-4) + " 变成 " + (-4 >> 2));

        //右移： 8 >> 2 = 1000 变成 2
        System.out.println("右移： 8 >> 2 = " + Integer.toBinaryString(8) + " 变成 " + (8 >> 2));

        /**
         * >>> 无符号右移，忽略符号位，空位都以0补齐
         * 当为正整数时是和右移结果一样的，但是为负数时，结果就会不一样了
         */

        //无符号右移： -4 >>> 2 = 11111111111111111111111111111100 变成 1073741823
        System.out.println("无符号右移： -4 >>> 2 = " + Integer.toBinaryString(-4) + " 变成 " + (-4 >>> 2));

        //无符号右移： 8 >>> 2 = 1000 变成 2
        System.out.println("无符号右移： 8 >>> 2 = " + Integer.toBinaryString(8) + " 变成 " + (8 >>> 2));


        System.out.println("————————————————————————");
        System.out.println("————————————————————————");

        //101
        System.out.println(Integer.toBinaryString(5));
        //011
        System.out.println(Integer.toBinaryString(3));

        //只有两位都为1，结果才为1
        //位与： 5 & 3 = 1 (1)
        System.out.println("位与： 5 & 3 = " + (5 & 3) + " (" + Integer.toBinaryString(5 & 3) + ")");

        //只要有一位是1，结果就是1
        //位或： 5 | 3 = 7 (111)
        System.out.println("位或： 5 | 3 = " + (5 | 3) + " (" + Integer.toBinaryString(5 | 3) + ")");

        //只有两位一个是1，一个是0，结果才是1
        //异或： 5 ^ 3 = 6 (110)
        System.out.println("异或： 5 ^ 3 = " + (5 ^ 3) + " (" + Integer.toBinaryString(5 ^ 3) + ")");

        //与全1做异或，然后取反
        //异或非： ~5 = -6 (11111111111111111111111111111010)
        System.out.println("异或非： ~5 = " + (~5) + " (" + Integer.toBinaryString(~5) + ")");
        

        System.out.println("二进制转十进制： " + Integer.valueOf("10101", 2));
        System.out.println("十进制转二进制： " + Integer.toBinaryString(21));
        
}

```