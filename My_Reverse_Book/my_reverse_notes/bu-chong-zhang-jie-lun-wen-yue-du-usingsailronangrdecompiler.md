---
description: >-
  补充章节+论文阅读+结构化分析+Ahoy SAILR! There is No Need to DREAM of C: A Compiler-Aware
  Structuring Algorithm for Binary Decompilation
---

# 补充章节-论文阅读：using-sailr-on-angr-decompiler

## 简介

论文标题：

《Ahoy SAILR! There is No Need to DREAM of C: A Compiler-Aware Structuring Algorithm for Binary Decompilation》



代码Github地址：

{% embed url="https://github.com/mahaloz/sailr-eval#using-sailr-on-angr-decompiler" %}

## 论文作者

**Zion Leonahenahe Basque（mahaloz）**

作者是shellphish成员，angr的开发者之一，这也可以解释为什么论文的代码实现使用了angr。

作者的个人博客：[https://mahaloz.re/about.html](https://mahaloz.re/about.html)

个人主页：[https://www.zionbasque.com/](https://www.zionbasque.com/)

github：[https://github.com/mahaloz](https://github.com/mahaloz)



## 反编译的控制流还原

在我[之前的文章](https://bbs.kanxue.com/thread-278789.htm)，有一些简略的介绍。这里快速的总结一下。



### 什么是控制流还原？

在反编译器的执行过程中，控制流还原的目标是将CFG还原成由if、while、for等组成的高级抽象结构。如下图有个控制流图，他的所有边在汇编中只是个jmp，反编译器需要理解控制流的结构，把条件分支和循环识别出来。

最终，生成更高级的，更上层的控制流结构更适合人类阅读与理解，毕竟正常人应该很难理解全是goto的CFG结构，相比之下分支和循环就好理解多了。

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### 主流反编译器的控制流还原算法如何实现？

主流的反编译，也就是IDA和Ghidra，所使用的控制流还原算法来自于编译器的【结构分析】技术。

可以参考鲸书：《高级编译器的设计与实现》第7章。其原理大致是预置好各种流图模式，然后在CFG中寻找能对应上的模式后，把流图按照规则进行"塌缩"，直到塌缩成一个点就完成分析了。

参考下面这张图，塌缩的规则：

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

以及一个还原的案例：

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

### 什么是“可规约的图”？

在学术上结构化分析的前提条件是控制流图必须是可规约的。

“可规约”可以简单的理解为，这个图能够按照一定的“塌缩规则”，最终压缩成一个点。否则这个CFG图无法正常的进行还原。

### 主流反编译器的控制流还原算法有什么缺陷？

如下图，节点2是个典型if结分支构，但是他的后继节点3却不被节点2支配；

所谓不被支配，意思是有一个【节点1 到节点3】 的路径绕不经过节点2，因此节点3与节点4不被节点2支配。

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

这违反了SESE(single entry + single exit)的well-define结构化分析规则。

所以，结构化分析还原的理论上来说，“钻石形”的cfg是不可规约的的，无法被正常还原。

### 为什么会出现这种缺陷？

正常写代码，只要不使用goto等非常规语法，写出来的代码控制流结构都是可以被还原的。

但是，编译器会对cfg图进行优化，主要目的是精简重复代码，减少条件判断语句增加效率等等。这一系列的优化可能会生成不可规约的控制流结构，造成【结构分析】技术无法正常还原，只能将边删除后插入goto来缓解。

虽然我[之前的文章](https://bbs.kanxue.com/thread-278789.htm)已经给了一个案例；不过《Ahoy SAILR! There is No Need to DREAM of C: A Compiler-Aware Structuring Algorithm for Binary Decompilation》这篇论文做得更加完美。

他将GCC编译器中能对控制流造成影响，会产生不可规约图结构的编译选项全部列了出来，还给出了影响程度的分析。

### 补充：为什么从理论上来说，“钻石形”的cfg无法被还原

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

如图，节点2明显是个if语句的入口，但是他的后继节点3却不被节点2支配；即，可以从节点1 ->节点3  这个路径绕过去;

因此节点3与节点4不被节点2支配，所以违反了SESE(single entry + single exit)的well-define结构化分析规则，所以“钻石形”的cfg无法被还原！



## 论文的主要目的

我们从论文的标题看起，这就很有趣。

《Ahoy SAILR! There is No Need to DREAM of C: A Compiler-Aware Structuring Algorithm for Binary Decompilation》

标题中的“DREAM”指的是NDSS 2015的论文 《No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations》，后续用Dream来指代。

2015年的Dream论文提出了一种与图匹配完全不同的方式：排序路径和抵达路径的条件，然后顺序的组装出条件代码配合图优化；这种方法从理论上证明了任何一个cfg图都能还原成不包含goto语句的控制流结构。

那么，SAILR这篇论文看标题就知道是反驳DREAM的方法，论文的主要观点是，没有goto或者说goto-less的结构并不意味着生成源码可读性高，还是应该在原有的结构化算法回归并改进。



## 补充：Dream论文的goto-less方案

Dream论文-ndss2015：[https://net.cs.uni-bonn.de/fileadmin/ag/martini/Staff/yakdan/dream\_ndss2015.pdf](https://net.cs.uni-bonn.de/fileadmin/ag/martini/Staff/yakdan/dream\_ndss2015.pdf)

这篇论文的贡献是，相比于传统的“结构化分析”技术，论文提出了一种完全不同的方法。

这种方法从理论上证明，不用考虑图规约问题，任意的cfg图都能还原成不包含goto语句的高级控制流结构。

具体还原方法请参考目录里关于dream/no-more-goto的补充章节（可能没写完）



## 论文想讨论的内容

首先，论文提出了no-more-goto这篇论文中将所有的goto全部消除这种做法不可取，提出了以下的观点：

* 虽然goto-less算法具有先进性，但是会大规模的修改cfg，导致降低可读性。
* 在现实中，有的代码里就是有goto的（把这部分的goto优化反而导致可读性降低），需要有区分的判断。
* 反编译结果的可读性不应当以生成的goto多少进行判断，应当与编译前的源码结构进行比较
* 编译优化是产生的不可规约控制流的主要原因。
* 当前反编译的结构化的技术要不是没有开源，要不是太久没有更新了，需要针对于特定编译器的更先进的反编译技术。



todo