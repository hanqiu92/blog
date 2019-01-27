---
title:       'A Note on Decision Behavior Modeling'
categories:  Reflection
tags:
- Behavior Modeling
- Reinforcement Learning
mathjax:     true
date:        2019-01-27 00:00:00
---

在这篇短文中，我将基于过去一年中的研究经验，简单地分享一下我目前对行为决策建模的见解。

In this post, I will briefly discuss some of my current thoughts on decision behavior modeling based on my research experience over the past year.

抽象地来看，决策行为即在给定决策信息$s$（下称状态）的情况下，通过某种策略$\pi$采取动作$a$

From an abstract point of view, decision making is to take an action $a$ according to a certain strategy $\pi$ given decision information $s$ (hereinafter referred to as the state)

$$a \sim \pi(s).$$

而行为决策建模，则希望根据观察数据$\mathcal{D} = \\{(s,a)\\}_n$，获取决策机制$\pi$的相关信息。

While decision behavior modeling attempts to infer the strategy $\pi$ from observations $\mathcal{D} = \\{(s,a)\\}_n$.

由于数据集$\mathcal{D}$是有限的，能够很好地拟合该数据集的$\pi$可能有无穷多个。因此，研究者通常需要对$\pi$的形式做出额外假设。而一个最基础且核心的假设是$\pi$服从**效用最大化原理**。

Because the dataset $\mathcal{D}$ has finite datapoints, there can be an infinite number of $\pi$ that fits well into the dataset. Therefore, researchers often need to make additional assumptions about the form of $\pi$. And one of the most fundamental assumptions is that $\pi$ follows the **utility-maximization principle**.

<!--more-->

## 效用最大化原理 The Utility-maximization Principle

效用最大化原理指出，决策者在面对决策问题时所表现出的行为应与最大化自身效用的行为相一致。通过该原理，行为决策建模问题可以被转换为效用函数估计问题。相比于直接使用统计或机器学习模型等描述性手段对策略$\pi$进行拟合，这种**规范性**（normative）建模方式考虑了我们对行为规律的先验认知（即理性决策行为），因此更适用于提供解释和**反事实**（counterfactual）判断。同时，由于该原理对行为模式有较强的正则化约束效果，基于此得到的行为模型具有较强的泛化和迁移能力（即不同场景下的一致性）。

The utility-maximization principle (UM) states that the behavior of a decision maker (DM) should be consistent with the behavior that maximizes her utility. Through this principle, decision behavior modeling problems can be transformed into utility function estimation problems. Compared to the direct use of descriptive methods, such as statistical or machine-learning models, to fit the strategy $\pi$, this **normative** approach takes into account our prior understanding of behavior (i.e., rationality) and is therefore more suitable for providing explanations and **counterfactual** judgments. At the same time, because of the regularization of the UM principle, the resulted behavior model has strong generalization and transfer ability (i.e., consistency in different scenarios).

然而，效用最大化原理隐含了以下假设：决策者具有“高超”的优化问题求解能力，无论决策问题有多复杂都能准确给出最优解。由于该假设并不完全符合直觉，效用最大化原理在过去的数十年中受到了很多质疑：大量学者通过进行实验或是分析实际经济数据，指出人们通常会做出次优决策；另外在理论方面还有许多学者提出了符合有限理性假设的行为模型，并通过实际数据比较说明这些模型的解释能力好于基于效用最大化原理的模型。

However, the UM principle implies the following assumption: the DM has an "excellent" optimization capability to provide the optimal solution accurately regardless of the complexity of the underlying decision problem. Because this assumption is not intuitively solid, many doubts have been cast on the UM principle in the past few decades: a large number of scholars, by conducting labotory experiments or analyzing real-world economic data, have pointed out that people usually make sub-optimal decisions; in addition, many scholars have proposed behavioral models within the bounded rationality framework and illustrate with real-world data that these models have better explanation power than the model from the UM principle.

