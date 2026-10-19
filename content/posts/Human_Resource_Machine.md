---
title: 'Human Resource Machine'
date: 2026-10-19T14:13:15+08:00
draft: true
author: "Jsxts"
description: ""
summary: ""
math: true
tags: []
categories: []
showToc: true
TocOpen: false
hidemeta: false
comments: false 
---

前段时间无聊，通过不断地优（卡）化（常），终于满灯通关了这个游戏。今天看到程设课的期末大作业竟然是复现这个游戏，于是在开始搓之前先回顾了一下，并顺便写一下对这个游戏的介绍以及部分关卡的攻略。

## Introduction

Human Resource Machine，中文名为程序员升职记，是一个用模块化结构编程完成对应任务来闯关的编程游戏。



![](/images/Human%20Resource%20Machine/image1.png)



*第二关界面*



其中可用的模块有输入、输出、加减、jump（类似于 goto）等，相较于 C++ 等编程语言较为有限，因此任务难度也较低，想要真正实现要求不难，其中真正困难的部分是点亮每关旁边的两个灯，分别是对代码长度和运算次数的限制，这需要我们对代码进行不断的优化。这两个限制在大多数情况下无法同时满足，因此我们需要两份代码分别点亮两个灯。



个人体验下来，对代码长度的限制一般较好解决， 通常算法正确时代码长度与关卡要求已经十分接近，剩下的就是删去一些可合并的某两句代码就能达到要求。较难的关卡代码长度的限制也会较为宽松。



![](/images/Human%20Resource%20Machine/image2.png)

*关卡：三排序，未经优化就能达到代码长度要求（$\leq 34$ 行）*



但是，对运算次数的限制通常恶心得多。很多时候明明代码非常短，运算次数却大大超过限制。对此做出优化的方法一方面是使用另一种更加优秀的算法，通常能大幅减少运算次数，以及采用大量分支结构来减少不必要的判断，这通常会使代码变得极为冗长。有时候这些还不够，必须通过循环展开来减少常数次的运算从而最终达到要求。事实上最后一种方法不需要动脑，是在只差一点的时候最好的解决方法，相较于改变算法常用得多。



![](/images/Human%20Resource%20Machine/image3.png)

*为了减少运算次数不得不把代码延长亿点点*



以上大致就是对这个游戏的玩法介绍，接下来是一些我认为较难的关卡的攻略：



# Solutions

以下代码可直接粘贴至游戏中。

## 第20年 乘法研讨会

这题算法和代码长度限制没有难点，就是把其中一个数减到 $0$ 的同时把另一个数加同样多次。但是 $15$ 行的代码写完之后发现运算次数达到了惊人的 $139$ 次，而循环展开的收效甚微，因此我们考虑始终让较小的数进行减到 $0$ 的操作，即在一开始比较一下大小，同时结合对其中一个数是 $0$ 的特判，即使不用循环展开也可以轻松满足要求。

```
-- HUMAN RESOURCE MACHINE PROGRAM --

    JUMP     b
a:
    OUTBOX  
    INBOX   
b:
c:
d:
    INBOX   
    JUMPZ    a
    COPYTO   6
    COPYTO   2
    INBOX   
    JUMPZ    j
    SUB      2
    JUMPN    g
    ADD      2
    COPYTO   2
    COPYTO   0
e:
    BUMPDN   6
    JUMPZ    f
    COPYFROM 0
    ADD      2
    COPYTO   0
    JUMP     e
f:
    COPYFROM 0
    OUTBOX  
    JUMP     d
g:
    ADD      2
    COPYTO   0
h:
    BUMPDN   0
    JUMPZ    i
    COPYFROM 6
    ADD      2
    COPYTO   6
    JUMP     h
i:
    COPYFROM 6
j:
    OUTBOX  
    JUMP     c
```



## 第28年 三排序

为数不多的代码长度可能更难满足的题。

想运算次数尽量少，直接用 jumpif 写一堆大分支，每种大小顺序都分别处理，写了 $62$ 行运算次数正好满足。

但想要代码短，就要通过三次比较和交换操作来把它们排好序再输出：

```c++ 伪代码
if (a > b) swap(a,b);
if (b > c) swap(b,c);
if (a > b) swap(a,b);
```

不难验证正确性。交换可以用临时变量，第三次交换可以利用第一次的代码，这样可以把代码压缩到 $33$ 行：

```
-- HUMAN RESOURCE MACHINE PROGRAM --

a:
    INBOX   
    COPYTO   0
    INBOX   
    COPYTO   1
    INBOX   
    COPYTO   2
    SUB      1
    JUMPN    c
b:
    COPYFROM 2
    COPYTO   3
    COPYFROM 1
    COPYTO   2
    COPYFROM 3
    COPYTO   1
c:
    COPYFROM 1
    SUB      0
    JUMPN    d
    COPYFROM 0
    COPYTO   3
    COPYFROM 1
    COPYTO   0
    COPYFROM 3
    COPYTO   1
d:
    COPYFROM 1
    SUB      2
    JUMPN    b
    COPYFROM 2
    OUTBOX  
    COPYFROM 1
    OUTBOX  
    COPYFROM 0
    OUTBOX  
    JUMP     a
```

