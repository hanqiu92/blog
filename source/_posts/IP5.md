---
title: 'Integer Programming Note V: Decomposition'
categories: Lecture Notes
tags:
- Integer Programming
mathjax: true
date: 2016-10-04 12:06:00
---

## Inner Approximation: $V$-representation, Dantzig-Wolfe decomposition, and Branch and Price

Assume we are given problem

$$\begin{aligned} z\_I =\max & \ c^Tx \\\\ s.t. & \ A\_1 x \leqslant b\_1 \\\\  & \ A\_2 x \leqslant b\_2\\\\ & \\ x \in R^{n\_1} \times Z^{n\_2} \end{aligned}$$

and we know that LP relaxation for $Q=\\{A\_2 x \leqslant b\_2, x \in R^{n\_1} \times Z^{n\_2}\\}$ is tight but that for the whole problem is weak. One can then think of dual approach utilizing the tighter local constraint structure:

<!--more-->

$$z\_{D}=\min \\{z\_{LR}(\lambda): \lambda\in R^{m\_1}\_+\\}$$

with

$$z\_{LR}(\lambda)=\max\_{x\in Q} \\{c^Tx+\lambda^T(b\_1-A\_1x)\geqslant z\_I\\},$$

which implies (by LP duality)

$$z\_{D}=\max \\{c^Tx: A\_1x\leqslant b\_1, x\in conv(Q)\\}.$$

Assume we have set of extreme points $\\{v\_i\\}\_k$ for $conv(Q)$. Then

$$z\_{LR}(\lambda)=\lambda^Tb\_1+\max\_i (c^T-\lambda^TA\_1)v\_i$$

is a piecewise linear function of $\lambda$ and thus $z_{D}$ can be obtained by dynamically finding better $v_i$. Notice that this is exactly the same as the column generation method with Dantzig-Wolfe decomposition of $conv(Q)$ for $z_D$.

Now, we can solve $z_I$ by branch and price: at each node in the branching tree, we keep solving the pricing problem, i.e, to find better extreme point $v_i$, and adding these new $v_i$ to the master problem until we reach the optimal $z_D$; then we do branch and bound on $x$ until $x\in Q$.

The above approach can be seen as inner approximation, since we use a subset of extreme points to approximate the polyhedron. But it is paradoxical that in branching process, we are actually using outer approximation. So in general branch and price is not straightforward and easy to implement. But for problems with natural approach for pricing subproblem, e.g., airline scheduling, or there is no good way to use $H$-representation of the feasible region, branch and price is still a good option.

## Outer Approximation: $H$-representation and Benders decomposition

Compared to inner approximation, outer approximation aims at approximating the feasible region with fewer and simpler constraints. Then, valid cuts are generated dynamically by subproblem to push the optimum towards feasibility. Such approach is useful when not all constraints can be given explicitly. For example, consider the following problem:

$$\begin{aligned} z_I =\min & \\ c^Tx + d^Ty \\\\ s.t. & \\ A x + B y \geqslant b \\\\  & \\ x \in P_1\subseteq Z^{n_1} \\\\  & \\ y \in P_2\subseteq R^{n_2} \end{aligned}$$

and we want to decouple $y$ from $x$. We can define

$$\begin{aligned} g(x)= \min & \ d^Ty\\\\ s.t. & \ By\geqslant b-Ax\\\\ & \ y\in P_2\\\\ \end{aligned}$$

then the problem will be transformed into

$$\begin{aligned} z_I= \min & \ c^Tx+\theta\\\\ s.t. & \ x\in Proj_x(\\{(x,y): Ax+By\geqslant b\\})\cup P_1\\\\ & \ (x,\theta)\in epi(g)\\\\ \end{aligned}$$

In this case, both $Proj_x(\\{Ax+By\geqslant b\\})$ and $epi(g)$ can be very complex and no explicit $H$-representation is available. But recall that

$$Proj_x(\\{(x,y): Ax+By\geqslant b\\})=\\{x: \mu^T(Ax-b)\geqslant 0, \forall \mu\in ray(C)\\}$$

with $C=\\{\mu\in R_+^m: B^T\mu=0\\}$, while $g$ can be expressed in dual form

$$g(x)=\max \\{v^T(b-Ax): v\in ext (Q)\\}$$

with $Q=\\{\mu\in R_+^m: B^T\mu=d\\}$; so both set can be approximated by a collection of cuts (in same format). Thus, we can consider following outer approximation

$$\begin{aligned} z_I= \min & \ c^Tx+\theta \\\\ s.t. & \ u_i^T(b-Ax)\leqslant 0 \\\\ & \ v_j^T(b-Ax)\leqslant \theta \\\\ \end{aligned}$$

with $u_i$ and $v_j$ can be generated dynamically by solving subproblem

$$\max \\{v^T(b-Ax^*): v\in ext (Q)\\}.$$

Notice that this is exactly the same as the Benders decomposition and row generation in LP.

## Nonlinear MIP

In the end, I briefly introduce nonlinear MIP, which is related to the approximation methods above. The first step to solve nonlinear MIP is to transform nonlinear objective into nonlinear constraints. Nonlinear objective is not preferred since normally in that case the optimum is located in the interior and branching is very complex. Then, there are three kinds of approach:

1) NLP based approach: at each branching node, solve the NLP directly with interior point method, and perform next branching if necessary. This kind of approach is common in standard NLP solvers.

2) LP based approach: this approach is similar to the outer approximation introduced above. The feasible region is approximated by many linear constraints, and in the solving process the optimal solution will get closer and closer to the feasible region. But this method may have slow convergence rate.

3) Lifted LP based approach: this approach also consider linear approximation, but after the nonlinear feasible region being lifted to higher dimensional space. The advantage of doing this is that fewer linear cuts is needed to achieve the same approximation ratio (recall the gain in formulation size of extended formulation). Currently this approach is widely applied in many MINLP solvers.

In general, if you have an MINLP problem, the first thing to check is whether the problem can be formulated with second order cone, which can be efficiently dealt with in CPLEX/Gurobi. Then, if the problem is convex in a disciplined way, you may try cvx or Pajarito (links can be found in the reference). For other cases, you can try NLP solvers or design ad-hoc heuristics, but no general performance can be guaranteed.

## Reference

Lubin, M., Yamangil, E., Bent, R., & Vielma, J. P. (2015). Extended Formulations in Mixed-integer Convex Programming. arXiv:1511.06710. Available at: http://arxiv.org/pdf/1511.06710v3.pdf.

Grant, M., Boyd, S., & Ye, Y. (2008). CVX: Matlab software for disciplined convex programming. Available at: http://cvxr.com/cvx/
