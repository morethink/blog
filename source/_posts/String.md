---
title: String解析
date: 2016-09-16
tags: String
categories: Java
---


>**可以证明，字符串操作是计算机程序设计中最常见的行为。**
>String对象是不可变的，查看JDK文档你就会发现，String类每一个看起来会修改String值得方法，实际上都是创建了一个全新的String对象，以包含修改后的字符串内容。而最初的string对象则丝毫未动。--Thinking in Java
<!-- more -->

 # 常量池
``` java
String str1 = "abc";
String str2 = "abc";
String str3 = new String("abc");
String str4 = new String("abc");
String str5 = "abc" + "li";
String str6 = "abc" + "li";
String str7 = str1 + "li";
String str8 = str1 + "li";
String str9 = str1 + str2;
String str10 = str1 + str2;

System.out.println(str1 == str2);
System.out.println(str3 == str4);
System.out.println(str5 == str6);
System.out.println(str7 == str8);
System.out.println(str9 == str10);
```

结果：
``` java
true
false
true
false
false
```

说明：
> 在JAVA虚拟机（JVM）中存在着一个字符串池，其中保存着很多String对象，并且可以被共享使用，因此它提高了效率。由于String类是final的，它的值一经创建就不可改变，因此我们不用担心String对象共享而带来程序的混乱。字符串池由String类维护，我们可以调用intern()方法来访问字符串池。

1. 我们再回头看看String str1 = "abc";  , 这行代码被执行的时候，JAVA虚拟机首先在字符串池中查找是否已经存在了值为"abc"的这么一个对象，它的判断依据是String类equals(Object obj)方法的返回值。如果有，则不再创建新的对象，直接返回已存在对象的引用；如果没有，则先创建这个对象，然后把它加入到字符串池中，再将它的引用返回。

2. new 就是在堆中重新分配了一块内存，虽然  str3  和  str4 内容相同，但它们指向的是不同的内存。

3. 使用 “ + ”时，
"字符1 " + "字符2" 在上面第一条已说明，
 “字符1”+  引用 和
 引用      + 引用  都将创建新的对象。
# String 相等 ? #

## 引用相等： ##
“==”，通过“==”运算符来比较内存地址是否相等

`C++String`类重载了==运算符以检测字符串内容的相等性。

## 内容相等：equals 方法 ##

比较两个String对象的内容是否相同时是使用equals方法，equals方法是String类继承与Object类的，
在**Object类的equals方法的本质其实是和“==”一样的，都是比较两个对象引用是否指向同一个对象**（即两个对象是否为同一对象）。
那为什么String类的equals方法却又是比较两个String对象的内容是否相同呢？

原来是这样的，String类继承Object类后，也继承了equals方法，但String类对equals方法进行了重写，改变了equals方法的比较形式。其实很多其他类继承Object类后也对equals方法进行了重写。

jdk1.8.0_92  中String类中equals 方法
``` java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

# 编码： #
     代码点和代码单元
字节是计算机存储信息的基本单位，**1 个字节等于 8 位**， **gbk 编码中 1 个汉字字符存储需要 2 个字节**，**1 个英文字符存储需要 1 个字节**。所以我们看到上面的程序运行结果中，每个汉字对应两个字节值，如“学”对应 “-47 -89” ，而英文字母 “J” 对应 “74” 。同时，我们还发现汉字对应的字节值为负数，原因在于每个字节是 8 位，最大值不能超过 127，而**汉字转换为字节后超过 127，如果超过就会溢出，以负数的形式显示。**

# 正则： #
## String类自带正则表达式：

public boolean matches(String regex)告知此字符串是否匹配给定的正则表达式。
调用此方法的 str.matches(regex) 形式与以下表达式产生的结果完全相同：
Pattern.matches(regex, str)


java 的 " \\ "等于其他语言的" \





# 知识点：

空串是一个对象

# 常用API
          public char **charAt**(int index)  
返回指定索引处的char值
