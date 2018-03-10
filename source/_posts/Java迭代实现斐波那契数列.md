---
title: Java迭代实现斐波那契数列
date: 2017-08-05
tags: 递归
categories: 算法
---

**剑指offer第九题Java实现**
题目：
大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项。


<!-- more -->


```Java
public class Test9 {
    public static void main(String[] args) {
        Test9 test9 = new Test9();
        System.out.println(test9.Fibonacci(390000));
        System.out.println(test9.Fibonacci3(39));
    }


    /**
     * 为什么不采用递归？因为递归实际是大量调用自身，当数量足够大的时候，需要同时保存成千上百个调用记录，容易发生内存溢出。
     * 怎么优化？
     * 1. 采用尾递归，但是Java并没有基于尾递归进行优化，也就是说Java中采用递归还是无法避免很容易发生"栈溢出"错误（stack overflow）。
     * 因为尾递归都是位于调用函数的最后一行，此时可以删除以前所保存的函数内变量，想当于每次只调用了一个函数。
     * 2. 采用迭代
     *
     * @param n
     * @return
     */
    public int Fibonacci(int n) {
        if (n <= 0) {
            return 0;
        }
        int f1 = 0, f2 = 1;
        for (int i = 1; i <= n; i++) {
            f1 = f1 + f2;
            f2 = f1 - f2;
        }
        // 下面这种写法更为巧妙
//        while (n-- > 0) {
//            f1 = f1 + f2;
//            f2 = f1 - f2;
//        }
        return f1;
    }


    /**
     * 采用递归的方式
     *
     * @param n
     * @return
     */
    public int Fibonacci3(int n) {
        if (n <= 0) {
            return 0;
        }
        if (n == 1 || n == 2) {
            return 1;
        } else {
            return Fibonacci3(n - 1) + Fibonacci3(n - 2);
        }
    }
}
```



**参考文档**
1. [尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)
