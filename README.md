# 我理想的反编译书籍

因为是理想，所以永远不可能达成。

既然没有其它人写，那我来写写看。

todo：施工中，框架完成，补充案例中；承诺群友争取2025年写完

# 在线阅读

本体

https://tommy-3.gitbook.io/my_reverse_book/

实践章节

https://tommy-3.gitbook.io/my_reverse_book/my_obfuscation_book

# 项目目录

## 第一章

人类有什么缺陷？

为什么要学习反编译的知识？

快速过一遍逆向工程中的反编译技术。

## 第二章

反编译技术发展历史、通用反编译器架构；

关键技术概览和反编译整体流程简述。

## 第三章

介绍SSA与反编译器，详细说明反编译整体流程，为什么SSA这么重要。

从二进制（机器码）→ IR → BasicBlock→ CFG的主要流程与关键技术和难题。

## 第四章

内容最多的一章，数据流分析与编译中端优化技术合集。

## 第五章

反编译器的控制流还原技术(struct), 将汇编中goto/jmp的结构转换为if-else-while-switch的高层次语意。

## 第六章

反编译器的类型系统还原技术(typing),

## 第七章

现代反编译技术+查漏补缺

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

.. todo

# 实践章节

实践章节，以及对抗现代化高强度混淆的技术，过于庞大了，因此移动到了另外一本书中：

https://tommy-3.gitbook.io/my_reverse_book/my_obfuscation_book



