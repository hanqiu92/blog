---
title:       "How to Develop Your Own Dual Simplex Solver V: Summary"
categories:  Study Notes
tags:
- Linear Programming
mathjax:     true
date:        2020-05-01 00:00:00
---

在这篇文章中，我将对本系列之前的内容进行总结，并针对LP求解器的设计开发和使用给出一些个人建议。

<!--more-->

## 对偶单纯形法的工作原理 How does DS work?

从几何的角度看，标准形式线性规划（LP）
$$
\begin{equation}
  \begin{aligned}
    \min \ & c^T x \\\\
    s.t. \ & Ax = b, \\\\
           & l \leq x \leq u.
  \end{aligned}
\end{equation}
$$

的原始可行域是一个超平面（$Ax = b$）与一个高维立方体（$l \leq x \leq u$）的交集（是一个多面体）。一组基$(B,L,U)$首先通过$(L,U)$对应了立方体的一个表面$x_L=l_L,x_U=u_U$，然后通过$B$对应了超平面与该立方体表面的交点$x$：
$$x_B = A_B^{-1}(b - A_L l_L - A_U u_U).$$

对偶单纯形法（DS）的基本工作原理是不断通过局部搜索来寻找更好的基$(B,L,U)$。在每一次迭代中，对于一组给定的基$(B,L,U)$，
* 如果所有变量都可行$l \leq x \leq u$，则此解为最优解，算法停止；
* 否则，存在不可行的变量$x_j > u_j$或者$x_j < l_j$。此时，先选取其中一个$j$（pricing阶段），然后沿着一个方向缩减不可行性$\delta_j = \max(x_j - u_j,l_j - x_j)$，直到遇到另一个立方体表面为止（ratio test阶段）。如果进一步使用了BFRT技术，则在缩减不可行性的阶段可以多次调整方向，螺旋式地逼近可行域，直到不能前进为止。容易看出停止的位置对应一组新的基$(B,L,U)$。

在pricing和ratio test这两个选择决策阶段中，使用不同的技术能够显著影响DS迭代过程的次数，但可能会带来计算上的成本：
1. 在pricing阶段中，相比简单地选用不可行指标$\delta_j$最大的变量（Dantzig规则），DSE方法能够大幅降低迭代次数，但是需要增加一定的计算量来维护DSE的权重；可以进一步通过随机采样一部分变量的方式来平衡迭代效率和计算效率。
2. 在ratio test阶段中，使用Harris RT能够避开一些数值不稳定的pivot从而降低迭代次数，但可能会引入一定的对偶不可行程度（dual infeasibility），从而对整体的数值误差控制提出了新的要求。

因此，在DS算法的实现中，往往需要对算法效率（迭代次数）、计算效率（计算量）、计算精度三者进行权衡。不过，还可以通过预处理、改进线性方程$Ax=b$求解效率等方式，全面提升DS算法的效率和精度。


## 线性规划求解器的高效使用 How to use LP solvers efficiently?

### 求解算法的选用

一般来说，DS比原始单纯形法（PS）的效率高。根据上述DS的工作原理，可以部分地解释为何如此：由于大部分问题的变量是双边有界的（$-\infty < l \leq x \leq u < \infty$），
* 一方面，找对偶可行解比原始可行解容易得多。相比于PS的第一阶段是为找可行解而找可行解，DS在找可行解的过程中也考虑了目标值（最优性）的影响，因此从整个求解流程上看效率更高。
* 另一方面，基结构$(B,L,U)$的多样性使得BFRT相比普通的ratio test能够大幅提高迭代效率。具体来说，在DS+BFRT中，首先决定哪个变量$j$离开狭义基$B$，然后决定哪个变量$j'$进入$B$。不妨假设$j'$原来在集合$L$中；当$j'$的属性需要变化时，它不仅可以进入$B$，也可以再次离开并进入$U$。如此反复，在DS+BFRT中，ratio test阶段不仅可以找到下一个$B$的元素，还能更新$U$和$L$。相比之下，在PS中，需要先决定哪个变量$j$进入$B$，再决定哪个$j'$离开$B$；在此过程中，$j'$是进入$L$还是$U$是被指定的，没有选择空间。

然而，根据问题结构的变化，上述认知的前提不一定成立。此时，DS、PS或者内点法（barrier）都可能效率更高。例如，对于一些网络流问题，由于原始可行解很好找，PS的效率不比DS差。一种更简单粗暴的解决方式是，可以先设计一个小的测试问题，对不同求解算法的效率进行分析，然后再对实际问题应用效率最高的一个。Gurobi中提供了一种concurrent求解选项：该选项高效地实现了对多种求解算法的同步调用。

另外，DS和PS都是普适的线性规划求解方法。在DS/PS中，线性方程求解$Ax=b$的效率很大程度上决定了整个求解器的效率；而由于其普适性，其对矩阵$A$的理解往往不如建模者的理解深入。在很多场景下，线性方程$Ax=b$的求解实际上对应了一个组合优化问题，在这种情况下使用组合算法效率往往更高。

