---
title: 'Integer Programming Note III: Encoding & Embedding'
categories: Lecture Notes
tags:
- Integer Programming
mathjax: true
date: 2016-10-04 12:00:00
---

## Unbalanced Branching and Encoding

Consider constraint $\sum\_i x\_i= 1$ with $x\_i\in \\{0,1\\}$, which arises naturally in problem with selection (e.g. union of polytope $x\in \cup\_i P\_i$). Recall that, in standard MIP solver, $x\_i$ is first relaxed into continuous variable, and then forced to be integer with B&B. For the constraint above, when we branch on a fractional $x\_i$, we have two choices, $x\_i=1$ or $x\_i=0$, but the result of each branch is different. With $x\_i=1$, we immediate know $x\_j=0, \forall j\neq i$, even though they are still assumed continuous. But with $x\_i=0$, the optimal solution might still contain another fractional $x\_j$ and further branching is needed. In extreme case, we need to branch $n-1$ times to locate the optimal integer solution.

<!--more-->

The basic idea to deal with such unbalanced branching is to encode each $x\_i$ and design more efficient branching trees. In the above case, we can consider encoding

$$\begin{aligned} \sum b\_ix\_i &= y\\\\  \sum x\_i&= 1\\\\ y \ &\in \\{0,1\\}^m, \end{aligned}$$

in which $b\_i\in \\{0,1\\}^m$ is the encoding for $x\_i$.

Notice that, if we let $m=n$ and $b\_i=e\_i$, we will recover the original constraint  $\sum\_i x\_i= 1, x\_i\in \\{0,1\\}$. This encoding is called unary encoding; we already know it is inefficient. What other encoding can we use?

In practice, two encoding are widely used: binary encoding and incremental encoding. In binary encoding, $m=\lceil log\_2n \rceil$ and $b\_i$ is selected to be mutually different and balanced in number for each row. An example for $n=4$ is

$$\begin{pmatrix}0 & 0 & 1 & 1\\\\0 & 1 & 1& 0 \end{pmatrix} \begin{pmatrix} x\_1\\\\x\_2\\\\x\_3\\\\x\_4 \end{pmatrix}=\begin{pmatrix}y\_1\\\\y\_2\end{pmatrix}$$

Now, assume in root LP the optimal solution has $y\_1$ fractional. When we branch on $y\_1$, we find the branching tree is more balanced: if $y\_1=1$, then $x\_1$ and $x\_2$ are forced to be $0$; if $y\_1=0$, then $x\_3$ and $x\_4$ are forced to be $0$. For $y\_2$, the situation is similar. In extreme case, we only need to branch $m$ times, which is much better than unary encoding.

Incremental encoding is slightly different. We have $m=n-1$, but with $b\_i=\sum\_{j<i} e\_j$. An example for $n=4$ is

$$\begin{pmatrix}0 & 1 & 1& 1\\\\ 0 & 0 & 1 & 1\\\\ 0 & 0 & 0 & 1 \end{pmatrix} \begin{pmatrix} x\_1\\\\x\_2\\\\x\_3\\\\x\_4 \end{pmatrix}=\begin{pmatrix}y\_1\\\\y\_2\\\\y\_3\end{pmatrix}$$

This encoding implies that $y\_1\geqslant y\_2\geqslant y\_3$. In extreme case (branching from $y\_1$ to $y\_{n-1}$), we might still need to branch $n-1$ times; but if the problem has good structure, incremental encoding might even be better than binary encoding. For example, consider problem $\min |x|, x\in\\{\alpha\_i\\}$ with both positive and negative $\alpha\_i$. If $\alpha\_i$ is perfectly ordered $\alpha\_1<\alpha\_2<\cdots<\alpha\_n$ , and we branch on $y\_j$ with $\alpha\_j<0<\alpha\_{j+1}$ , then this branch can lead us to global optimal solution.

In practice, if you have a good understanding of you problem, you might be able to design an appropriate encoding to speed up branching; but generally binary encoding or incremental encoding will work. In case of SOS1/2 constraints, it is better to define them as "SOS1/2" if you are using CPLEX or Gurobi, since there might be some other black magic to speed up.

## Embedding

In above discussion, we try to use encoding to improve branching efficiency for certain constraints. Such idea can actually be extended for union of polytope $P=\cup\_i P\_i$. Consider encoding $H$: $h\_i\in\\{0,1\\}^m$ for each $P\_i$. By embedding, we can turn focus to object

$$P\_i'=P\_i\times\\{h\_i\\},$$

Notice that after embedding, $Q=conv(\cup\_i P\_i')$ is ideal ($h\_i$ characterizes the "extreme" part). So branching is not a major concern if there is no additional constraint. Then what else should we consider?

Remember that we still need to have a representation of $Q$, and its size (in terms of $H$-representation, the number of constraints) has great impact on LP efficiency. To understand the significance of encoding on size, we consider a simple example $P=\\{0\\}\cup [1,2]\cup [a,a+0.5]\cup [a+1,a+2.5]$ and binary encoding $H\subseteq \\{0,1\\}^2$. Now, if we do the embedding

$$P'=\\{0\\}\times \\{0,0\\} \cup [1,2]\times \\{0,1\\} \cup [a,a+0.5] \times \\{1,0\\} \cup [a+1,a+2.5] \times \\{1,1\\} ,$$

the convex hull $Q$ is a nicely shaped polyhedron. But in case

$$P'=\\{0\\}\times \\{0,0\\} \cup [1,2]\times \\{1,1\\} \cup [a,a+0.5] \times \\{1,0\\} \cup [a+1,a+2.5] \times \\{0,1\\} ,$$

the convex hull is more complex.

As dimension grows, such difference becomes more significant. If we select a random encoding to embed, it is possible to have exponential number of constraints. So, we want to use an encoding $H$ that $Q(H)$ has a small number of facet:

$$H=\arg\min\_H(size(Q(H)))$$

Generally, design of such embedding is ad-hoc and general construction is not viable (think of permutations of $\\{P\_i\\}$). But when $P\_i$ admits a natural ordering, we might be able to conclude some techniques in designing embedding. One interesting example is piecewise linear function on $R^1$ (which induces the SOS2 constraints) or $R^2$. It is shown that with proper meshing and ordering, an encoding method called gray code can achieve the logarithm size in $n$. Some related papers can be found in the reference.

In summary, the advantage of using embedding is that we don't need to create copies of original variables, which is similar to the non-extended formulations in last post. If the problem and $P\_i$ are well defined, embedding provide an efficient way to make those non-extend formulations ideal. For example, to model piecewise linear function on $R^1$, unary embedding is exactly the ideal extension of non-extended $V$-formulation, which is only sharp. But generally such idea is hard to implement.

## Reference

Vielma, J. P. (2015). Embedding Formulations and Complexity for Unions of Polyhedra. arXiv:1506.01417. Available at: http://arxiv.org/pdf/1506.01417v2.pdf

Yıldız, S., & Vielma, J. P. (2013). Incremental and encoding formulations for mixed integer programming. Operations Research Letters, 41(6), 654-658.

Vielma, J. P., & Nemhauser, G. L. (2011). Modeling disjunctive constraints with a logarithmic number of binary variables and constraints. Mathematical Programming, 128(1-2), 49-72.