遗憾的是，这些新的理论并不能够彻底取代效用最大化原理：它们从某些特定角度改进了原始效用最大化原理的缺陷，但是仍处于“决策优化”的框架内。事实上，一方面，在**最优性**以外我们并没有很多合乎直觉的从决策问题到策略机制的映射关系以供选择；另一方面，对任何决策行为，我们总能通过补充缺失信息的方式令其达到最优性（当然，这可能会损失某些不同行为之间的一致性；关于这方面的讨论属于(公理化)决策论的范畴，有兴趣的读者可以从Kreps的Notes on the Theory of Choice[1]读起）。因此，在我看来，与其通过“创造”有限理性理论来修正效用最大化原理，倒不如通过**修正决策问题的设定**来拓展效用最大化原理的应用面。更具体地，我们应该关注：决策者是在何种场景下、基于何种信息、对何种效用目标进行判断，从而作出相应行为？或者说，对于决策行为

Unfortunately, these new theories cannot completely overthrow the UM principle: they improve it from some specific perspectives, but are still within the "decision optimization" framework. In fact, on the one hand, besides **optimality**, we do not have much choice on the mapping from decision-making problems to strategies. On the other hand, for any decision-making behavior, we can always make it optimal in some decision-making problems by supplementing some missing information (of course, this may result in a loss of consistency among behaviors; discussions on this aspect fall into the field of axiomatic decision theory, and interested readers can refer to Kreps' Notes on the Theory of Choice[1] for more details). Therefore, in my opinion, instead of aiming at "creating" new bounded rationality models to "correct" the UM principle, it is better to expand the applicability of the UM principle by **reformulating the underlying decision problems**. More specifically, we should consider the following question: under what circumstances and based on what information and utility function did the DM take the observed action? In other words, for action

$$a \sim \pi(s),$$

研究者是否考虑了正确的决策信息$s$？

does the researcher consider the correct decision information $s$?

## 有限理性与不完全信息 Bounded Rationality and Incomplete Information

前面的讨论中提到，大量对效用最大化原理进行修正的工作都是关于决策者的有限理性的。虽然“有限理性”这个词常被用来支持“决策者缺乏最优决策能力”这一论断，但在我看来，有限理性可能恰好反映了决策者在求解决策问题时的理性行为。正如优化工作者通常需要权衡求解精度和计算开销，对于复杂的决策问题决策者如果要寻找效用最大化的行为，可能需要付出很高的时间代价（或者说，未来效用的损失）。在这种情况下，使用启发式方法（heuristics）去优化行为通常会带来更好的整体收益。当然，具体有哪些可以考虑的启发式方法、每种方法又有多高的时间成本因问题而异。

As mentioned in previous discussions, many works on "correcting" the UM principle is about the bounded rationality of the DM. Although the term "bounded rationality" is often used to support the argument that "the DM lacks optimization capability", in my opinion, it may actually reflect the rational behavior of the DM in solving decision problems. Just as practitioners in optimization often need to balance between the solution accuracy and the computational overhead, for complex decision-making problems, the DM will pay a high cost of time (or loss of future utility) if she looks for utility-maximizing behavior. In this case, using heuristics to optimize behavior usually leads to better overall utilities (of course, the set of feasible heuristics to be considered and the corresponding costs of time varies from problem to problem).


从另一个角度上看，求解决策问题的复杂度实际上反映了决策者对于最优行为所拥有的信息量：问题越难求解，意味着决策者对最优行为的信息越少、搜寻代价越高。这种思路可以进一步拓展到一般的不完全信息场景：事实上，在很多情况下，决策问题的相关信息较多，决策者在一开始可能只拥有一小部分信息。此时，在作出最终决策之前，决策者通常需要决定是否以及如何获取更多信息。例如，我们可以考虑一个经典的汽车购置决策问题。对于一个消费者来说，他不太可能了解市面上的所有在售车型；但是他可以通过上网搜索、参加车展、试驾等一系列方式获取更多信息。然而，不同的信息获取方式各有优劣：比如说，进行试驾能够获得对一款车型非常深入的了解，但是花费的时间成本较大；而阅读汽车杂志、观看广告等行为只需要少量时间即可获得多种车型的基本信息。因此，这个汽车购置决策问题还包含一个汽车信息获取的决策子问题。在过去十年中，已经有大量的理论和实证工作对这种信息处理机制进行了深入的讨论。有兴趣的读者可以参考Sims[2]，Gabaix[3]，以及Masatlioglu and Nakajima[4]。

From another perspective, the complexity of the solution to the decision problem actually reflects the amount of information that the DM has for optimal behavior: the more difficult the problem is to solve, the less information the DM has for the optimal behavior and the higher cost of information collecting. This view can be further extended to general scenarios with incomplete information: in fact, in many cases, the decision-making problem has much relevant information, but the DM may only have a small fraction of it at the beginning. In this case, the DM often needs to decide whether and how to obtain more information before making the final decision. For example, let us consider the problem of buying a car. For a customer, she is unlikely to be familiar with all available models on the market; but she can obtain more information through searching methods such as online search, auto shows, and test drive. However, these methods have their own advantages and disadvantages: for example, through test drive, the DM can obtain a very deep understanding of a model, but it costs a lot of time; while by reading car magazines and watching ads, the DM can obtain basic (and only basic!) information on a wide range of models in a short period of time. Therefore, this car purchase problem also includes a car information acquisition subproblem. In the past decade, many theoretical and empirical studies have discussed this information acquisition mechanism in depth. Interested readers can refer to Sims [2], Gabaix [3], and Masatlioglu and Nakajima [4].

在上述讨论中，我都只考虑了决策问题的“当前”信息。但是，决策问题通常（在时间维度上）都不是孤立的，决策者的决策行为很有可能依赖于其**对历史信息的归纳**和**对未来的期望**。在下面两节中，我将就这两方面因素进行更详细的讨论。

In the above discussion, I have only considered the "immediate" information of the decision problem. However, decision-making problems are usually not isolated/independent in the time dimension, and the behavior of the DM is likely to depend on the DM's **summaries of past information** and **expectations for the future**. In the next two sections, I will discuss these two aspects in more detail.

## 决策序列与动态学习过程 Decision Sequences and Learning Dynamics

在日常生活中，许多问题需要进行多期决策。例如，我们可以考虑一个经典的多期消费决策问题：给定当期生产产出与前期投资的关系，决策者需要决定如何对每期的产出分配消费和投资比例以最大化（从消费中获得的）总效用。相比单阶段决策行为，多阶段决策行为相对更难分析，因为在给定效用函数的情况下，对最优决策行为的求解仍然很复杂（注：在这种场景下要找到**时间维度上一致的**有限理性/次优决策解释也是很困难的！）。不过，在过去的数十年中，计量经济学家和计算机科学家仍然对此类问题（两派学者分别称之为动态离散选择和逆向强化学习问题）进行了非常深入的研究，提出了包括嵌套不动点算法（Rust[5]）在内的一系列效用估计和行为建模方案。关于动态离散选择方面的工作，有兴趣的读者可以参考Aguirregabiria and Mira[6]和Heckman and Navarro[7]。对于逆向强化学习方面的进展，有兴趣的读者可以参考我[最近一篇短文](https://hanqiu92.github.io/2018/09/01/gan_irl_methods/)。

In our daily life, there are many problems that require multi-period decisions. For example, in a multi-period consumption problem, the DM needs to allocate her credit (from production) to consumption and investment in each period to maximize her total utility from consumption, given the relationship between the production output in a period and investments from previous periods. Multi-period decision-making behavior is relatively more difficult to analyze than single-period decision-making behavior, because even when the (single-period) utility function is explicitly given the optimal strategy can still be very complicated (note: in this case, finding a ** time consistent ** bounded rationality/suboptimality explanation can also be very difficult!). Nevertheless, in the past few decades, econometricians and computer scientists have conducted in-depth research on such issues (the two groups called these problems "dynamic discrete choice" and "inverse reinforcement learning", respectively) and suggested a series of utility estimation and behavioral modeling methods, including the nested fixed-point algorithm (Rust [5]). For work on the dynamic discrete choice problem, interested readers can refer to Aguirregabiria and Mira [6] and Heckman and Navarro [7]. For recent progress in inverse reinforcement learning, interested readers can refer to [my recent post](https://hanqiu92.github.io/2018/09/01/gan_irl_methods/).

然而，这些工作都是建立在“决策者对于相应的多阶段决策问题有完全信息”这一假设上的。正如上一节所讨论的，决策者很有可能缺乏关于决策问题和最优策略的信息。这意味着我们可以考虑从以下两个角度对已有的工作进行拓展：决策者对演化机制/环境反馈的不确定性，以及决策者不完美的远期规划能力。下面我将指出，这两点都可以通过**学习过程**来进行描述。

However, these works had all assumed that "the DM has complete information about the underlying multi-period decision problem". As discussed in the previous section, the DM is likely to lack information about the decision problem and the optimal strategy. This means that we can consider extending the existing work from two perspectives: the DM's uncertainty of the state transition mechanism/environmental feedback, and the DM's imperfect (long-term) planning capabilities. As I will illustrate below, both of these can be described by the **learning dynamics**.

一方面，最优长期决策的求解计算开销大，决策者通常不能在合适的时间内给出决策问题的最优解，因此依照上述讨论的方式，我们可以认为次优行为是对计算开销和求解精度的权衡结果。但是对于序列决策过程，不同时间点发生的决策问题具有相似的解结构，因此在求解当前决策问题时决策者可以复用此前的计算结果，而不需要从零开始。又注意到最优长期决策的求解通常涉及迭代过程（例如，对于经典的无限期的动态规划问题，我们可以使用值函数迭代或策略迭代的方式进行求解），我们可以认为决策者在求解当前决策问题时主要是在利用此前迭代中获得的关于最优解的信息。这意味着序列决策中的次优行为可以被看作是优化迭代过程中的副产物。

On the one hand, to solve for the optimal strategy has a large computational cost, so the DM usually cannot find the optimal action in a reasonable time. Therefore, according to the above discussion, we can view the suboptimal behavior as a result of the tradeoff between computational costs and accuracies. However, for the sequential decision process, the decision problems in different periods have a similar structure, so when she tries to solve a decision problem in the future the DM can reuse her previous calculation results rather than start from scratch. Also notice that algorithms for the optimal long-term strategy usually involve iterative processes (for example, we can solve the classical infinite-time dynamic programming problems by value function iterations or policy iterations), we can think that in solving the current decision problem the DM mainly relies on her information about the optimal solution obtained from previous iterations. This means that suboptimal behavior in sequential decisions can be seen as a by-product of the iterative process.

另一方面，对于演化机制的不确定性，我们应注意到强化学习问题并没有假设决策者对演化机制有完全信息，而仅假设演化机制在学习过程中保持不变；决策者是在学习过程中通过不断的尝试和反馈来更新对演化机制的认知和对策略进行迭代。这意味着在给定未来演化机制不变的预期下，在每个具体时间点上决策者对演化机制的信息都可以通过历史经验来描述。（注：如果我们同样采用**过程**而非**结果**的角度来审视“未来演化机制不变”这一预期，我们可以使用关于这一预期的历史信息和某种一致性/不变性假设来描述在每个具体时间点上决策者对于这一预期成立的预期；这一递推过程可以无限进行下去，直到得到最终不变的不变性假设/预期。个人感觉可以用集合/测度论语言对其进行描述，或者考虑多层循环神经网络（RNN）进行建模。）

On the other hand, for the uncertainty of the state transition, we should notice that in reinforcement learning problems we generally do not assume the DM has complete information about the state transition function, but only assume that the state transition remains unchanged during the learning process. In the learning process, the DM updates her perception of the state transition mechanism and her strategies through continuous trials and errors. This means that given the expectation of an invariant state transition function in the future, the DM's knowledge of the state transition in each period can all be described by her experience in previous periods. (Note: Regarding the "invariant state transition" expectation, if we think of the **process** rather than the **result**, we can also use some invariant assumptions and the DM's previous information on this expectation to describe her current knowledge of this expectation. This recursive process can go on until we obtain the final invariant assumption/expectation. I think that this can be described by the language of set/measure theory, or modeled by a multi-layer recurrent neural network (RNN).)

综合上面两点，我认为序列决策中决策者的行为更有可能来自于一个动态学习（优化）过程，而非优化结果。在该视角下，决策者的行为不仅依赖于其在真实环境中的状态，还受其行为历史的影响。在下一节中，我将基于这些理解进一步讨论决策者的历史信息处理机制。

Combining these two points, I think that the DM's behavior in a sequential decision process is more likely to come from a dynamic learning (optimization) process than an optimized result. From this perspective, the behavior of DM depends not only on her state in the actual environment but also on her history. In the next section, I will discuss the DM's processing mechanisms of past information in more details.

## 历史信息处理与策略迭代 Processing of Past Information and Policy Iterations

在上面的讨论中我提到，决策者在作出决策行为时可能会考虑历史信息（下记$s^h$）。一种简单的在决策行为中引入历史信息$s^h$的方法是将其作为策略的输入变量

In the above discussion, I mentioned that when making a decision the DM may consider historical information (hereinafter $s^h$). A simple way to introduce $s^h$ into the formulation of the decision-making problem is to include it as an input to the strategy

$$a \sim \pi(s,s^h).$$

然而，上式隐含了“策略$\pi$与历史信息$s^h$相独立”这一假设，而这一假设在很多现实场景、特别是在上面提及的动态学习过程中并不成立：策略本身也应随着历史信息的更新而在不断迭代。要解决这一问题，我们可以从更抽象的视角上考虑迭代机制$F$，并将策略$\pi$也作为迭代过程的一部分

However, the above formulation implies an assumption that "strategy $\pi$ is independent of historical information $s^h$", even though this assumption is not valid in many real-world scenarios, including the aforementioned dynamic learning process: as historical information accumulates, the strategy itself should also be updated. To resolve this problem, we can consider an iterative mechanism $F$ which including the strategy $\pi$ in the iterative process

$$ a\_t,(\pi,s^h)\_t \sim F(s\_t,(\pi,s^h)\_{t-1}), $$

其中$t\in N^+$代表时间点。（注意到上式与循环神经网络具有相似的结构：$s_t$可以看做时间$t$时系统新增的输入，$a_t$为输出，而$(\pi,s^h)$则是不断更新的隐状态变量。）

where $t\in N^+$ represents the time period. (Note that the above formulation has a similar structure to the recurrent neural network: $s_t$ can be seen as the input to the system at time $t$, $a_t$ the output, and $(\pi,s^h)$ the state variables.)

现在，一个新的问题是要确定$F$的形式：从规范性的角度出发，$F$应代表某种优化机制，但要找到一种合适的表达形式并不容易；而如果使用表达能力强的模型（例如LSTM）表示$F$，则估计结果很可能缺乏解释性。下面，我将尝试从现有的在线学习和强化学习算法中提取一些**基本建模元素**。

Now, a new question is to determine the form of $F$: from a normative point of view, $F$ should represent an optimization mechanism, but it is nontrivial to find a suitable form; and if we consider models with high capacities (such as LSTM) for $F$, the estimation can be short of explanation power. Next, I will briefly review some of the existing online learning and reinforcement learning algorithms and try to distill some **basic elements for modeling**.

我们首先关注策略$\pi_t$的形式（假设决策行为仍然服从$a_t \sim \pi_t(s_t)$）。从现有的强化学习方向看，如果学习过程是基于策略的，那么策略$\pi_t$本身可以被参数化；如果学习过程是基于值函数（效用）$Q_t$的，那么策略可以表示为一个最大化运算与参数化值函数的复合$\pi\_t = \arg\max\_{a\in A}Q\_t(\cdot,a)$。在两种情况下，策略都可以表示成**参数化形式**$\pi_t = \pi(\cdot,\theta_t)$。

We first look at the form of the strategy $\pi_t$ (assuming that the decision behavior still follows $a_t \sim \pi_t(s_t)$). According to the existing reinforcement learning algorithms, if the learning process is policy-based, the strategy $\pi_t$ is parameterized; if the learning process is value-based, the strategy can be expressed as the compound of the maximum operator and a parameterized value function $Q_t$: $\pi\_t = \arg\max\_{a\in A}Q\_t(\cdot,a)$. In both cases, the strategy can be represented by a **parameterized form** $\pi_t = \pi(\cdot,\theta_t)$.

接下来我们考虑隐状态$s^h$中的元素，以及相应的迭代机制：

Next we consider the elements in the hidden state $s^h$ and the corresponding iterative mechanism:

1) 对于经典的（在线）随机梯度下降算法，状态$s_t$可以表示为特征和目标的组合$(x_t,y_t)$，动作$a_t$则为模型参数$\theta_t$（策略$\pi_t$是平凡的相等函数）。$\theta\_t$可以表示为前一阶段参数$\theta\_{t-1}$和当前梯度$g\_t=g(\theta\_{t-1},s\_t|f)$的加权和

1) For the classical (online) stochastic gradient descent algorithm, the state $s_t$ can be expressed as a combination of features and the label $(x_t, y_t)$, and the action $a_t$ is the model parameter $\theta_t$ (strategy $\pi_t$ is an trivial equality function). $\theta\_t$ can be expressed as the weighted sum of the parameter $\theta\_{t-1}$ in the previous stage and the current gradient $g\_t=g(\theta\_{t-1}, s\_t|f)$

