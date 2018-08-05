---
title: Java实现单链表的快速排序和归并排序
date: 2018-02-18
tags: [链表,排序]
categories: 算法
---

本文描述了LeetCode 148题 [sort-list](https://leetcode.com/problems/sort-list/description/) 的解法。

题目描述如下:
Sort a linked list in O(n log n) time using constant space complexity.

题目要求我们在O(n log n)时间复杂度下完成对单链表的排序，我们知道平均时间复杂度为O(n log n)的排序方法有快速排序、归并排序和堆排序。而一般是用数组来实现二叉堆，当然可以用二叉树来实现，但是这么做太麻烦，还得花费额外的空间构建二叉树，于是不采用堆排序。
<!-- more -->
故本文采用快速排序和归并排序来对单链表进行排序。

# 快速排序

在一般实现的快速排序中，我们通过首尾指针来对元素进行切分，下面采用快排的另一种方法来对元素进行切分。

我们只需要两个指针p1和p2，这两个指针均往next方向移动，移动的过程中保持p1之前的key都小于选定的key，p1和p2之间的key都大于选定的key，那么当p2走到末尾时交换p1与key值便完成了一次切分。

图示如下：
![](http://img.blog.csdn.net/20140326225106296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZG91ZmVpX2Njc3Q=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

代码如下：
```java
public ListNode sortList(ListNode head) {
    //采用快速排序
   quickSort(head, null);
   return head;
}
public static void quickSort(ListNode head, ListNode end) {
    if (head != end) {
        ListNode node = partion(head, end);
        quickSort(head, node);
        quickSort(node.next, end);
    }
}

public static ListNode partion(ListNode head, ListNode end) {
    ListNode p1 = head, p2 = head.next;

    //走到末尾才停
    while (p2 != end) {

        //大于key值时，p1向前走一步，交换p1与p2的值
        if (p2.val < head.val) {
            p1 = p1.next;

            int temp = p1.val;
            p1.val = p2.val;
            p2.val = temp;
        }
        p2 = p2.next;
    }

    //当有序时，不交换p1和key值
    if (p1 != head) {
        int temp = p1.val;
        p1.val = head.val;
        head.val = temp;
    }
    return p1;
}
```

# 归并排序



归并排序应该算是链表排序最佳的选择了，保证了最好和最坏时间复杂度都是nlogn，而且它在数组排序中广受诟病的空间复杂度在链表排序中也从O(n)降到了O(1)。

归并排序的一般步骤为：
1. 将待排序数组（链表）取中点并一分为二；
2. 递归地对左半部分进行归并排序；
3. 递归地对右半部分进行归并排序；
4. 将两个半部分进行合并（merge）,得到结果。


首先用快慢指针(快慢指针思路，快指针一次走两步，慢指针一次走一步，快指针在链表末尾时，慢指针恰好在链表中点)的方法找到链表中间节点，然后递归的对两个子链表排序，把两个排好序的子链表合并成一条有序的链表。



代码如下：

```java
public ListNode sortList(ListNode head) {
    //采用归并排序
    if (head == null || head.next == null) {
        return head;
    }
    //获取中间结点
    ListNode mid = getMid(head);
    ListNode right = mid.next;
    mid.next = null;
    //合并
    return mergeSort(sortList(head), sortList(right));
}

/**
 * 获取链表的中间结点,偶数时取中间第一个
 *
 * @param head
 * @return
 */
private ListNode getMid(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    //快慢指针
    ListNode slow = head, quick = head;
    //快2步，慢一步
    while (quick.next != null && quick.next.next != null) {
        slow = slow.next;
        quick = quick.next.next;
    }
    return slow;
}

/**
 *
 * 归并两个有序的链表
 *
 * @param head1
 * @param head2
 * @return
 */
private ListNode mergeSort(ListNode head1, ListNode head2) {
    ListNode p1 = head1, p2 = head2, head;
   //得到头节点的指向
    if (head1.val < head2.val) {
        head = head1;
        p1 = p1.next;
    } else {
        head = head2;
        p2 = p2.next;
    }

    ListNode p = head;
    //比较链表中的值
    while (p1 != null && p2 != null) {

        if (p1.val <= p2.val) {
            p.next = p1;
            p1 = p1.next;
            p = p.next;
        } else {
            p.next = p2;
            p2 = p2.next;
            p = p.next;
        }
    }
    //第二条链表空了
    if (p1 != null) {
        p.next = p1;
    }
    //第一条链表空了
    if (p2 != null) {
        p.next = p2;
    }
    return head;
}
```


完整代码放在：
https://github.com/morethink/algorithm/blob/master/src/main/java/algorithm/leetcode/L_148_SortList.java


**参考文档**：
1. [链表排序（冒泡、选择、插入、快排、归并、希尔、堆排序）](http://www.cnblogs.com/TenosDoIt/p/3666585.html)
