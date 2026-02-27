# AI时代已经来了，把时间放到更加高效的结合AI工作去吧！

# 我理想的反编译书籍（施工中）

因为是理想，所以永远不可能达成。

既然没有其它人写，那我来写写看。

todo：施工中，框架完成，补充案例中；承诺群友争取2025年写完

# 在线阅读

本体在线阅读

https://tommy-3.gitbook.io/my_reverse_book/

后续：混淆与对抗章节

https://tommy-3.gitbook.io/my_reverse_book/my_obfuscation_book

# 介绍

随着现代编程语言和开发框架的演进，反编译技术也越来越复杂。高级语言的抽象特性、编译器优化的多样性，以及代码混淆技术的广泛应用，使得从二进制机器码还原出可读性高、语义清晰的源代码变得愈发困难；

这个项目将结合理论与实践，从反编译工程师的角度出发，，探讨如何应对编译器优化、代码混淆和多语言环境带来的难题。

# 目录

## 第一章

人类有什么缺陷？

为什么要学习反编译的知识？

快速过一遍逆向工程中的反编译技术。

## 第二章

反编译技术发展历史、通用反编译器架构；为什么反编译器最终变成了现在这个样子？

以及，关键技术概览和反编译整体流程简述。

## 第三章

介绍SSA与反编译器，详细说明反编译整体流程，为什么SSA这么重要。

从二进制（机器码）→ IR → BasicBlock→ CFG的主要流程与关键技术和难题。

## 第四章

内容最多的一章，数据流分析与编译中端优化技术合集。

## 第五章

反编译器的控制流还原技术(struct), 将汇编中goto/jmp的结构转换为if-else-while-switch的高层次语意。

## 第六章

反编译器的类型系统还原技术(typing), 还原类型数据+结构体等高级语意。

## 第七章

变量还原，反编译技术如何还原出适合人类阅读的变量。

以及，现代反编译技术与各种查漏补缺

## 第八章

第八章为案例分析章节，案例分析章节将分析真实世界中的反编译器是如何实现的，他们的架构啥样，有什么缺陷。

案例分析一：jadx反编译器架构、关键技术、案例讲解

案例分析二：IDA反编译器架构、关键技术、案例讲解

案例分析三：Ghidra反编译器架构、关键技术、案例讲解

案例分析四：Binary-ninja反编译器架构、关键技术、案例讲解

## 理论补充章节

从199?年到20??年间，反编译技术发展中，学术界关键论文的阅读笔记。

当前已更新：

USENIX 2024: ["Ahoy SAILR! There is No Need to DREAM of C:
A Compiler-Aware Structuring Algorithm for Binary Decompilation"](https://www.zionbasque.com/files/publications/sailr_usenix24.pdf)

计划更新:

NDSS 2105:["No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations"](https://www.ndss-symposium.org/ndss2015/ndss-2015-programme/no-more-gotos-decompilation-using-pattern-independent-control-flow-structuring-and-semantics/)

.. todo：DCC相关 Ghidra相关

# 实践章节

实践章节，主要是对抗现代化高强度混淆的技术；例如OLLVM/VMP等商业化保护技术，

这个部分过于庞大了，因此移动到了另外一本书中：

https://tommy-3.gitbook.io/my_reverse_book/my_obfuscation_book