$$\theta\_t = \theta\_{t-1} + \alpha g(\theta\_{t-1},s_t|f),$$

其中$f$是目标模型。注意在这种情况下我们不需要考虑$s^h$。

where $f$ is the target model. Note that in this case we don't need to consider $s^h$.

2) 对于无模型强化学习，一种常用的稳定训练过程的方法是**经验回放**：梯度$g\_t$是从历史样本中采样计算所得，而不是依赖当前样本。在该情况中，历史状态$s^h$即为经验池；而在迭代过程$F$中我们还需要考虑从经验池中进行采样的方法$f\_\{sample\}$（均匀采样、优先级采样等）。

2) For model-free reinforcement learning algorithms, a commonly used method to stabilize the training process is **experience replay**: The gradient $g\_t$ is calculated from the historical sample rather than the current one. In this case, the historical state $s^h$ is the experience pool; and in formulating the iteration process $F$ we also need to consider the sampling method $f\_\{sample\}$ (e.g., uniform sampling, prioritized sampling).

3) 对于基于模型的强化学习，历史状态$s^h$为模型$M\_t$；策略参数$\theta\_t$的训练迭代是通过对模型$M\_t$进行仿真采样实现的。

3) For model-based reinforcement learning algorithms, the historical state $s^h$ is the model $M\_t$; the update of the strategy parameter $\theta\_t$ follows the gradients computed by simulating the model $M\_t$.

