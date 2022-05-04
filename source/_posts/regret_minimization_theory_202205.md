---
title: 'Regret Minimization Methods'
categories: Study Notes
tags:
- Reinforcement Learning
- Game Theory
mathjax: true
date: 2022-05-04 00:00:00
---

在算法博弈论中，一个核心的课题是如何对于给定的问题求解均衡。近期，我粗略地学习了[CMU 15-888](http://www.cs.cmu.edu/~sandholm/cs15-888F21/)这门课程，该课程的前一部分从理论的角度，回顾了均衡求解、遗憾值最小化、Blackwell Approachability等概念之间的联系，为理解相关方向的前沿进展提供了很好的基础框架。在这篇文章中，将对该课程中提到的主要结论进行简单整理，便于后续回顾。

<!--more-->

## 基本定义 Definitions

首先，在这一节中，我们将给出一些基本概念的定义。

### 均衡 Equilibriums

“均衡”这一概念对应的是博弈下的一类状态，在这些状态下，参与博弈的各方都没有主动改变策略的动机。根据不同的对“改变策略动机”的具体设定，我们可以给出以下的不同的均衡定义：

* 纳什均衡（NE）：这是最知名的均衡概念；在该均衡中，要求给定其他各方的均衡策略，每方的任意策略所带来的收益都不能超过均衡策略。
$$
u(s\_i,s\_{-i}) \leq u(s\_i',s\_{-i}), \ \forall i,s\_i'.
$$
* 相关均衡（CE）：在该均衡中，给定全局的均衡策略分布$\sigma$以及其他各方服从该均衡策略，每方的任意策略修正（modification）$f\_i$所带来的收益都不能超过均衡策略$\sigma$。该均衡概念相比于NE条件有所放松（均衡对各方的约束力更弱），但均衡存在的可能性更高，同时均衡更易于求解（通常只需要多项式时间复杂度）。
$$
\mathbb{E}\_{s \sim \sigma} u(s\_i,s\_{-i}) \leq \mathbb{E}\_{s \sim \sigma} u(f\_i(s\_i),s\_{-i}), \ \forall i,f\_i.
$$
* 粗糙相关均衡（CCE）：这一均衡概念相比CE，进一步降低了修正函数$f\_i$的自由度，因此求解难度得到进一步降低，但是得到的均衡对各方的约束力更弱。具体来说，
  * 对于正则形式（normal-form）博弈，修正函数$f\_i: s\_i \to s\_i'$取值需为常数，故均衡条件可改写为
$$
\mathbb{E}\_{s \sim \sigma} u(s\_i,s\_{-i}) \leq \mathbb{E}\_{s \sim \sigma} u(s\_i',s\_{-i}), \ \forall i,s\_i'.
$$
  * 对于扩展形式（extensive-form）博弈，修正函数$f\_i$在每个信息集$I$上的取值需为常数（注意$f\_i$可被理解为是定义在信息集集合$\mathcal{I}$和动作集合$\mathcal{A}$的笛卡尔积上的函数，将一组$(I,a)$转化为$(I,a')$，因此前述条件要求对于给定的$I$，对任意$a$，$(I,a)$都需要映射为同一个$(I,a')$）。

### 遗憾值与遗憾优化器 Regrets & Regret Minimizers

现在，让我们考虑一个（策略）学习问题：假定存在一个跟环境（其中可能包括其他主体）互动的主体（agent），其在每个时刻$t$产出动作$x^t \in \mathcal{X}$，然后观测到环境的反馈$l^t$；该主体会根据反馈结果更新自己的状态，从而调整后续时刻的动作。这里，假设反馈$l^t: \mathcal{X} \to \mathbb{R}$是一个效用函数，可用于离线评估动作$x^t$的效果。我们可以使用以下面向对象（object-oriented）的语言框架来抽象出这个主体的学习策略：该学习策略包括两个方法，一个是observe($l$)，即根据环境反馈$l: \mathcal{X} \to \mathbb{R}$更新状态；一个是act()，即根据自身状态生成下一期的策略$x\_t \in \mathcal{X}$。

在上述定义下，遗憾值（regret）这一概念，可以理解为“当主体与环境互动了相当长的时间（例如，$T$步）后，其实际效用收益与给定历史反馈$\\{l^t\\}\_t$下最优事后回顾（ex-post）效用收益的差距”。一种普适的遗憾值定义是Phi-regret：

$$
R^T\_{\Phi} = \max\_{\phi \in \Phi} \sum\_{t=1}^T [l^t(\phi(x^t)) - l^t(x^t)],
$$

其中$\Phi$是由一些从动作域$\mathcal{X}$到自身的映射所组成的集合。不同的映射集合$\Phi$对应不同的具体的遗憾值概念，有的更强，有的更弱；一些常见的遗憾值概念列举如下：

* 交换遗憾值（swap regret）：当$\Phi$由所有可行的$\mathcal{X} \to \mathcal{X}$的映射组成；
* 内部遗憾值（internal regret）：当$\Phi = \\{\phi\_{a\to b}\\}\_{a,b \in \mathcal{X}}$由所有的“单点扰动”映射组成，其中
$$
\phi\_{a\to b}(x) =
\begin{cases} 
x & \text{if } x != a; \\\\ 
b & \text{if } x = a. 
\end{cases}
$$
* 外部遗憾值（external regret）：当$\Phi = \\{\phi\_{x'}\\}\_{x' \in \mathcal{X}}$由所有的常数映射组成，其中$\phi\_{x'}(x) \equiv x'$。

最后，我们称一个学习策略$\mathcal{R}$为（某类）遗憾优化器，如果其能够保证随着该学习策略的持续运行，对应的遗憾值$R^T$相对于运行时间$T$以（渐近）次线性的速度增长，即$R\_T \sim o(T)$。

### 遗憾优化器与均衡求解器之间的关联 Connections among Regret Minimizers and Equilibrium Solvers

遗憾优化理论的重要性在于，遗憾优化器与均衡求解器是存在对应关系的。具体来说，

* 当一个博弈中的所有主体的学习策略使得交换遗憾值或内部遗憾值的增长速度为次线性时，其平均动作$\lim\_{T}\frac{1}{T}\sum\_{t=1}^T x\_t$将收敛到一个相关均衡上。
* 当一个博弈中的所有主体的学习策略使得外部遗憾值的增长速度为次线性时，其平均动作$\lim\_{T}\frac{1}{T}\sum\_{t=1}^T x\_t$将收敛到一个粗糙相关均衡上。

虽然根据上面的结果，从均衡求解的角度外部遗憾值上界不如交换遗憾值上界强。但是，下面的构造结果指出，在一定的附加条件下，我们可以使用外部遗憾优化器来构造更普适的$\Phi-$遗憾优化器。

* 假设映射集合$\Phi$中每个映射$\phi$都有至少一个不动点$x\in\mathcal{X}$，且已有一个定义在$\Phi$上的外部遗憾优化器$\mathcal{R}$。那么，一个$\Phi-$遗憾优化器$\mathcal{R}\_{\Phi}$可以构造如下：
  * $\mathcal{R}\_{\Phi}$.act的输出是$\mathcal{R}$.act的输出$\phi^t$的不动点$x^t$；
  * 给定$\mathcal{R}\_{\Phi}$.observe的输入$l^t$，我们将复合函数$L : \phi \to l^t(\phi(x^t))$输入到$\mathcal{R}$.observe中。

事实上，上述构造方法也正是目前前沿的相关均衡求解算法所使用的基础工具之一。

## 遗憾优化器的设计 Design of Regret Minimizers

在接下来这一节中，我们将讨论遗憾优化器的设计方法。

### 在线学习 Online Learning

首先，我们需要注意到，前述的策略学习场景可以被纳入到在线学习框架中，因此我们可以直接将常用的在线学习方法应用于遗憾优化器的设计中。特别地，Follow-the-regularized-leader (FTRL)和online mirror descent (OMD)是两种兼具理论性质和实践价值的在线学习方法；已有的理论工作显示，这两种方法的遗憾值上界处于$O(T^{1/2})$级别，因此按照定义，可以直接作为遗憾优化器使用。

### Blackwell Approachability & Regret Matching

接下来，我们将引入一组新的概念：Blackwell Approachability (BA)博弈和BA收敛条件。BA博弈包含两个参与者，己方和对手方；己方的目标是无论对手采取何种动作，己方的平均效用最终能收敛到一个目标范围内。具体来说，一个BA博弈可以由以下符号表示$\Gamma = (\mathcal{X},\mathcal{Y},u,S)$，其中：

* 策略空间$\mathcal{X},\mathcal{Y}$是闭凸集；
* 效用函数$u:\mathcal{X}\times\mathcal{Y}\to \mathbb{R}^n$是一个双线性函数；
* $S \subset \mathbb{R}^n$是目标效用值集合。

在该博弈中，每一步己方需要先选择动作$x^t \in \mathcal{X}$，然后对手选择动作$y^t \in \mathcal{Y}$，最后己方得到效用$u^t=u(x^t,y^t)$。己方的目标是使得平均效用$\bar{u}=\frac{1}{T}\sum\_{t=1}^T u(x^t,y^t) \to S$渐近收敛到目标集合，无论对手如何选取其动作$\\{y^t\\}$。该目标即为BA收敛条件。

Blackwell证明了，如果任意包含$S$的半平面$\mathcal{H}$都是可实现的（forceable，定义为：存在$x^* \in \mathcal{X}$，使得对任意$y\in \mathcal{Y}$，$u(x^*,y) \in \mathcal{H}$），那么存在策略来达成BA收敛条件。一种具体的策略构造方式如下：在第$T$步，

1. 计算此前各步的平均效用$\bar{u}^T = \frac{1}{T-1} \sum\_{t=1}^{T-1} u(x^t,y^t)$；
2. 计算$\bar{u}^T$向集合$S$的投影$v$（即使得距离$d(v,\bar{u})$最小的在集合$S$内的元素）；
3. 如果$\bar{u}^T \in S$，那么选择任意$x^T$即可；否则，选择与半平面$\mathcal{H}=\\{z\in \mathbb{R}^n| (\bar{u}^T - v)^T(z - v) \leq 0\\}$对应的动作$x^T$；
4. 观察对手的动作$y^T$和自身的效用$u(x^T,y^T)$，并进入下一步。

之所以要讨论BA博弈，是因为BA博弈和遗憾优化器的设计之间有非常紧密的联系。一方面，某些特定的遗憾最小化问题可以表示为BA博弈的形式，因此可以通过前面的策略构造方式来得到对应的遗憾优化器。特别地，对于单纯形$\Delta^n$上的（外部）遗憾值优化问题$R^T = \max\_{x \in \Delta^n} \sum\_{t=1}^T (l^t)^T (x - x^t)$，我们可以考虑以下形式的BA博弈$\Gamma = (\Delta^n,\mathbb{R}^n,u,\mathbb{R}^n\_{\leq 0}), \ u(x^t,l^t)=l^t - [(l^t)^T x^t] \bf{1}$；分析表明，平均遗憾值$R^T/T$不会超过该BA博弈的渐近平均效用与目标集合的距离$d(\bar{u},\mathbb{R}^n\_{\leq 0})$，因此按前面构造方法生成的学习策略可以当做单纯形上的遗憾优化器。进一步地，通过一些代数变换可知，此时的学习策略即为regret matching(RM)：其中，observe函数将更新累计遗憾值$r^t = r^{t-1} + l^t - [(l^t)^T x^t] \bf{1}$，而act函数将给出动作$x^t = \frac{[r^t]^+}{||[r^t]^+||\_1}$.

另一方面，我们又可以使用遗憾优化器来构造BA博弈的学习策略。这样做的好处是，不同的遗憾优化器可以生成不同的学习策略，进而又可以在特定的遗憾优化问题上产生不同的算法。这种构造思路基本沿用了前面介绍的构造法的思路，只是在选取切平面$\mathcal{H}$时不局限在投影点$v$上。具体来说，假设目标集合$S$是个锥（cone）（如果不是，考虑一个包含$S$的锥$S'$即可），那么根据锥的性质，对于对偶锥$S^*$中的任一点$\theta$，半平面$\mathcal{H}\_{\theta} = \\{x \in \mathbb{R}^n| \theta^T x \leq 0 \\}$都包含$S$。因此，可以证明，如果我们使用一个$S^* \cap \\mathbb{B}^n\_2$上的遗憾优化器，来根据效用值$u^t$选取每步的$\theta^t$以及半平面$\mathcal{H}\_{\theta^t}$，那么相应构造出来的学习策略也是能够满足BA收敛条件的。特别地，如果我们使用FTRL作为基础的遗憾优化器，那么对应的学习策略应用在单纯形上将对应RM方法；而如果使用OMD作为基础的遗憾优化器，那么对应的学习策略应用在单纯形上将对应RM+方法。这样，我们就证明了RM方法和RM+方法的有效性。

### 遗憾回路 Regret Circuits

前面两小节介绍的遗憾优化器构造方法虽然是相对通用的，但是当目标策略空间$\mathcal{X}$的复杂度较高时，实践上计算会很复杂。此时，遗憾回路（regret circuit）这种工具提供了一种解决方案：当$\mathcal{X}$具备一定的分解结构时，我们可以先使用前两小节介绍的方法构造出各子结构上的遗憾优化器，然后通过一些标准的合成步骤（组件）生成父结构上的遗憾优化器。特别地，对于一般的扩展形式（extensive-form）博弈，我们可以基于各信息集上的正则形式（normal-form）子博弈所对应的遗憾优化器，通过遗憾回路工具构造出整体博弈的遗憾优化器。我们以往介绍过的[CFR算法](https://hanqiu92.github.io/blogs/2021/CFR_202103/)也可以看作是遗憾回路应用的一个例子。

下面，我们介绍两种遗憾回路的合成组件。我们首先考虑笛卡尔积$\mathcal{X} = \mathcal{X}\_1 \times \mathcal{X}\_2$这种形式。假设分别给定$\mathcal{X}\_1$和$\mathcal{X}\_2$上的遗憾优化器$\mathcal{R}\_{\mathcal{X}\_1}$,$\mathcal{R}\_{\mathcal{X}\_2}$；那么$\mathcal{X}$上的遗憾优化器$\mathcal{R}\_{\mathcal{X}}$可以构造如下：

* $\mathcal{R}\_{\mathcal{X}}$.act的输出$x^t$是$\mathcal{R}\_{\mathcal{X}\_1}$.act的输出$x\_1^t$和$\mathcal{R}\_{\mathcal{X}\_2}$.act的输出$x\_2^t$的笛卡尔积$(x\_1^t,x\_2^t)$；
* 给定$\mathcal{R}\_{\mathcal{X}}$.observe的输入$l^t$，我们将其按维度拆分成$l\_1^t$和$l\_2^t$，并分别输入到$\mathcal{R}\_{\mathcal{X}\_1}$.observe和$\mathcal{R}\_{\mathcal{X}\_2}$.observe中。

可以证明，这样构造的遗憾优化器$\mathcal{R}\_{\mathcal{X}}$的遗憾值满足$R\_{\mathcal{X}}^T = R\_{\mathcal{X}\_1}^T + R\_{\mathcal{X}\_2}^T$。

接下来，我们考虑凸包$\mathcal{X} = \text{co}\\{ \mathcal{X}\_1, \mathcal{X}\_2 \\}$这种形式。假设分别给定$\mathcal{X}\_1$和$\mathcal{X}\_2$上的遗憾优化器$\mathcal{R}\_{\mathcal{X}\_1}$,$\mathcal{R}\_{\mathcal{X}\_2}$，以及二维单纯形$\Delta^2$上的遗憾优化器$\mathcal{R}\_{\Delta^2}$；那么$\mathcal{X}$上的遗憾优化器$\mathcal{R}\_{\mathcal{X}}$可以构造如下：

* $\mathcal{R}\_{\mathcal{X}}$.act的输出$x^t$是$\mathcal{R}\_{\mathcal{X}\_1}$.act的输出$x\_1^t$、$\mathcal{R}\_{\mathcal{X}\_2}$.act的输出$x\_2^t$、和$\mathcal{R}\_{\Delta^2}$.act的输出$(p\_1^t,p\_2^t)$的线性组合
$$x^t = p\_1^t x\_1^t + p\_2^t x\_2^t;$$
* 给定$\mathcal{R}\_{\mathcal{X}}$.observe的输入$l^t$，一方面我们直接将其输入到$\mathcal{R}\_{\mathcal{X}\_1}$.observe和$\mathcal{R}\_{\mathcal{X}\_2}$.observe中，另一方面计算其与$x\_1^t$和$x\_2^t$的内积$((l^t)^T x\_1^t,(l^t)^T x\_2^t)$后，输入到$\mathcal{R}\_{\Delta^2}$.observe中。

可以证明，这样构造的遗憾优化器$\mathcal{R}\_{\mathcal{X}}$的遗憾值满足$R\_{\mathcal{X}}^T \leq R\_{\Delta^2}^T + \max\\{ R\_{\mathcal{X}\_1}^T,R\_{\mathcal{X}\_2}^T \\}$。

## 结合预测器的求解算法 Predictor-based Algorithms

在前一节中，我们介绍了数种遗憾优化器设计思路。在这一节中，我们将讨论，如果我们能够对环境及其反馈有一定的预测能力，那么能否进一步优化学习策略的效率？

为了便于理解和讨论“对环境反馈有预测能力”的具体含义，下面我们将对前述的策略学习问题进行一定的简化：第$t$步的反馈函数$l^t: x^t \to (l^t)^T x^t$简化为为向量$l^t$与策略$x^t$的内积。在该特定场景下，预测能力即指对反馈向量$l^t$的预测；在下面的讨论中，我们使用符号$m^t$来代表对应的预测值。另外，我们还将仅考虑外部遗憾值，即
$$
R^T = \max\_{x \in \mathcal{X}} \sum\_{t=1}^T (l^t)^T(x - x^t).
$$

当考虑了对环境反馈的预测值$m^t$后，学习策略中的act函数可以扩展为act($m^t$)，即根据对下一阶段的环境反馈预测，来选取下一阶段的策略；而observe函数则保持不变。特别地，当我们不具备预测能力时，可以简单地令$m^t = 0$来回退到基础版本的学习策略中。

同时，学习策略的遗憾值上界评估标准也需要进行一定的调整：由于预测值$m^t$的准确性外生于学习策略本身的效率属性，遗憾值上界有必要同时包含策略效率属性和预测器准确度两部分信息。效用变差遗憾值上界（Regret bounded by Variation in Utilities, RVU）是一种常用的满足上述需求的评估标准，该上界由三个参数$(\alpha,\beta,\gamma)$确定：我们称一个学习策略满足RVU$(\alpha,\beta,\gamma)$上界，如果其遗憾值满足
$$
R^T \leq \alpha + \beta \sum\_{t=1}^T || l^t - m^t ||\_{\*}^2 - \gamma \sum\_{t=2}^T || x^t - x^{t-1} ||^2.
$$

现有的研究结果表明，一些常见的遗憾优化器在引入预测值后对应的RVU上界分别为：(O)FTRL $\sim (\Omega/\eta,\eta,1/4\eta)$、(O)MD $\sim (\Omega/\eta,\eta,1/8\eta)$，其中$\eta$是步长参数、$\Omega$是正则函数在定义域$\mathcal{X}$的变化幅度上界。如果令步长$\eta \sim T^{-1/2}$，那么可知(O)FTRL和(O)MD的遗憾值上界都是$O(T^{1/2})$，再次说明了这两种方法具有良好的学习效率。

进一步地，我们可以通过BA博弈将上述算法的RVU上界应用到RM和RM+上，分别得到以下遗憾值上界：

* 不含预测值：$R^T \leq \sqrt{2\sum\_{t=1}^T || l^t - [(l^t)^T x^t] \bf{1} ||\_2^2}$；
* 结合预测值：$R^T \leq \sqrt{2\sum\_{t=1}^T || (l^t - m^t) - [(l^t - m^t)^T x^t] \bf{1} ||\_2^2}$；

这组结果说明了，当预测结果$m^t$是相对准确的时候，引入预测值确实可以提高策略效果（即达到更低的遗憾值上界）。
