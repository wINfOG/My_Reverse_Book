# 我理想的反编译书籍

因为是理想，所以永远不可能达成。

既然没有其它人写，那我来写写看。

todo：施工中，看来还要施工好久；承诺群友争取2025年写完

# 在线阅读

https://tommy-3.gitbook.io/my_reverse_book/

# 项目目录（画饼）

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

反编译器的控制流还原技术（struct）

## 第六章

反编译器的类型系统还原技术（typing）

## 第七章

现代反编译技术+查漏补缺

## 第八章

第八章为案例分析章节，案例分析章节将分析真实世界中的反编译器是如何实现的，他们的架构啥样，有什么缺陷。

## 第九章

案例分析一：jadx反编译器架构、关键技术、案例讲解

## 第十章

案例分析二：IDA反编译器架构、关键技术、案例讲解

以及

案例分析三：Ghidra反编译器架构、关键技术、案例讲解

## 理论补充章节A

从199?年到20??年间，反编译技术发展中，学术界关键论文的阅读笔记。

# 实践章节

实践章节的目标是使用已有的成熟反编译码框架，对抗越来越艰难的外部环境。

## 动静态结合

介绍动静态结合的常用方法，能大大加速逆向效率

## 对抗轻度混淆

介绍对抗间接跳转，花指令，字符串加密等，比较简单混淆的快速批量还原方法。

## IDA+mircocode

在IDA中编写自己的mircocode中间语言优化插件；学习编写去除Ollvm-flat的Demo。

## Ghira+Sleigh

使用Ghidra和它的sleigh框架支持任意指令集的反编译+F5能力，可以拿一些vmp来练手。

## Binary-Ninja

//todo
