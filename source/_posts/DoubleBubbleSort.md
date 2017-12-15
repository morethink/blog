---
title: Java - 双冒泡法排序
date: 2016-10-02
tags: 面试
categories: 算法
---

# 最开始的代码
我采用的是我原来进行快速排序所用的方法，一直做不出来。
为什么我会采用原来快速排序的方法？因为我的记忆中好像就是这样的，因此我根据记忆中的快速排序在进行改变，然而，却无法真正的写出双冒泡排序算法，所以，真正的学习是，即使随着时间的流逝，你已经不记得某些东西了，但是，你可以凭借理解在重新写出来。

<!-- more -->

``` java
public static void sort(int[] num, int l, int r) {
        int i = l;
        int j = l + 1;
        int key = num[l];
        while () {
            while (num[j] > key)
                j++;
            if (j < r)
                num[i] = num[j];
            while (i < j && num[i] < key)
                i++;
            if (i < j)
                num[j++] = num[i];
        }
        num[i] = key;
        for (int n : num)
            System.out.print(n + " ");
    }
```
# 改良后的
[快速排序法(双冒泡排序法)](http://www.cnblogs.com/sjxbg/p/5743838.html)

[冒泡排序 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%86%92%E6%B3%A1%E6%8E%92%E5%BA%8F)
``` java
    public static void sort(int[] num, int l, int r) {
        if (l < r) {
            int i = l;
            int j = l + 1;
            while (j < r) {
                if (num[j] < num[l]) {
                    i++;
                    int temp = num[i];
                    num[i] = num[j];
                    num[j] = temp;
                }
                j++;
            }
            //System.out.println(i);
            int temp = num[i];
            num[i] = num[l];
            num[l] = temp;
            sort(num, l, i);
            sort(num, i + 1, r);
        }
    }
```

# 大数据排序
当我将数组大小增大到3000000后，出现异常。
[ 十道海量数据处理面试题与十个方法大总结](http://blog.csdn.net/v_july_v/article/details/6279498)

# 交换两个数
java 没有指针，所以不可以使用swap函数进行交换，采用下面的方式
```java
 int temp = num[i];
 num[i] = num[j];
 num[j] = temp;
```
也有其他的方式：
[交换两个整数的三种实现方法(C/C++)](http://www.51testing.com/html/38/225738-220963.html)

# 单链表的排序
通过尾插法建立一个单链表，然后进行双冒泡排序

```java
class Student {
    int num;
    Student next;
}

public class LinkedList {
    public static void main(String[] args) {
        Student head = null;
        for (int i = 1; i <= 50; i++) {
            head = addBack(head, (int) (Math.random() * 100));
        }
        show(head);
        sort(head, null);
        System.out.println();
        show(head);
    }

    public static Student addBack(Student head, int num) {
        Student newStudent = new Student();
        newStudent.num = num;
        newStudent.next = null;
        Student student = head;
        if (head == null) {
            head = newStudent;
            return head;
        }
        while (student.next != null) {
            student = student.next;
        }
        student.next = newStudent;

        return head;
    }

    public static void show(Student student) {
        while (student != null) {
            System.out.println(student.num);
            student = student.next;
        }

    }

    public static void sort(Student start, Student end) {
        if (start != end) {
            Student p1 = start;
            Student p2 = p1.next;
            while (p2 != end) {
                if (p2.num < start.num) {
                    p1 = p1.next;
                    int temp = p1.num;
                    p1.num = p2.num;
                    p2.num = temp;
                }
                p2 = p2.next;
            }
            int temp = p1.num ;
            p1.num =start.num ;
            start.num =temp;
            sort(start, p1);
            sort(p1.next, null);
        }

    }

}
```