## 第38年 数位炸弹

算法也是十分的简单啊，非常轻松就能写出 $29$ 行的代码，然而它运行了 $211$ 次 ······

尝试循环展开，然而依旧有 $198$ 次，而且由于算法太简单，优化空间几乎没有了。

于是把目光移向对每一位的减法，发现每次减一都要做很多判断和各种 copy，于是再次祭出大分支，对这一位一直减，减到 $0$ 就跳出然后加对应的次数，这样我们就需要对这一位是 $0\sim 9$ 分别处理，而我们只有加一操作，也就是要分别加 $0\sim 9$ 次 ······

当然这样处理完百位之后，发现已经很接近了，于是对十位只需每减 $3$ 次进行一些判断，最后可以减少到 $161$ 次运算。

```
-- HUMAN RESOURCE MACHINE PROGRAM --

a:
b:
    COPYFROM 9
    COPYTO   3
    COPYTO   4
    INBOX   
    SUB      11
    JUMPN    k
    SUB      11
    JUMPN    j
    SUB      11
    JUMPN    i
    SUB      11
    JUMPN    h
    SUB      11
    JUMPN    g
    SUB      11
    JUMPN    f
    SUB      11
    JUMPN    e
    SUB      11
    JUMPN    d
    SUB      11
    JUMPN    c
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     l
c:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     s
d:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     t
e:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     r
f:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     q
g:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     p
h:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    BUMPUP   3
    JUMP     o
i:
    ADD      11
    COPYTO   0
    BUMPUP   3
    BUMPUP   3
    JUMP     m
j:
    ADD      11
    COPYTO   0
    BUMPUP   3
    JUMP     n
k:
    ADD      11
    COPYTO   0
l:
m:
n:
o:
p:
q:
r:
s:
t:
u:
    COPYFROM 0
    SUB      10
    JUMPN    x
    SUB      10
    JUMPN    w
    SUB      10
    JUMPN    v
    COPYTO   0
    BUMPUP   4
    BUMPUP   4
    BUMPUP   4
    JUMP     u
v:
    ADD      10
    COPYTO   0
    BUMPUP   4
    BUMPUP   4
    JUMP     y
w:
    ADD      10
    COPYTO   0
    BUMPUP   4
x:
y:
    COPYFROM 3
    JUMPZ    z
    OUTBOX  
    COPYFROM 4
    OUTBOX  
    COPYFROM 0
    OUTBOX  
    JUMP     b
z:
    COPYFROM 4
    JUMPZ    aa
    OUTBOX  
aa:
    COPYFROM 0
    OUTBOX  
    JUMP     a
```

（代码不忍直视）

## 第40年 质数工厂

注意这题相同的质因子要输出多次。一种代码比较短的写法是：让这个数一直除以 $2$，然后一直除以 $3$，以此类推直到变成 $1$。这种方法不需要枚举质数，因为如果除不了 $2$ 一定除不了 $4$。这样刚好 $28$ 行：

```
-- HUMAN RESOURCE MACHINE PROGRAM --

    BUMPUP   24
a:
    COPYFROM 24
    COPYTO   23
    BUMPUP   23
    INBOX   
    COPYTO   0
b:
    COPYFROM 0
    BUMPDN   0
    JUMPZ    a
    BUMPUP   0
    COPYTO   2
c:
    COPYFROM 24
    COPYTO   1
d:
    COPYFROM 2
    SUB      23
    JUMPN    f
    JUMPZ    e
    COPYTO   2
    BUMPUP   1
    JUMP     d
e:
    COPYFROM 23
    OUTBOX  
    COPYFROM 1
    COPYTO   0
    COPYTO   2
    JUMP     c
f:
    BUMPUP   23
    JUMP     b
```

但这样会有 $538$ 次运算。发现对合数的试除导致了无用的运算，于是考虑换一种办法，就是先把比较小的质数列出来（观察数据发现没有超过 $19$ 的质因子），再只对每个质数试除，注意同一个质数要除多次。加上循环展开优化一些常数，这样可以减少到 $388$ 次运算。

```
-- HUMAN RESOURCE MACHINE PROGRAM --

    COPYFROM 24
    COPYTO   0
    BUMPUP   0
    COPYTO   18
    BUMPUP   0
    COPYTO   1
    BUMPUP   1
    ADD      0
    COPYTO   2
    ADD      0
    COPYTO   3
    ADD      0
    ADD      0
    COPYTO   4
    ADD      0
    COPYTO   5
    ADD      0
    ADD      0
    COPYTO   6
    ADD      0
    COPYTO   7
a:
    COPYFROM 24
    COPYTO   23
    INBOX   
    COPYTO   22
b:
    BUMPDN   22
    JUMPZ    a
    BUMPUP   22
    COPYTO   19
c:
    COPYFROM 18
    COPYTO   20
d:
    COPYFROM 19
    SUB      [23]
    JUMPN    g
    JUMPZ    e
    COPYTO   19
    BUMPUP   20
    COPYFROM 19
    SUB      [23]
    JUMPN    h
    JUMPZ    f
    COPYTO   19
    BUMPUP   20
    JUMP     d
e:
f:
    COPYFROM [23]
    OUTBOX  
    COPYFROM 20
    COPYTO   19
    COPYTO   22
    JUMP     c
g:
h:
    BUMPUP   23
    JUMP     b
```

