---
title: 'Integer Programming Note II: Formulation Strength & Size'
categories: Lecture Notes
tags:
- Integer Programming
mathjax: true
date: 2016-10-04 11:30:00
---

Assume we need to solve a mixed integer linear program (MILP), and the feasible region is $S\subseteq R^{n_1}\times Z^{n_2}$.  For some reasons we don't have a simple formulation of $S$, and we want to use some polyhedron $P=\\{(x,y)\in R^{n_1+n_2}\times R^{m_1+m_2}: Ax+By\leqslant b\\}$ and its subset $P_I=P\cap R^{n_1}\times Z^{n_2}\times R^{m_1}\times Z^{m_2}$ to represent $S$. Notice that we can use auxiliary variables $y$; these representation are called extended (sometimes we MUST have some integer variables $y$ to construct a valid formulation; in this case, the formulation is not extended). One important question is: what is a good representation?

<!--more-->

It is clear that the projection of $P_I$ on $R^{n_1}\times Z^{n_2}$ should be the same as $S$, or we will either miss some feasible points, or include some infeasible points. In formal language, $P_I$ is a formulation of $S$ iff

$$S=Proj_x(P_I)$$

## Formulation Strength, Integral/Ideal Formulation, and Sharp Formulation

In common approach to solve MILP, we will relax $P_I$ to $P$ and solve the "root" LP. Then we do branch and bound (B&B) to force $y$ to be integer. One simple observation is that, if our LP solution $(x,y)$ is already in $P_I$, then we will save a lot of computation. This property is called (locally) integral/ideal, and is defined as:

$$ext(P)\subseteq R^{n_1}\times Z^{n_2}\times R^{m_1}\times Z^{m_2}.$$

In the above definition, $ext(P)$ is the set of all extreme point (and thus potential LP solution) in $P$.

Integrality is an important property. Many combinatorial optimization problem, including network flow and bipartite graph matching, can be formulated as an IP with integral formulation, and thus solved by standard LP solver. Sometimes, even if we cannot design or proof a globally integral formulation, we might still achieve great computational improvement by constructing ideal formulation for a subset of constraints.

Now we introduce a slightly weaker property. A formulation $P_I$ is called sharp, if

$$conv(S)=Proj_x(P).$$

In case of non-extended formulation, sharpness is equivalent with integrality; but when we have auxiliary variables, sharp formulation is not as good as integral ones. One major reason is that we might have an extreme point $(x,y)$ of $P$ with fractional $y$ for some integer variables. In this case, even if $x$ is optimal for $S$, we still need to do extra B&B on $y$. But generally sharp formulation is good enough for practical use.

## Formulation Size and Extended Formulations

Sometimes, formulation with only original variables needs to use a large number of linear constraints, while extended formulation can be much smaller. For example, consider

$$S=\\{x\in Z^n: \sum_i|x_i|\leqslant 1\\}.$$

If we are only allowed to use $x_i$ in the formulation, we need to use $2^n$ constraints:

$$\sum_i s_ix_i\leqslant 1, \forall s\in \\{-1,1\\}^n.$$

But if we consider auxiliary variables $y_i=|x_i|$, we only need $2n+1$ constraints:

$$\sum_i y_i\leqslant 1$$

and

$$-y_i\leqslant x_i\leqslant y_i, \forall i.$$

In general, projection of high dimensional object on subspace will have much complicate structure. One question we might ask is: how much gain we can make from using auxiliary variables?

Consider polyhedron $P=\\{(x,y)\in R^{n+m}: Ax+By\leqslant b\\}$. If we let $C=\\{\mu\in R^l_+: B^T\mu=0\\}$, then we have (by Farkas' lemma)

$$\begin{aligned} Proj_x(P)&=\\{x\in R^n: \exists y\in R^m s.t. (x,y)\in P\\}\\\\ &=\\{x\in R^n: \mu^TAx\leqslant \mu^Tb, \forall \mu\in ray(C)\\}. \end{aligned}$$

Notice that $B^T$ is in size $m\times l$; so the size of $ray(C)$ is controlled by $m$ and $l$

$$Card(ray(C))\leqslant \binom{l}{m+1},$$

which is a polynomial of $l$ if $m$ is fixed. Thus, to have simple extended formulation, we need to use more auxiliary variables when the problem size increases.

There is also a proposition showing that, if we have some knowledge about the dimension of polytope $P$, we can give tighter bound on the formulation size. Consider the following definition:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1) The slack matrix of polytope $P=\\{x\in R^n: Ax\leqslant b\\}$, given $ext(P)=\\{v_j\\}^l$, is

