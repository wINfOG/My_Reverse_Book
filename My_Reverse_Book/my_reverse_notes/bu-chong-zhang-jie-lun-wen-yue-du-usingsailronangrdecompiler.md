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

## "我"的评价

我认为，这篇论文的价值不在于它的优化算法，以及和DREAK论文的比较部分。

它的价值在于：

第一：作者从真正逆向工程的角度出发，提出了适用于判断反编译生成CFG结构优劣的标准+算法，即和源码直接比较。之前的算法都是从圈复杂度，goto数量等等源码的判断方法出发，而忽略了可读性这个最重要的指标。

第二：作者详细的分析了编译器的哪些编译优化选项会生成不可规约的CFG图，这是一个比较繁琐的工作。熟悉这些编译选项以及优化方法，对于逆向工程有很大的帮助。



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

因此节点3和节点4不被节点2支配，所以违反了SESE(single entry + single exit)的well-define结构化分析规则，所以“钻石形”的cfg无法被还原！



## 论文的主要目的

我们从论文的标题看起，这就很有趣。

《Ahoy SAILR! There is No Need to DREAM of C: A Compiler-Aware Structuring Algorithm for Binary Decompilation》

标题中的“DREAM”指的是NDSS 2015的论文 《No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations》，后续用Dream来指代。

2015年的Dream论文提出了一种与图匹配完全不同的方式：排序路径和抵达路径的条件，然后顺序的组装出条件代码配合图优化；这种方法从理论上证明了任何一个cfg图都能还原成不包含goto语句的控制流结构。

那么，SAILR这篇论文看标题就知道是反驳DREAM的方法，论文的主要观点是，没有goto或者说goto-less的结构并不意味着生成源码可读性高，还是应该在原有的结构化算法回归并改进。



## 补充：Dream论文的goto-less方案