## 第41年 排序楼层

比较容易实现的一种排序方法是选择排序，即每次找到最小的数并输出，然后再去掉这个数。直接实现可以做到 $32$ 行，但是运算次数高达 $1119$ 次。于是采用一些暴力方式：

- 进行一些循环展开

- 特判 $n=1$ 的情况

- 优化一下枚举，省去不必要的运算（比如不要自己和自己比较）

- 最后两个数单独比，不用在循环里面比

再加上一些对代码的结构的优化，可以把运算次数降到 $700$ 次以下。

```
-- HUMAN RESOURCE MACHINE PROGRAM --

a:
b:
c:
    COPYFROM 24
    COPYTO   20
    COPYTO   21
d:
    INBOX   
    COPYTO   [20]
    JUMPZ    f
    BUMPUP   20
    INBOX   
    COPYTO   [20]
    JUMPZ    g
    BUMPUP   20
    INBOX   
    COPYTO   [20]
    JUMPZ    e
    BUMPUP   20
    JUMP     d
e:
f:
g:
    COPYFROM 20
    COPYTO   17
    BUMPDN   17
    JUMPZ    k
    COPYTO   18
    BUMPDN   18
h:
    COPYFROM 21
    COPYTO   22
    COPYTO   19
    BUMPUP   22
i:
    COPYFROM [19]
    SUB      [22]
    JUMPN    j
    COPYFROM 22
    COPYTO   19
j:
    BUMPUP   22
    SUB      20
    JUMPN    i
    COPYFROM [21]
    COPYTO   23
    COPYFROM [19]
    COPYTO   [21]
    OUTBOX  
    COPYFROM 23
    COPYTO   [19]
    BUMPUP   21
    SUB      18
    JUMPN    h
    COPYFROM [21]
    SUB      [17]
    JUMPN    l
    COPYFROM [17]
    OUTBOX  
    COPYFROM [18]
    OUTBOX  
    JUMP     a
k:
    COPYFROM [17]
    OUTBOX  
    JUMP     b
l:
    COPYFROM [18]
    OUTBOX  
    COPYFROM [17]
    OUTBOX  
    JUMP     c


DEFINE LABEL 17
eJyTZmBgUPNgiVrl1pS/ydmsLtrhx9Roh8dzzZzaOhd5GqVs982M5Qt0j2ELCUn9GqnYJhTN2ycU3bUY
qI3hampVw4Z0s7r/uaK1MvlmdWGFDK1bCwsnhhXOn21ZoLbgUc6aJVsLMxf9qYqZeb0mZibDKBgFo2DQ
AQD2EyvV;

DEFINE LABEL 18
eJwTYmBgmNSlVSrWuaGSs0nvSkbT5wN32tQWXOksnMjWP7vx5uQNlVkzc8tdZ6d2LJ4dM9N1dtu59lk/
bgG1MfzZ2rV47lbZjW6bAq4yjIJRMAqGJAAASEwitg;

DEFINE LABEL 20
eJzjZmBgWJFVmpaXsWWyTubJ7ur8JQVbC/9kZZfuzFha/ifrfuWSgi+1VQ3BdWoLGEbBKBgFww4AAEY3
FXo;

DEFINE LABEL 21
eJxjZ2Bg8Mw62c2ezdv3P7dpi0z+8+1AIYZXme8TGUbBKBgFwx4AALF2Coo;

DEFINE LABEL 22
eJwTZGBgsG6332Td/n5ZbZvagp/t16b8bI+ZKdcmuEKube/xhNYV54PrAq4+L9t641b+1huOua8vArUw
8PU8ipBrexSxqGV+GMMoGAWjYMgCAFymIRI;

DEFINE LABEL 23
eJzjZmBgqKi5lbmrenZjdum9E9mlhdeel/24FV1ddx8oxXA2Q3J6a7PkdI++mJkMo2AUjIJhBwDOAxQL
;


```



# Conclusion

事实上，由于这个游戏的代码块功能较为僵硬且单一，很多时候仅仅是一些细节都需要大量的努力来调整，以及上面的优化过程同时要确保其他地方不受到影响并非易事。因此，该游戏的游玩体验不能说很好。但是，经过长时间优化最后取得成功无疑是极有成就感的。以及，就算你无心冲击完美，简单地通关这个游戏也能对程序设计思维带来提升，并从中感受到乐趣。



所以，快来试试吧！



[下载链接（百度网盘）](https://pan.baidu.com/s/1FlQ_TMCCgnPhvhEs9KNSDA?pwd=shkb)
