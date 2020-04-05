---
title:       "How to Develop Your Own Dual Simplex Solver II: The Dual Simplex Algorithm"
categories:  Study Notes
tags:
- Linear Programming
mathjax:     true
date:        2020-03-15 00:00:00
---

在这篇文章中，我将介绍对偶单纯形法(Dual Simplex, DS)的基本流程，并结合一个简单的例子分析其几何含义。

<!--more-->

## 算法流程 The Algorithm

我们首先来回顾线性规划问题和解的定义。一个线性规划问题可以由以下的原始和对偶问题组表示：
$$
\begin{equation}
  \begin{aligned}
    \min \ & c^T x \\\\
    s.t. \ & Ax = b, \\\\
           & l \leq x \leq u.
  \end{aligned}
  \label{eq:primal}\tag{1.1}
\end{equation}
$$ $$
\begin{equation}
  \begin{aligned}
    \max \ & b^T \lambda + u^T s_u + l^T s_l \\\\
    s.t. \ & A^T \lambda + s_u + s_l = c, \\\\
           & s_l \geq 0, \ s_u \leq 0.
  \end{aligned}
  \label{eq:dual}\tag{1.2}
\end{equation}
$$

其中，$A$是一个$n\times m$维的矩阵（$n\leq m$），决策变量$x$和$s$是$m$维的向量，$\lambda$是$n$维的向量。该问题的一组基(basis)定义为下标集合$\\{1,2,\cdots,m\\}$的一个分割$(B,L,U)$；而每组基$(B,L,U)$通过以下方式对应一组解$(x,\lambda,s)$：
$$
\begin{equation}
  \begin{aligned}
    & A\_B x\_B = b - A\_Lx\_L - A\_Ux\_U, \\ x\_L = l\_L, \\ x\_U = u\_U, \\\\
    & A\_B^T \lambda = c\_B, \\ s\_{L} = c\_L - A\_L^T \lambda, \\ s\_{U} = c\_U - A\_U^T \lambda, \\ s\_{B} = 0. \\\\
  \end{aligned}\label{eq:KKT_decompose}\tag{1.3}
\end{equation}
$$

根据优化理论，一组基于基$(B,L,U)$的解$(x,\lambda,s)$为最优解，当且仅当其满足以下可行性约束
$$l\_B \leq x\_B \leq u\_B, \\ s\_{L} \geq 0, \\ s\_{U} \leq 0. \label{eq:cons}\tag{1.4}$$

对偶单纯形法(DS)的基本思路，可以认为是在保证对偶可行性$s\_{L} \geq 0, \\ s\_{U} \leq 0$（即原始最优性）的情况下，不断通过基空间上的局部搜索来优化对偶目标函数，直到原始可行性条件$l\_B \leq x\_B \leq u\_B$（即对偶最优性）被满足。简单来说，在每一步迭代中，给定一组对偶可行的基$(B,L,U)$和对应的解$(x,\lambda,s)$，

* 如果原始变量也可行$l\_B \leq x\_B \leq u\_B$，则此解为最优解，算法停止；
* 否则，从不可行的下标（$x_i > u_i$或$x_i < l_i$）中，选取其中一个$i\in B$离开$B$，从而令对应的$x_i$变得可行（并令对偶目标函数上升）。这一阶段被称为pricing/CHUZR阶段。然后，选取另一个下标$j \notin B$进入$B$，使得新的基$(B',L',U')$对应的解$(x',\lambda',s')$仍是对偶可行的。这一阶段被称为ratio test/CHUZC阶段。生成新的基后，进入下一步迭代。

下面，我将介绍DS迭代步骤的具体细节。不妨设给定的对偶可行的基和解分别是$(B,L,U)$和$(x,\lambda,s)$，且该解不是原始可行的。
1. （pricing/CHUZR阶段）在这一阶段中，我们需要选取一个原始不可行的列$i$离开$B$。当$i$离开$B$时，取决于$x\_i$是超过上界还是低于下界，应令$i$进入$U$或$L$。此时，对应的对偶变量$s\_i$将根据方向$$
  d\_i =
  \begin{cases}
    1 & x\_i < l\_i \\\\
    -1 & x\_i > u\_i
  \end{cases}\tag{1.5}