综合以上几点，我们可以暂且认为隐状态$s^h$主要包含由过去决策行为组成的**经验池**和通过过去决策行为训练得到的**模型**。这些信息实际上刻画了决策者对潜在演化机制/环境反馈的了解，同时又基本决定了策略的迭代方式。由于$s^h$也包括了决策者对演化机制的不确定性，我们还可以使用这套框架来描述一些考虑探索-利用平衡的方法（例如bootstrapped DQN[8]），因为这类方法本质上是对不确定性度量的系统性利用。

Based on the above discussion, we can temporarily assume that the hidden state $s^h$ mainly contains an **experience pool** of past decision behaviors and the **models** trained from past decision behaviors. These elements both describe the DM's understanding of the underlying state transition mechanisms/environmental feedback and largely determine the update process of the strategy. Since $s^h$ also considers the DM's uncertainty of the state transition, we can use this framework to describe methods that consider the exploration-exploitation tradeoff (such as bootstrapped DQN[8]), because the essence of such methods is the systematic exploitation of knowledge of uncertainty.

通过上面的讨论，我们只是大致能够确定迭代机制$F$中所需要考虑的元，而这些元素之间具体应该如何互相影响目前还不清楚。因此，后续还需要进行大量工作。下面是一些我认为可以尝试的方向：从规范视角出发，尝试建立决策者的**最小化遗憾值**（regret minimization）序列决策问题（对单任务场景）或**元学习**问题（对多任务场景）以求解最优信息处理机制；从描述视角出发，探索和利用生理上的**记忆机制**与已有迭代学习（优化）算法的异同。

