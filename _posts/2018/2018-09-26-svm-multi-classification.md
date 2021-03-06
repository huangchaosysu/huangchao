---
layout: single
author_profile: true
title: "SVM实现多个分类的方案"
date: 2018-09-26 14:30:53
# toc: true
tags:
  - 算法
  - 机器学习
categories:
  - 算法
---

SVM算法最初是为二值分类问题设计的，当处理多类问题时，就需要构造合适的多类分类器。

目前，构造SVM多类分类器的方法主要有两类

（1）直接法，直接在目标函数上进行修改，将多个分类面的参数求解合并到一个最优化问题中，通过求解该最优化问题“一次性”实现多类分类。这种方法看似简单，但其计算复杂度比较高，实现起来比较困难，只适合用于小型问题中；

（2）间接法，主要是通过组合多个二分类器来实现多分类器的构造，常见的方法有one-against-one和one-against-all两种。

一对多法（one-versus-rest,简称OVR SVMs）

训练时依次把某个类别的样本归为一类,其他剩余的样本归为另一类，这样k个类别的样本就构造出了k个SVM。分类时将未知样本分类为具有最大分类函数值的那类。

假如我有四类要划分（也就是4个Label），他们是A、B、C、D。

于是我在抽取训练集的时候，分别抽取

（1）A所对应的向量作为正集，B，C，D所对应的向量作为负集；

（2）B所对应的向量作为正集，A，C，D所对应的向量作为负集；

（3）C所对应的向量作为正集，A，B，D所对应的向量作为负集；

（4）D所对应的向量作为正集，A，B，C所对应的向量作为负集；

使用这四个训练集分别进行训练，然后的得到四个训练结果文件。

在测试的时候，把对应的测试向量分别利用这四个训练结果文件进行测试。

最后每个测试都有一个结果f1(x),f2(x),f3(x),f4(x)。

于是最终的结果便是这四个值中最大的一个作为分类结果。

评价：

这种方法有种缺陷,因为训练集是1:M,这种情况下存在biased.因而不是很实用。可以在抽取数据集的时候，从完整的负集中再抽取三分之一作为训练负集。

一对一法（one-versus-one,简称OVO SVMs或者pairwise）

其做法是在任意两类样本之间设计一个SVM，因此k个类别的样本就需要设计k(k-1)/2个SVM。

当对一个未知样本进行分类时，最后得票最多的类别即为该未知样本的类别。

Libsvm中的多类分类就是根据这个方法实现的。

假设有四类A,B,C,D四类。在训练的时候我选择A,B; A,C; A,D; B,C; B,D;C,D所对应的向量作为训练集，然后得到六个训练结果，在测试的时候，把对应的向量分别对六个结果进行测试，然后采取投票形式，最后得到一组结果。

投票是这样的：
A=B=C=D=0;
(A,B)-classifier 如果是A win,则A=A+1;otherwise,B=B+1;
(A,C)-classifier 如果是A win,则A=A+1;otherwise, C=C+1;
...
(C,D)-classifier 如果是A win,则C=C+1;otherwise,D=D+1;
The decision is the Max(A,B,C,D)

评价：这种方法虽然好,但是当类别很多的时候,model的个数是n*(n-1)/2,代价还是相当大的。