$$S_{ij}=b_i-a^iv_j$$

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;where $a^i$ is the $i$th row of $A$.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2) The non-negative rank of a matrix $S\in R\_+^{m\times n}$ is the smallest $l$ s.t. $\exists U\in R\_+^{m\times l}$ and $V\_+^{l\times n}$, $S=UV$. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The non-negative rank of a polytope $P$ is the non-negative rank of its slack matrix.

Then, the proposition is:

For polytope $P\in R^n$ and its non-negative rank $l$, every extended formulation of $P$ has at least $l$ constraints, and there exists an extended formulation with $n+l$ variables and $O(n+l)$ constraints.

In practice, this proposition is not very useful since the decomposition $S=UV$ is a hard problem. But if the specific problem has good combinartorial structure and such decomposition exists in a natural way, then this proposition will be helpful in finding good extended formulation.

## Case Study: Union of Polytopes

Now let us turn to practical applications. We consider a simple problem: to construct a formulation of union of polytopes $\cup_i P_i$. First, we assume that we have access to both $H$-formulation and $V$-formulation of each polytope $P_i$:

$$\begin{aligned} P\_i&=\\{x: A^ix\leqslant b\_i\\}\\\\ &=\\{x: x=\sum\_j \lambda\_{v\_j} v\_j, \sum\_j \lambda\_{v\_j}=1,\lambda\_{v\_j}\in [0,1], v\_j\in ext(P\_i)\\}. \end{aligned}$$

1) One traditional formulation technique is to make copies of $x$ for each $P_i$, and use auxiliary variables $y_i\in \\{0,1\\}$ to weight these copies. For $H$-formulation, this can be written as

$$\begin{aligned} A^ix^i&\leqslant b^iy_i\\\\  \sum x^i &= x\\\\  \sum y^i &=1\\\\ y_i &\in \\{0,1\\}, \forall i. \end{aligned}$$

For $V$-formulation, we can make copies of $\lambda_v$:

$$\begin{aligned} \sum\_i\sum\_v v\lambda\_v^i &=x\\\\  \sum\_v \lambda\_v^i &= y\_i\\\\  \sum y^i &=1\\\\ \lambda^i &\in R\_+^{ext(P\_i)} \\\\ y\_i &\in \\{0,1\\}, \forall i. \end{aligned}$$

Both formulations are integral; but we have constructed too many auxiliary variables. In case of complex $P_i$, these formulations can be huge, and runtime memory becomes an issue. So we might need to consider non-extended formulations. But if $P_i$ have simple forms, these formulations are very efficient (even though we can still improve it with encoding, which will be discussed in next post).

2)  To construct non-extended formulation, we need to construct a set of shared constraints for all $P_i$, and use binary variables $y_i$ to signal activation of tighter constraints for each $P_i$. For $H$-formulation, we can consider shared matrix

$$D= \begin{bmatrix} A_1\\\\  \vdots\\\\ A_n\end{bmatrix}$$

and formulation

$$\begin{aligned} Dx\space{}&\leqslant \sum d^iy\_i\\\\  \sum y^i &=1\\\\ y\_i&\in \\{0,1\\}, \forall i. \end{aligned}$$

In general, to ensure feasibility, we need to use big-$M$ in rows of $d^i$ not belonging to $A_i$. Thus, in most of cases this formulation is neither ideal or sharp. Comparing with a global big-$M$, specific big-$M$ for each entry will lead to stronger formulation, but computation of these big-$M$ relies on actual structure of $P_i$ and might not be trivial.

To avoid the complexity of $A_i$, we can use $V$-formulation, where the structures of $\lambda_v$ are simplexes. Following the same shared constraints idea, $V$-formulation has a much simpler form:

$$\begin{aligned} \sum\_i\sum\_v v\lambda\_v &=x\\\\  \sum\_v \lambda\_v &= 1\\\\  \lambda\_v\space & \leqslant \sum\_{v\in ext(P\_i)} y\_i \\\\ \sum y^i &=1\\\\  y\_i &\in \\{0,1\\}, \forall i. \end{aligned}$$

This formulation is sharp but not ideal; so it is not as strong as the extended ones, but have much smaller size. Comparing with non-extended $H$-formulation, it is much stronger. Also, we can add some constraints to make it ideal, by applying technique called  embedding, which will also be discussed in next post.

In summary, we discuss four different formulations based on different approaches. It is hard to tell which is the best in general, and decision should be made based on specific problem structure. A general advice is to try all formulation with small instance, select the best few, and put them into parallel computation in practice.

## Reference

Vielma, J. P. (2015). Mixed integer linear programming formulation techniques. SIAM Review, 57(1), 3-57.