Dream论文-ndss2015：[https://net.cs.uni-bonn.de/fileadmin/ag/martini/Staff/yakdan/dream\_ndss2015.pdf](https://net.cs.uni-bonn.de/fileadmin/ag/martini/Staff/yakdan/dream_ndss2015.pdf)

这篇论文的贡献是，相比于传统的“结构化分析”技术，论文提出了一种完全不同的方法。

这种方法从理论上证明，不用考虑图规约问题，任意的cfg图都能还原成不包含goto语句的高级控制流结构。

具体还原方法请参考目录里关于dream/no-more-goto的补充章节（可能没写完）



## 论文主体

首先，论文提出了no-more-goto这篇论文中将所有的goto全部消除这种做法不可取，提出了以下的观点：

* 虽然goto-less算法具有先进性，但是会大规模的修改cfg，导致降低可读性。
* 在现实中，有的代码里就是有goto的（把这部分的goto优化反而导致可读性降低），需要有区分的判断。
* 反编译结果的可读性不应当以生成的goto多少进行判断，应当与编译前的源码结构进行比较
* 编译优化是产生的不可规约控制流的主要原因。
* 当前反编译的结构化的技术要不是没有开源，要不是太久没有更新了，需要针对于特定编译器的更先进的反编译技术。



## 编译器优化对于控制流的影响

既然导致不可规约控制流产生的主要原因是编译选项，反编译器设计的时候就需要明确，那些编译选项会有这些影响？

论文的测试方法：使用GCC编译器开启O2优化后，分别关掉各个优化选项；使用IDA反编译整个二进制，统计生成goto语句的数量，论文结果如下。

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

可以看出，生成goto语句影响较大的编译选项有以下两个：

* A：Jump Threading
* D：Cross Jumping



而且，哪怕使用O0进行编译，还是会有goto语句生成。

下面稍微解释一下这些编译选项对于控制流的影响。

### A：Jump Threading

参考wiki：[https://en.wikipedia.org/wiki/Jump\_threading](https://en.wikipedia.org/wiki/Jump_threading) 中的例子，Jump Threading编译优化方案用于简化条件逻辑，可以有效的减少分支判断次数。由于现代CPU流水线和分支预测的特性，减少分支判断可以显著提升性能。

插入的基本块会影响控制流结构

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### B：Common Subexpression Elimination（CSE）

{% embed url="https://en.wikipedia.org/wiki/Common_subexpression_elimination" %}

参考本书主要章节对于CSE优化的说明，编译器可能会将多条CSE语句压缩到一个块里，然后进行跳转。

### C：Switch Conversion

和上面的CSE类似，只不过条件更为苛刻，对于swich中的赋值语句，编译器可能会通过goto来压缩语句，优化前的示例代码：

```
switch(a) {
    case 1:
        //do-something
        ret = 100;
        error = "error";
        break;
    case 2:
        //do-something
        ret = 100;
        error = "error";
        break;
    .....
}
```

优化后，将两个case尾部相同的代码使用goto连接，压缩代码大小。

```
switch(a) {
    case 1:
        //do-something
        ret = 100;
        error = "error";
        goto label;
    case 2:
        //do-something
        label:
        ret = 100;
        error = "error";
        break;
    .....
}
```



### D：Cross Jumping

参考我[之前的文章](https://bbs.kanxue.com/thread-278789.htm) “为什么IDA F5后的代码会有GOTO语句？” 中的案例

优化前的代码：

```
int fun(int a, int b)
{
    int ret = 0;
    if (getchar() > 0x10) {
        ret = a+b;
    } else {
        ret = a-b;
        if (getchar()>0x20) {
            printf("hello");
            printf("ret=xx");
            return ret;
        }
    }
    printf("hi");
    printf("ret=xx");
    return ret;
}
```

编译器优化中发现， printf("ret=xx");  return ret; 这两句代码完全一致，为了减少代码量就插入一个goto语句将这两个尾部进行融合。优化后插入了goto。

```
int fun(int a, int b)
{
    int ret = 0;
    if (getchar() > 0x10) {
        ret = a+b;
    } else {
        ret = a-b;
        if (getchar()>0x20) {
            printf("hello");
            goto LABEL
        }
    }
    printf("hi");
LABEL:
    printf("ret=xx");
    return ret;
}

```

如下图

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### E：Software Thread Cache Reordering

没太读明白，看着意思是在多线程情况下，对于热点代码，通过代码复制增加效率



### F：Loop Header Optimizations

循环头部提取优化（循环不变优化）

对于循环体内开头不变且不受循环影响的部分，提取到循环体外避免每次都执行，但是如果这部分代码中有跳转，就会导致出现goto语句。

简单代码案例：

```
for (int i = 0; i < 0x100; i++) {
    if (g_datas > 0) {
        break;
    }
    ..... // do-something
}
```

编译器优化后会将g\_datas 的判断移出来

```
if (g_datas > 0) {
    goto label;
}
for (int i = 0; i < 0x100; i++) {
 ..... // do-something
}
label:
```

### G：Builtin Inlining

为了执行效率，编译器会把strcmp memset这种函数用内置的汇编inline掉。因而这部分代码中的判断条件会产生goto语句

### H：Switch Lowering

对于大型switch语句的二分与hash化查找优化，可能会产生goto语句

### I：no-return function

参考主体章节中“间接跳转与间接调用的处理”中关于no-return的内容

exit等这一类函数执行后永远不会返回，编译器也就不再生成后续的代码。

如果反编译器没有正确的识别并传播这类信息，就会导致多加一个边，进而产生反编译问题。



## 如何判断反编译的结构化分析效果

之前的反编译论文中，往往以goto 数量+圈复杂度+代码行数对反结构化分析效果在进行评测。

但是，论文中指出，无论是goto 数量/圈复杂度/代码行，这都是对源码的分析方法，对于二进制的反编译结果并不是那么可靠。

最可靠的方法应当是：默认源代码的可读性是最优的，直接拿编译前的源代码和反编译F5后的结果进行对比。

论文提出了使用GED（Control-Flow Graph Edit Distance）图距离算法（这个指标以前用于二进制的图相似度匹配）但是GED是一个NP-hard问题且开销很高。因此做了一个简化版本的GED算法，我不是算法专家，不做展开。

&#x20;

## 对于不可规约控制流的还原的研究

既然上面的章节提出并整理了编译选项对于控制流的影响，作者提出了三种修改的控制流的场景。分别为

* ISD：Irreducible Statement Duplication（A/E/F）编译器将一个语句扩展成多个语句的场景。
* ISC：Irreducible Statement Condensing  (B/C/D）编译器将多个语句压缩成单个语句的场景。
* MISC：不属于上面两类的都算在这里。

这些优化所有的目的都是为了生成更适合人类阅读的控制流图。

### 对抗ISD优化

ISD：Irreducible Statement Duplication，指的是编译器将一个语句扩展成多个语句的优化场景。

对于反编译器，需要把重复的语句合并掉，来生成更好的控制流图。

反编译器优化案例如下：

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

算法步骤概要

* KMP算法，寻找一个CFG中完全重复的两段代码对。
* 寻找满足要求的代码对，1. 代码对后续存在一个goto语句导致控制流不完整； 2. 它们的后继节点是同一个（中间可以插入其它节点）
* 将两个重复代码合并成一个节点，判断节点的进出条件，重新组装CFG



### 对抗ISC优化

ISC：Irreducible Statement Condensing ，指的是编译器将多个语句压缩成单个语句的优化场景。

反编译器的优化核心是找到合适的需要重复的代码，通过复制CFG的节点来减少最后的不可规约场景。如果一个goto语句的后继节点通过复制可以减少goto 的数量，那就执行优化。

当然，这个优化算法有着明显的性能问题，为了知道拿个边是goto语句，需要先完成至少一轮的结构分析。

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 其它（MISC）类型优化

不是主要内容，跳过。

## 效果展示

可以看到，使用论文中的GED图距离算法对比源码和反编译后的CFG图，SAILR有着明显的优势。

而DREAM的goto-less算法虽然goto的数量没有，但是GED却很高。

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