Through the above discussion, we are able to roughly determine the elements that need to be considered in $F$, but is not clear about how these elements should interact with each other. Therefore, more works are needed in the future. Following are some interesting future directions: from the normative perspective, we can attempt to formulate optimization problems for the information processing mechanisms, in the form of **regret minimization** (for scenarios with a single task) or **meta learning** (for scenarios with multiple tasks); from the description perspective, we can explore and exploit the similarities and differences between the **memory mechanism** in our brains and the existing iterative learning (optimization) algorithms.

## 总结 Summary

在这篇短文中，我尝试从更高的视角出发对决策行为进行分析。总体来说，我认为在建立决策者的效用最大化问题时可以进一步考虑决策者对历史信息的归纳和对未来的规划。但后者在本质上也是由前者所决定的，因此问题的关键在于找到合理的**对历史信息处理机制的表示**。不过，这里的讨论只是提出了一种分析视角，而并没有给出具体解决方案。（注：事实上，我认为要提出一套统一的建模框架可能是十分困难的。）

In this post, I try to analyze decision-making behavior from a more general perspective. I think that in formulating the DM's decision problem, we can further consider the DM's summaries of past information and expectations for the future. But because the latter is essentially determined by the former, the key to a good formulation is to find an appropriate representation of the **historical information processing mechanism**. However, the discussion here only presents a perspective for analysis and does not provide any practical solution. (Note: In fact, I think it may be very difficult to propose a unified modeling framework for this problem.)

