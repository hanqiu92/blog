---
title:       "How to Develop Your Own Dual Simplex Solver I: Introduction"
categories:  Study Notes
tags:
- Linear Programming
mathjax:     true
date:        2020-02-15 00:00:00
---

在接下来这个系列中，我将介绍线性规划中的对偶单纯形法的基本原理，并示范如何实现一个简单的对偶单纯形法求解器。

<!--more-->

## 写作动机 Motivation

这个系列的写作主要有以下几个目的：
1. 对偶单纯形法是目前线性规划和混合整数规划求解器中的核心模块。为了更好地利用已有的求解器资源，有必要了解其具体运作机理和局限性。
2. 为后续介绍混合整数规划求解原理的系列提供基础。
3. 对文字与代码结合的写作方式的首次尝试。
4. 完成一个简易的线性规划求解器，后续可能可以用于自己的一些小项目中。

基于上述动机的优先级，在本系列的写作/代码中，我将更注重**直观性(intuition)**和**可读性**，而选择性地忽略一些目前主流求解器中必备的、但与对偶单纯形法本身关系不大的计算优化技术。（还有一个主要的原因是这些技术都具有一定的技术壁垒，个人实现的时间成本较高。）

在本系列的写作中，我所用到的参考资料包括A. Koberstein的[博士论文](https://www.researchgate.net/profile/Achim_Koberstein/publication/35632487_The_dual_simplex_method_techniques_for_a_fast_and_stable_implementation_Elektronische_Ressource/links/0a85e52ef5144e9031000000/The-dual-simplex-method-techniques-for-a-fast-and-stable-implementation-Elektronische-Ressource.pdf)和相关引用文献，以及开源求解器[Clp](https://github.com/coin-or/Clp)、[SoPlex](http://soplex.zib.de/)的代码。要顺畅地理解本系列中的内容，阅读者需要对线性规划理论有一定的了解。

## 写作方式 Format

对偶单纯形法里用到的许多技术都是为了解决实际算法执行过程中出现的问题而引入的。因此，现有算法实现的效果评估对理解后续算法思路至关重要。为了简化从算法原理到代码实现到获得效果反馈之间的流程，在本系列的写作中，我将采用blog post加上ipython notebook的展示形式：在blog中，我主要对算法的基本思路进行讨论；而在notebook中，我将结合具体代码实现对一些算法细节进行介绍。

## 写作计划 Agenda

在本系列的第一篇文章（也就是本文）中，我将介绍线性规划求解的预备知识，包括问题的定义、解的相关概念等。
第二篇文章将介绍对偶单纯形法的基本流程，并结合简单例子分析其几何意义。
第三篇文章将介绍初始对偶可行解的寻找方法以及计算过程中数值误差的处理，以进一步完善对偶单纯形法的求解流程，保障求解结果的可靠性。
第四篇文章将介绍一系列常用的预处理技术，以进一步提升对偶单纯形法的求解效率。
在第五篇文章（最后一篇）中，我将就前述讨论进行总结，并针对求解器的设计和使用给出一些个人建议。


## 预备知识 Preliminary

**问题(problem)的定义。**线性规划的一般形式为
$$
\begin{equation}
  \begin{aligned}
    \min \ & c^T x \\\\
    s.t. \ & Ax = b, \\\\
           & l \leq x \leq u.
  \end{aligned}
  \label{eq:primal}\tag{1.1}
\end{equation}
$$

其中，决策变量$x$是一个$m$维的向量，$A$是一个$n\times m$维的矩阵，其中$n$为约束的个数。下界$l$和上界$u$的取值可以为$\pm \infty$。问题$\eqref{eq:primal}$一般被称为原始问题。根据对偶理论，问题$\eqref{eq:primal}$的对偶问题可以写为
$$
\begin{equation}
  \begin{aligned}
    \max \ & b^T \lambda + u^T s_u + l^T s_l \\\\
    s.t. \ & A^T \lambda + s_u + s_l = c, \\\\
           & s_l \geq 0, \ s_u \leq 0.
  \end{aligned}
  \label{eq:dual}\tag{1.2}
\end{equation}
$$

其中，决策变量$\lambda$和$s$分别为$n$维和$m$维的向量。

**解(solution)的评估**。给定一组解$(x,\lambda,s)$，如何判断其是否可行及是否最优？对于问题$\eqref{eq:primal}$和$\eqref{eq:dual}$，相应解$x$和$(\lambda,s)$的可行性可以直接通过约束的满足情况进行判断。解的最优性则可以通过以下的KKT条件进行判断：
$$
\begin{equation}
  \begin{aligned}
    & Ax = b, \\\\
    & l \leq x \leq u, \\\\
    & A^T \lambda + s_u + s_l = c, \\\\
    & s_l \geq 0, \ s_u \leq 0, \\\\
    & s_u(x - u) = 0, \ s_l(x - l) = 0.
  \end{aligned}
  \label{eq:KKT}\tag{1.3}
\end{equation}
$$

容易看出，KKT条件$\eqref{eq:KKT}$包括原始和对偶问题的可行性条件，以及互补松弛条件
$$s_u(x - u) = 0, \ s_l(x - l) = 0. \label{eq:compSlack}\tag{1.4}$$

**基(basis)的定义。**在单纯形法中，一个基本的概念是基。不妨假设原始问题的可行域$\mathcal{A}=\\{Ax = b,l \leq x \leq u\\}$是一个多面体。从几何的角度分析我们可知，原始问题$\eqref{eq:primal}$至少有一个最优解对应着多面体$\mathcal{A}$的顶点。因此，要求解原始问题的最优解，只需要找出多面体$\mathcal{A}$所有顶点中最优的一个即可。而根据线性代数知识，$\mathcal{A}$中的任意一个顶点都可以用一组（广义）基$(B,L,U)$表示：$x\_L = l\_L, x\_U = u\_U, x\_B = A\_B^{-1}(b - A\_Lx\_L - A\_Ux\_U)$，其中$B,L,U$为下标集合$\\{1,2,\cdots,m\\}$的一个分割。（**注：在一般的讨论中，基仅指集合$B$。在本系列中，我称$B$为狭义基。**）因此，对最优顶点的搜索可以转化为对最优基$(B,L,U)$的搜索。单纯形法的基本思路就是采用局部搜索的方式来求解最优基；原始单纯形法和对偶单纯形法的主要区别在于是在原始问题还是在对偶问题的可行域内进行局部搜索。

**基于基的解。**在给定一组基$(B,L,U)$时，利用互补松弛条件$\eqref{eq:compSlack}$，KKT条件$\eqref{eq:KKT}$可以改写为
$$
\begin{equation}
  \begin{aligned}
    & A\_B x\_B = b - A\_Lx\_L - A\_Ux\_U, \\ x\_L = l\_L, \\ x\_U = u\_U, \\\\
    & A\_B^T \lambda = c\_B, \\ s\_{l,L} = c\_L - A\_L^T \lambda, \\ s\_{u,U} = c\_U - A\_U^T \lambda, \\\\
    & s\_{l,B} = s\_{u,B} = s\_{u,L} = s\_{l,U} = 0; \\\\
    & l\_B \leq x\_B \leq u\_B, \\ s\_{l,L} \geq 0, \\ s\_{u,U} \leq 0.
  \end{aligned}\label{eq:KKT_decompose}\tag{1.5}
\end{equation}
$$

因此，根据公式$\eqref{eq:KKT_decompose}$中的前三式，基$(B,L,U)$不但可以决定原始问题的解$x$，还可以决定对偶问题的解$(\lambda,s)$；而且，如果这组解$(x,\lambda,s)$同时满足可行性约束$l\_B \leq x\_B \leq u\_B, \\ s\_{l,L} \geq 0, \\ s\_{u,U} \leq 0$，则其必为最优解。**注：**特别地，当上下界$l,u$有界时，我们总能通过交换$L$和$U$中的下标来保证$s\_{l,L} \geq 0, \\ s\_{u,U} \leq 0$，从而对于任意$B$，我们都能找到相应的$L$和$U$使得对应的对偶解$(\lambda,s)$可行。这部分地解释了为何在通常情况下对偶方法效率更高。

**数值误差的影响。**通过基$(B,L,U)$来表示解$(x,\lambda,s)$的一个好处是互补松弛条件$\eqref{eq:compSlack}$永远是默认被满足的。此时，根据KKT条件$\eqref{eq:KKT}$的定义，对解$(x,\lambda,s)$的最优性检验可以减弱为可行性检验。同时，对原始解$x$和对偶解$(\lambda,s)$的统一表示在一定程度上能提高整个求解流程对于数值误差的稳健性。下面来看一个简单例子。假设在求解过程中的某一步中，存在某个下标$i\in N = L\cup U$，对应的对偶变量$s\_i = s\_{l,i} + s\_{u,i}$的真实取值在0附近。受到数值误差的影响，实际计算的$s\_i$可能是一个微小的正值或者负值。现在，如果我们要手动维持互补松弛条件，那么相应的原始变量$x\_i$的取值很可能会在$l\_i$和$u\_i$之间来回波动。当这两个值的差异较大时，迭代优化过程会受到严重干扰。而如果我们不严格地维持互补松弛条件，则有可能在迭代优化过程中原始解和对偶解相关性不断减弱，从而导致算法中一些需要同时使用原始解和对偶解的计算失效，进而导致算法不收敛。

下面，我们进入[代码实现部分](https://github.com/hanqiu92/dual_simplex_solve/blob/master/1_preliminary.ipynb)。