### 求解器的选用

对于不需要考虑成本的使用者来说，基本只需要考虑Gurobi和Cplex，不仅因为它们求解效率高，而且相关的接口和文档也更完善。但是，如果需要考虑预算，那么可以考虑开源求解器Clp。Clp基本是开源中最快的DS求解器，而且社区也提供了很多调用接口（例如PuLP、CyLP），可以很方便与其他任务进行整合。

### 建模与输入的注意事项

通过代码实现，可以知道
1. 矩阵$A$的稀疏结构对于求解效率有至关重要的影响。因此，在使用建模语言（例如PuLP）时，可以对系数为零的元素进行过滤。
2. 输入变量的上下界能够一定程度提高DS的求解效率。因此，虽然预处理步骤能够从约束中推断出变量的上下界信息，但是在建模过程中可以避免引入类似$x_j \leq u_j$的约束。


## 对偶单纯形法求解器的开发 How to develop a DS solver?

基于某些原因，使用者可能需要从头开始实现一个简单的DS求解器。本系列中的尝试可以为此提供一些参考。特别地，我认为在整个求解器实现中，有两个任务最为关键：线性方程$Ax=b$的高效求解，以及对数值误差的控制流程设计。
1. 线性方程求解的常用思路是做LU分解。对于稠密矩阵，LU分解的算法已经非常成熟；但是LP问题的输入矩阵$A$一般很稀疏。通过分析和利用稀疏性，LU分解的效率可以显著提升。事实上，本系列的实现中调用了一些线性方程求解器，这些求解器内部实现了多种复杂的利用稀疏性的算法。由于该领域目前仍不断有新的算法的提出，这些求解器在不断地优化，个人去做线性方程求解的优化似乎效率较低。然而，在LP求解中调用外部的线性方程求解器可能会有一些内存甚至是IO上的额外开销，从而削弱了外部求解器的优势。同时，单纯形法中的重要一步是对LU分解结果的更新，而这一步似乎是LP问题独有的，目前我没有发现有通用线性方程求解器对这一步提供支持。这显著地降低了外部求解器的吸引力。
2. 通过本系列中的尝试，可以充分地认识到数值误差的控制流设计对于求解器稳健性和求解效率的重要性。特别地，对于PILOT系列等数值不稳定的问题，不同的控制流设计会导致求解效率有数量级的差异。在本系列的实现中，基本上完全通过对问题进行扰动实现误差控制；而实际上，在很多步骤中，可以结合数值误差产生的原因设计更高效的误差控制手段。当然，对这些特定场景下的控制方案进行统一管理并不是一件简单的事情。

要进一步提高求解器的效果，开发者可以参考已有的开源工作。在本系列的实现中，我参考了[Clp](https://github.com/coin-or/Clp)和[SoPlex](http://soplex.zib.de/)的代码，下面简单归纳一些基础认知。Clp的实现与A. Koberstein的[博士论文](https://www.researchgate.net/profile/Achim_Koberstein/publication/35632487_The_dual_simplex_method_techniques_for_a_fast_and_stable_implementation_Elektronische_Ressource/links/0a85e52ef5144e9031000000/The-dual-simplex-method-techniques-for-a-fast-and-stable-implementation-Elektronische-Ressource.pdf)中的描述的差别不是很大，但是引入了较精细的误差控制流程。一个主要的算法层面的区别是Clp没有进行第一/二阶段的区分，而是通过设置虚拟的上下界来保证对偶解可行性，然后动态地对这些虚拟的上下界进行管理，在必要的时候扩大这些虚拟上下界，直到对应的变量离开该界为止。虽然Koberstein的论文中提到这种方法，但是他没有选用，也没有做数值实验进行对比，可能是这种方案的效果强依赖与一些参数，而调参比较困难。Clp虽然求解效率很高，但是代码注释写得不够清楚。相比之下，SoPlex代码注释写得很好，阅读体验显著提升。在算法实现方面，相比Clp，SoPlex主要有以下两点区别：
1. DSE阶段没做采样优化效率；
2. 在BFRT阶段找对偶步长的时候使用排序循环，而不是像Clp一样用二分法；同时也没有加入Harris RT的元素来对数值误差进行处理。

建议有兴趣的读者可以自行阅读这两个求解器的相关代码。对于Clp，可以重点阅读ClpSimplexDual、ClpDualRowSteepest这两个文件。对于SoPlex，可以重点阅读spxsolve、steeppr、boundflippingrt这些文件。

最后，本系列中仅讨论了DS算法的序列化实现。目前，各主流求解器都实现了DS算法的并行化运行。在现有计算架构下，这能显著提升整体优化效率。此外，还有一系列基于分布式架构的实现。后续，可以进一步对相关技术进行探索。