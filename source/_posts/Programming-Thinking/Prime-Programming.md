---
title: Prime - 程序员的修养
date: 2016-09-28
categories: 编程思考
---
# 求质数算法的N种境界
[求质数算法的N种境界[1] - 试除法和初级筛法](https://program-think.blogspocomt./2011/12/prime-algorithm-1.html)

# 过程
尽管题目并没有要我们写一个最优的算法，但是身为一个程序员，优化应该是一种习惯，在编程的过程中，随着思考进行优化。
如果你只能想出一个最简单的方法，难道你会有什么竞争力吗？
<!-- more -->
## 1 最容易想到
最开始我用的就是这个方法，可以说这是最简单的一种方法了，而且最开始，我就是想的这种方法，说明：我没有对这个问题进行思考，没有去优化它，而作为一个程序员，如何提高效率是拿到一个问题首先要思考的事情。
``` java
       public static void main(String[] args) {
        long startTime = System.currentTimeMillis();   //获取开始时
        int num = 0;
        for (int i = 2; i < 2000000; i++) {
            boolean flag = true;
            for (int j = 2; j < i - 1; j++)
                if (i % j == 0) {
                    flag = false;
                    break;
                }
            if (flag) {
                System.out.println(i);
                num++;
            }
        }
        System.out.println("The number is " + num);
        long endTime = System.currentTimeMillis(); //获取结束时间
        System.out.println("程序运行时间： " + (endTime - startTime) + "ms");
    }
```

> 时间太长，已经不能计算。


## 2 奇数，$\sqrt{n}$
思考后发现
1. 素数一定是奇数
2. 若 n=ab 是个合数（其中 a 与 b ≠ 1）, 则其中一个约数 a 或 b 必定至大为  $\sqrt{n}$.
``` java
        public static void main(String[] args) {
        long startTime = System.currentTimeMillis();   //获取开始时
        int num = 1;
        for (int i = 3; i < 2000000; i += 2) {
            boolean flag = true;
            for (int j = 2; j <= (int) Math.sqrt(i); j++)
                if (i % j == 0) {
                    flag = false;
                    break;
                }
            if (flag) {
                System.out.println(i);
                num++;
            }
        }
        System.out.println("The number is " + num);
        long endTime = System.currentTimeMillis(); //获取结束时间
        System.out.println("程序运行时间： " + (endTime - startTime) + "ms");
    }
```

> The number is 148933

程序运行时间： 2820ms

## 3数学知识的运用


> **算术基本定理** :

每个大于1的整数均可写成一个以上的素数之乘积，且除了质约数的排序不同外是唯一的
``` java
    public static void main(String[] args) {
        int num = 0;
        for (int i = 2; i < 200; i++) {
            boolean flag = true;
            for (int j = 2; j < i - 1; j++)
                if (i % j == 0) {
                    flag = false;
                    break;
                }
            if (flag) {
                System.out.println(i);
                num++;
            }
        }
        System.out.println("The number is " + num);
    }
```

> The number is 148933
> 程序运行时间： 1773ms


速度果然有很大的提高。
以时间换空间。
