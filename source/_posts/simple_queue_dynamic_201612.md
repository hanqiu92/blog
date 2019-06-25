---
title: 'Reflection on Simple Queues'
categories: Reflection
tags:
- Queuing Theory
- Stochastic Process
mathjax: true
date: 2016-12-17 12:00:00
---
This semester, when I was working on homework of 1.200, there was a frequently asked question: under what situation would the queue system have a steady state? It is claimed that system with finite capacity always have a steady state, but no proof is given in the class. In this post, I will discuss the dynamical system approach for this problem, and suggest several extensions.

<!--more-->

## Modeling the Dynamics

Let us consider the easiest $M$/$M$/1/$k$ queue, where $k$ is a finite number. Following is a picture of the temporal transition (By Gareth Jones - [MM1_queue_state_space.png](https://commons.wikimedia.org/w/index.php?curid=17204982), CC BY-SA 3.0)

![two_segments](/blog/images/MM1_queue_state_space.png)

The state transition matrix is then given by

$$A^T = \begin{pmatrix} -\lambda & \lambda & 0 & \cdots & 0 & 0 \\\\ \mu & -(\mu+\lambda) & \lambda & \cdots & 0 & 0 \\\\ &&\vdots&&& \\\\ 0 & 0 & 0 & \cdots & \mu & -\mu \end{pmatrix} $$

It is taught from the class that to steady state probability $\pi$ can be obtained from solving equation $A\pi = 0$. But why is so?

Let us think about how to obtain the transition diagram. We consider a small time slice $dt$, and find the first order derivative of transition probabilities between states. That means, the dynamical system on the probability of states can be described by the following linear ODE:

$$ \frac{d}{dt}\pi = A\pi.$$

It is obvious that this system is well defined since operator $A$ projects one probability distribution to another probability distribution. And now, we can analyze the queuing system using the tools for more general differential dynamical systems.

The general solution of the ODE above is

$$\pi_t = e^{At} \pi_0.$$

Clearly, since $\pi\_t$ will always satisfy the regularity condition as a probability distribution, $e^{At}$ will not have any eigenvalue with positive real part greater than 1. We can also prove this in a more formal way. Notice that $M = I + A$ is a Markov matrix on the graph of states and it is strongly connected, we know that $M$ is [irreducible](https://en.wikipedia.org/wiki/Irreducibility_(mathematics). Then by Perron-Frobenius Theorem, we know that there is a maximal eigenvalue $\lambda\_{max}$ such that the corresponding eigenvector $\pi^\*$ is positive, and any non-negative eigenvector $\pi$ is a multiple of $\pi^\*$. But since $A^Te = 0$, we know the left eigenvector $e$ exists and $\lambda\_{max}$ must be 1. Then it is clear that the maximal eigenvalue of $e^{At}$ must be 1, the corresponding eigenvector $\pi^\*$ exists and must be stable, and $\pi\_t\to\pi^\*$.

## Extensions

It seems that the analysis above uses complicate approach to validate a simple statement. But under the same framework, we can deal with systems with varying arrival and service rate, which is common in practice. For example, let us consider a varying arrival rate $\lambda_t$ along with the transition matrix $A_t$ and the ODE

$$ \frac{d}{dt}\pi = A_t\pi.$$

Now, the general solution can be expressed as

$$\pi_t = e^{\int_0^t A_s ds} \pi_0 = e^{t\bar{A}_t} \pi_0,$$

where $\bar{A}\_t = \frac{1}{t}\int\_0^tA\_s ds$ is the time average of the transition matrix and is the transition matrix corresponding to average arrival rate $\bar{\lambda}\_t = \frac{1}{t}\int\_0^t\lambda\_s ds$. If $\lambda\_t$ is periodic with period $T$, by applying the same technique (informally) on $\bar{A}\_T$ we know that there is a stable trajectory $\pi\_T^\*$ on $S^k \times [0,T]$ and $\pi\_{nT+t}\to\pi_T^\*(\cdot,t)$ as $n\to\infty$.

To go one step further, we might be able to consider different arrival/service process. For example, consider a $M$/$D$/1/$k$ queue, in which the service process is deterministic. Now we have two matrices, the transient arrival

$$A^T = \begin{pmatrix} -\lambda & \lambda & 0 & \cdots & 0 & 0 \\\\ 0 & -\lambda & \lambda & \cdots & 0 & 0 \\\\ &&\vdots&&& \\\\ 0 & 0 & 0 & \cdots & -\lambda & \lambda \\\\ 0 & 0 & 0 & \cdots & 0 & 0 \end{pmatrix} $$

and the deterministic service with period $T=1/\mu$

$$B^T = \begin{pmatrix} 0 & 1 & 0 & \cdots & 0 & 0 \\\\ 0 & -1 & 1 & \cdots & 0 & 0 \\\\ &&\vdots&&& \\\\ 0 & 0 & 0 & \cdots & -1 & 1 \\\\ 0 & 0 & 0 & \cdots & 0 & -1 \end{pmatrix} $$

So within period $T$ the dynamics is

$$\pi_t = e^{tA} \pi_0,$$

and at time $T$ we have

$$\pi_T = Be^{TA} \pi_0.$$

Then, to analyze the long term behavior of this system, we only need to know the spectral property of matrix $F=Be^{TA}$.

We can also consider queues with infinity capacity. In this case, we need to define transition operator $A$ on $S^{\infty}$, the set of all discrete probability distribution, and analyze the spectrum of $A$. This is somewhat complicate so currently I will not discuss it. But one interesting question is, whether can we extend this notion into general probability distribution case and consider dynamics on the corresponding set/space.

## Summary

In this post I briefly review some simple queuing systems from the perspective of dynamical system. It is shown that this framework is powerful and can be generalized in many different settings. Moreover, we might be able to not only focus on steady state but also analyze the transient dynamics, which can be helpful for subsequent simulation design.
