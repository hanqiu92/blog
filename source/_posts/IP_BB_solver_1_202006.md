---
title:       "How to Develop Your Own Branch & Bound Solver I: Introduction"
categories:  Study Notes
tags:
- Integer Programming
mathjax:     true
date:        2020-06-01 00:00:00
---

在接下来这个系列中，我将介绍混合整数规划中的分支定界法(branch & bound, B&B)的基本原理，并示范如何实现一个简单的B&B求解器。

<!--more-->

## 写作动机 Motivation

这个系列的写作主要有以下两个目的：

1. 混合整数规划(MIP)问题在日常工作中很常见，但求解方法多样（调用求解器、设计启发式(heuristic)算法），且通常需要根据问题结构进行定制化。通过了解求解器的基本运作机理和局限性，可加深对MIP问题分析套路的认知，以更高效地协同利用各求解方法。具体来说，我将主要关注B&B相关的技术手段。
2. 完成一个简易的混合整数线性规划求解器，后续可能可以用于自己的一些小项目中。

在本系列的写作中，我所用到的参考资料包括T. Ahterberg的[博士论文](https://depositonce.tu-berlin.de/bitstream/11303/1931/2/Dokument_41.pdf)和相关引用文献，以及开源求解器SCIP的代码。要顺畅地理解本系列中的内容，阅读者需要对线性规划理论有一定的了解。在代码实现中，主要依赖接口[CyLP](https://github.com/coin-or/CyLP)实现对线性规划求解包[CLP](https://github.com/coin-or/Clp)的调用；评估的benchmark则选用的是开源求解器[CBC](https://github.com/coin-or/Cbc)，其效率比目前最快的求解器要差一个数量级左右。

## 写作计划 Agenda

在本系列的第一篇文章（也就是本文）中，我将介绍分支定界法(B&B)的基本思路及实现中的注意事项。
第二篇文章将介绍问题拆分(branching)和结点选择(node selection)模块的常用实现方法。
第三篇文章将回顾割(cut)方法的基本思路，介绍将其引入B&B求解流程中的常用手段及注意事项，并实现最基础的变量边界加强(bound tightening)方法。
在第四篇文章（最后一篇）中，我将对前述内容进行总结，并针对MIP的求解方法设计及求解器的使用给出一些个人理解和建议。

## 分支定界法 Branch & Bound Method

混合整数规划(MIP)问题的一般形式为
$$
\begin{equation}
  \begin{aligned}
    \min \ & c^T x \\\\
    s.t. \ & Ax = b, \\\\
           & l \leq x \leq u, \\\\
           & x \in \mathbb{R}^{m\_1} \times \mathbb{Z}^{m\_2}.
  \end{aligned}
  \label{eq:basic}\tag{1.1}
\end{equation}
$$

相比于线性规划(LP)问题，MIP的主要区别仅在于某些变量的取值必须是整数。因此，我们可以考虑以下求解思路：

1. 放松问题\eqref{eq:basic}的整数约束，并通过LP求解算法得到一个实数解$x$；
2. 如果$x$满足整数约束，则找到最优解；
3. 否则，从$x$中选出一个不满足整数约束的变量$x\_j$，将其可行域切分为$[l\_j,\lfloor x\_j \rfloor]$和$[\lceil x\_j \rceil,u\_j]$，以排除不可行解$x$；注意此时原问题的所有整数可行解都将落在某个子问题的可行域中，即此切分并不排除任一整数可行解。
4. 将切分可行域产生的子问题加入一个问题列表中，然后从中选出一个子问题重复上述步骤。如果该子问题产生一个整数可行解$x^*$，且其目标值不差于其他所有子问题的最优目标值的最小值（即全局目标值的下界），则此解即为最优解。

此即B&B方法的基本原理。B&B方法可被看作是一种特殊的基于LP求解器的分治法(divide and conquer)。据此，B&B方法具有以下性质：

* 可以提供优化下界，用于判断最优性(optimality gap)；
* 算法效率没有理论保证，求解过程中可能会产生指数多个子问题；特别地，不能保证在时间限制内找到可行解；
* 在子问题（结点，node）的生成（变量选择与可行域切分，branching）和选择(selection)上存在一定的灵活性；后续我们会看到，不同的实现会带来显著的效率差异。

上面对B&B流程的描述较为宽泛；在具体实现上，为了branching和node selection等模块的灵活性，我们需要一个更清晰的B&B求解框架。具体来说，我们可以参考T. Ahterberg的博士论文中对B&B方法的描述（算法2.1）：

1. 根据原始问题生成根结点$N\_0=R$；
2. 初始化问题结点队列$\mathcal{L} = \\{N\_0\\}$以及全局解$x=\emptyset$和目标值上下界$\hat{c},\check{c}=\infty$；
3. (Select) 从队列$\mathcal{L}$中选取一个问题结点$N\_j$，并令问题队列$\mathcal{L}=\mathcal{L}/\\{N\_j\\}$；
4. (Solve) 调用LP求解器求解问题$N\_j$；如果问题可行，则可得最优解$x\_j$和目标值$\check{c}\_j$（作为下界估计）；否则，令$x\_j=\emptyset,\ \check{c}\_j = \infty$；
5. (Process) 根据下界估计$\check{c}\_j$，对B&B树中结点$N\_j$与根结点$N\_0$路径上相关结点$N\_{j'}$的下界信息$\check{c}\_{j'}$进行更新，然后进行以下判断：
   1. 如果$\check{c}\_j$不小于全局最优目标值/上界$\hat{c}$，则可以直接忽略该结点；
   2. 如果$x\_j$整数可行且$\check{c}\_j < \hat{c}$，则更新全局解$x \gets x\_j$和$\hat{c} \gets \check{c}\_j$；
   3. (Branch) 否则，根据$x\_j$的情况，将问题$N\_j$拆分成子问题结点$N\_{j\_1},\cdots,N\_{j\_n}$，并将它们加入队列$\mathcal{L}$中。
6. 如果队列$\mathcal{L}$为空，返回$x,\hat{c},\check{c}$；否则，回到第3步。

接下来，我将根据上述框架，在[这个notebook](https://github.com/hanqiu92/branch_bound_solve/blob/master/1_branch_bound_tree.ipynb)中实现一个简单的B&B求解流程。