$$移动变成非零值。而根据系统\eqref{eq:KKT_decompose}，每当$s\_i$沿着方向$d\_i$变化一单位时，其他对偶变量$(\lambda,s)$的变化量应为
$$ \delta \lambda = d_i \cdot -A_B^{-T}e_i, \ \delta s = d_i\cdot A^TA_B^{-T}e_i; \tag{1.6}\label{eq:dual_delta}
$$ 其中$e_i$是在位置$i_B$上为1、其他位置为0的$n$维向量，而$i_B$是下标$i$在$B$中的位置。这些变化会进一步影响对偶问题\eqref{eq:dual}的目标函数。通过计算可以发现，$s\_i$每单位变化所导致的对偶目标函数变化值正好是$x_B$的越界量(primal infeasibility)$$
  \Delta x\_i =
  \begin{cases}
    l\_i - x\_i & x\_i < l\_i \\\\
    u\_i - x\_i & x\_i > u\_i \\\\
    0 & otherwise
  \end{cases}\tag{1.7}
$$的绝对值。因此，一种简单的选择$i$的方式是选单位对偶目标增量最大的一个：$\arg\max\_{i\in B} |\Delta\_i|$；这种方法称为Dantzig规则。然而，大量的计算经验表明，一种名为**dual steepest edge (DSE)**的选择方法要高效得多。这种方法近似地考虑了对偶变量$s$的**总变化**所带来的对偶目标增量；具体来说，这种方法选择 $$
  i = \arg\max\_{i\in B} \frac{|\Delta x\_i|}{\| A\_B^{-T}e\_i \|\_2},\tag{1.8}\label{eq:DSE}