至于在实际应用方面，我认为还是应该根据**奥卡姆剃刀原则**，先从基础的效用估计方法入手；根据经验，很多情况下并不需要使用过于复杂的建模元素。但我相信随着研究者信息获取和处理能力的提升、以及决策者面临的决策问题变得越来越复杂，在未来应该会涌现更多这方面的工作。

As for applications, I think we should follow the **principle of Occam's razor** and start with the basic utility estimation methods; according to my experience, in many cases, it is not necessary to use complex modeling elements. But I believe that as researchers' ability to acquire and process information increases and the decision problems faced by the DM become more complex, more work in this area should emerge in the future.


## 参考文献 References

[1] Kreps, D. (2018). Notes on the Theory of Choice. Routledge.

[2] Sims, C. A. (2003). Implications of rational inattention. Journal of monetary Economics, 50(3), 665-690.

[3] Gabaix, X. (2014). A sparsity-based model of bounded rationality. The Quarterly Journal of Economics, 129(4), 1661-1710.

[4] Masatlioglu, Y., & Nakajima, D. (2013). Choice by iterative search. Theoretical Economics, 8(3), 701-728.

[5] Rust, J. (1987). Optimal replacement of GMC bus engines: An empirical model of Harold Zurcher. Econometrica: Journal of the Econometric Society, 999-1033.

[6] Aguirregabiria, V., & Mira, P. (2010). Dynamic discrete choice structural models: A survey. Journal of Econometrics, 156(1), 38-67.

[7] Heckman, J. J., & Navarro, S. (2007). Dynamic discrete choice and dynamic treatment effects. Journal of Econometrics, 136(2), 341-396.

[8] Osband, I., Blundell, C., Pritzel, A., & Van Roy, B. (2016). Deep exploration via bootstrapped DQN. In Advances in neural information processing systems (pp. 4026-4034).