$$其中，$|A\_B^{-1}e\_i| = |\delta \lambda|$正是对偶变量$\lambda$的改变方向。直观来看，$\| A_B^{-T}e_i \|\_2$越小，那么$\|\delta s\|\_2$就可能越小，从而在可行性约束下对偶步长$\alpha\_{dual}$（$s\_i$的可变化量）就可能越长，因此，式\eqref{eq:DSE}中的估计值可以认为是对对偶目标总增加值的近似。
最近，有研究者提出了一种新的pricing思路：positive edge(PE)，并通过数值实验说明在DSE中加入PE能显著降低迭代次数和计算时长。目前开源求解器CLP中实现了该方法；有兴趣的读者可以进一步参考[这篇论文](https://hal.archives-ouvertes.fr/hal-02099584/document)。
2. （ratio test/CHUZC阶段）在这一阶段中，我们需要找到一个列$j$进入$B$。这里的主要选择标准是要保证对偶变量的可行性；由于变量$\lambda$是无界的，因此只需要关注变量$s$的可行性。根据式\eqref{eq:dual\_delta}，当对偶步长为$\alpha\_{dual}$时，$s$会变为$s + \alpha\_{dual}\cdot \delta s$；而随着$\alpha\_{dual}$的增加，在某一时刻，$s$中某个元素$s\_j\notin B$会从非零值变为0。注意到在约束\eqref{eq:cons}中，$j$的属性（$j\in L$或者$j\in U$）对应了$s\_j$的正负性，因此，在$j$的属性不变时，如果$\alpha\_{dual}$继续增加，则$s$会变得对偶不可行。这意味着此时算法需要停下，令$j$进入$B$，然后进入下一步迭代。
然而，我们也可以通过改变/翻转$j$的属性(bound flipping)，使得当$\alpha\_{dual}$继续增加时，$s$仍是对偶可行的。问题在于，根据系统\eqref{eq:KKT_decompose}，$j$的翻转会导致原始变量$x\_B$也发生变化，有可能会使原始不可行的程度变大，甚至让迭代过程进入死循环(cycling)。因此，这里需要有一个标准来判断是否应该翻转$j$。由于DS迭代过程的目标是优化对偶目标函数，算法可以分析在$j$翻转后，继续增加$\alpha\_{dual}$是否仍能够带来对偶目标的收益。如果收益为正，那么就翻转$j$，继续增加$\alpha\_{dual}$，并找下一个下标$j$；否则，令$j$进入$B$，然后继续迭代。
根据前面的讨论，当没有$j$被翻转时，每增加单位对偶步长会为对偶目标增加$|\Delta x\_i|$（下称梯度）；而通过进一步的计算可知，每当一个$j$发生翻转时，梯度会下降$(u\_j - l\_j) |\delta s\_j|$。因此，步长$\alpha\_{dual}$可以不断增加，直到遇到一个$j$使得梯度$|\Delta x\_i| - \sum\_j (u\_j - l\_j) |\delta s\_j|$变为负值；此时，继续扩大$\alpha\_{dual}$会降低对偶目标。这种在迭代中对变量$j$的属性进行动态翻转的思路称为**bound flipping ratio test (BFRT)**。实际计算表明，BFRT能够显著降低DS的迭代次数，是各类高性能DS实现中的重要一部分。
3. 在选好$i$和$j$（即下一组基$(B',L',U')$）后，我们需要对解$(x,\lambda,s)$的取值进行更新。根据上述讨论，对偶变量$\lambda$和$s$可以直接通过对偶步长$\alpha\_{dual}$进行更新：
$$
  \begin{aligned}
    \lambda & \gets \lambda + \alpha\_{dual}\cdot \delta \lambda; \\\\
    s & \gets s + \alpha\_{dual}\cdot \delta s. \\\\
  \end{aligned}\tag{1.9}
$$ 而对于原始变量$x$，$x\_{N/\\{j\\}},(N=L\cup U)$的变化值$\Delta x\_{N/\\{j\\}}$和$x\_i$的变化值$\Delta x\_i$可以直接根据新基$(B',L',U')$确定，因此只需要计算$x\_{B}$和$x\_{j}$。根据系统\eqref{eq:KKT_decompose}，$x\_B$和$x\_j$的变化值需要满足以下条件
$$
A\_B \Delta x\_B + A\_j \Delta x\_j + A\_{N/\\{j\\}} \Delta x\_{N/\\{j\\}} = 0 \tag{1.10}
$$ 因此，可以首先令
$$
    x\_B \gets x\_B - A\_B^{-1}A\_{N/\\{j\\}} \Delta x\_{N/\\{j\\}}; \tag{1.11}
$$ 然后，按以下方式计算原始步长$\alpha\_{primal}$，并更新$x\_B$和$x\_j$：
$$
  \begin{aligned}
    \delta x\_B & = A\_B^{-1}A\_j; \\\\
    \alpha\_{primal} & = -(e\_i^T A\_B^{-1}A\_{N/\\{j\\}} \Delta x\_{N/\\{j\\}}+\Delta x\_i) / e\_i^T\delta x\_B; \\\\
    x\_j & \gets x\_j + \alpha\_{primal}; \\\\
    x\_B & \gets x\_B - \alpha\_{primal}\cdot \delta x\_B.
  \end{aligned}\tag{1.12}
$$



## 几何理解 Geometric Understanding

下面，我将通过一个toy problem，介绍DS算法和BFRT技术在几何上的意义。考虑以下原始问题
$$
\begin{equation}
  \begin{aligned}
    \min\_x \ & x\_3 \\\\
    s.t. \ & x_1 + x_2 + \frac{10}{3} x_3 = 3, \\\\
           & 0 \leq x\_1,x\_2,x\_3 \leq 1.
  \end{aligned}
  \label{eq:toy}\tag{1.13}
\end{equation}
$$ 下图对问题\eqref{eq:toy}进行了可视化：其中，红色虚线构成的平面对应线性约束$x_1 + x_2 + \frac{10}{3} x_3 = 3$，而蓝色实线构成的立方体对应边界约束$0 \leq x\_1,x\_2,x\_3 \leq 1$。从图中容易看出，问题\eqref{eq:toy}的可行域是由点$(0,0,0.9)$,$(1,0,0.6)$,$(1,1,0.3)$,$(0,1,0.6)$所围成的四边形，而最优解是点$(1,1,0.3)$。那么，DS算法是如何找到该最优解的呢？

<img width=450 src="/blogs/images/dual_simplex/BFRT_geometry.png">

首先，根据定义，基$(B,L,U)$对应的原始$x$是超平面$Ax=b$与立方体表面$x\_L=l\_L,x\_U=u\_U$的交点。因此，可以认为，包括DS在内的单纯形法的目标是在保持$x$在超平面$Ax=b$上的同时，寻找最优相交表面$x\_L=l\_L,x\_U=u\_U$。特别地，在问题\eqref{eq:toy}中$|B|=1$，因此每组基对应的表面是一条直线（两个固定维度加一个可变维度）；最优解$(1,1,0.3)$对应的基和直线分别是$B=\\{3\\},U=\\{1,2\\}$和$x\_1=1,x\_2=1$。

由于在DS过程中需要保证解的对偶可行性，下面，我将分析对偶变量$s$和对偶可行性的几何意义。假设给定基$(B,L,U)$，则通过简单计算可知，$s\_B = 0$，且$s\_j \ (j\in L\cup U)$代表原始变量$x$在超平面$Ax=b$上沿着$dx\_{(L\cup U)/\\{j\\}} = 0, dx\_j = 1$方向移动时，目标函数$c^T x$的增量。对应到问题\eqref{eq:toy}中，对于基$B=\\{3\\},L=\\{1,2\\}$和对应的解$(0,0,0.9)$，根据系统\eqref{eq:KKT_decompose}可以算得$s=(-0.3,-0.3,0)$；容易看出，$s\_1$和$s\_2$正分别是在超平面$x_1 + x_2 + \frac{10}{3} x_3 = 3$上从点$(0,0,0.9)$出发，沿着方向$(1,0,?)$和$(0,1,?)$到点$(1,0,0.6)$和点$(0,1,0.6)$时，目标函数$x\_3$的变动值。在搞清楚$s$的意义后，对偶可行性$s\_{L} \geq 0, \\ s\_{U} \leq 0$也就容易理解了：对偶可行性对应的是（在不考虑$x\_B$的可行性情况下的）原始最优性，即不存在局部优化方向。例如，在问题\eqref{eq:toy}中，对于基$B=\\{3\\},L=\\{1,2\\}$和对应的解$(0,0,0.9)$，$s=(-0.3,-0.3,0)$指出通过令$x\_1$从$l\_1$变成$u\_1$可以提高目标函数值，因此这组基是对偶不可行的。进一步地，从图上容易看出，除了最优解$(1,1,0.3)$外，其他对偶可行解都在直线$x_1+x_2=3$上。

下面，我将从对偶可行基$B=\\{1\\},L=\\{2,3\\}$和点$x=(3,0,0)$出发，使用DS算法来寻找最优解。首先是pricing阶段。在这一例子中，由于$|B|=1$，在pricing阶段不需要考虑变量选择的问题；但从图中可以大概推断在一般场景中pricing阶段变量选择的集合意义。具体来说，Dantzig规则是从所有不可行的原始变量中，选取离边界最远的一个；而DSE则进一步考虑超平面$Ax=b$与投影面$x\_i=u\_i$或$x\_i=l\_i$的夹角：如果夹角比较小，则意味着在变量$x\_i$到达投影面时其他变量的值会有很大变动，因此有必要先选择。

现在，pricing阶段的结果显示，下标$1$将离开$B$进入$U$，同时原始变量$x\_1$将从3变为$u\_1 = 1$。那么，对偶变量$s$会怎么变化呢？简单计算可知，$s=c=(0,0,1)$，而$\delta s=(-1,-1,-10/3)$。因此，为了保证对偶可行性$s\_2 \geq 0$，对偶步长必须为0，且$s\_2$进入$B$。换句话说，在ratio test阶段，算法从下标$2$和$3$中选择了下标2。从图中可以看出这一选择的合理性：如果选择了下标3，则下一个解会变成$(1,0,0.6)$，而这个解是对偶不可行的（在局部范围可以沿着方向$dx=(0,1,-0.3)$优化）。

在上面的计算中，通过一次迭代，算法从点$(3,0,0)$移动到了点$(1,2,0)$。简单的计算表明，再通过一次DS迭代就可以抵达最优解$(1,1,0.3)$。如果把DS算法的迭代路径$(3,0,0) \to (1,2,0) \to (1,1,0.3)$画在图上，那么可以大致认为，在几何上，**DS算法是在可行域的最优性一侧的外部不断向可行域逼近直至到达可行域**。

下面，我将进一步分析BFRT技术的几何意义。仍然假设算法从对偶可行基$B=\\{1\\},L=\\{2,3\\}$和点$x=(3,0,0)$出发，并在pricing阶段选取了下标1离开$B$，而在ratio test阶段选取了下标2进入$B$。从图中可以看出，下标2进入$B$会导致$x\_2$超过了上界$l\_2 = 1$，从而在之后的迭代步骤中算法需要再次将其“拉回”可行域中。如果算法能够高效判断选中的下标是否会在迭代后越界，并且在会越界的情况下让其在边界上停住、并寻找下一个进入$B$的下标（相当于切换前进方向），那么就能降低后续的迭代次数。这正是BFRT技术的几何意义；事实上，通过简单计算可知，$|\Delta x_i| - (u_j - l_j) |\delta s_j|$表示了$x_j$翻转之后的变量$x_i$的越界量，因此通过该值可以判断是否应该切换方向：如果该值大于0，则$j$进入$B$之后$x_j$会越界（否则不足以消除$x_i$的越界量），反之亦然。通过BFRT，在DS的每次迭代中，对应下标集合$L\cup U$的原始变量的可行性都可以得到保证，而且解$x$会螺旋式地逼近投影面$x\_i=u\_i$或$x\_i=l\_i$。特别地，对于问题\eqref{eq:toy}，从基$B=\\{1\\},L=\\{2,3\\}$和点$x=(3,0,0)$出发并使用DS+BFRT，则只需要迭代一步，途径点$(2,1,0)$到达最优解$(1,1,0.3)$。

在了解了DS的基本原理之后，下面，我们进入[代码实现](https://github.com/hanqiu92/dual_simplex_solve/blob/master/2_dual_simplex.ipynb)。
