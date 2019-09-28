---
title:       "What I've Learned in the Past Year (2018-19)"
categories: Reflection
tags:
- Shared Mobility
- Behavior Modeling
- Integer Programming
mathjax:     true
date:        2019-06-23 00:00:00
---

In the past year, I have been in the ride-hailing industry and have encountered many interesting problems. In this post, I would like to summarize my experience in formulating and tackling these problems and share my personal ideas on future directions.

<!--more-->

## Understanding Ride-hailing Businesses From a Macroscopic View

Comparing with other travel modes such as driving and public transit, ride-hailing could provide a higher degree of flexibility to riders: they can wait for drivers to pick them up. Because the riders are very sensitive to the waiting time for pick up, densities and network effects are crucial for a ride-hailing business to gain market power. To further illustrate the connection between densities and market power, in the following I will introduce a static equilibrium framework which was first developed (as far as I know) by Arnott[1] and recently reintroduced by Castillo[2]:

Assume in a ride-hailing system there are $D$ drivers and currently $I$ of them are available. It is not hard to see that, for an incoming rider, the expected pickup distance $d$ and the expected waiting time for pickup $w$ are monotonically decreasing with respect to the availability $I$. If we further assume $T$ to be the average trip time, then the (unit-time) service capacity $R$ that the system can consistently provide is given by

$$D = I + (T + w(I)) R. \tag{1.1}$$

To understand this equilibrium equation, simply notice that among the $D$ total drivers, $I$ are available, $R$ are going to finish their rides in 1 minute, $R$ are in 2 minutes, etc.

However, the supply $D$ and the demand $R$ depend on external factors such as prices. Specifically, we might assume $D$ depends on the expected income and $R$ depends on the expected price and waiting time:

$$D = G(r_D), r_D = p_D \cdot \frac{RT}{D};$$

$$R = F(r_R), r_R = a -  b p_R - c w(I),$$

where $a,b,c,F,G$ are given constants.

Now, suppose that the operator can scale $D$ and $I$ by the same factor $\alpha > 1$. On the one hand, the operator can have

$$p_D \propto \frac{D}{R}$$

to maintain the same income level $r_D$ for drivers, and $\frac{D}{R}$ is monotonically decreasing as $\alpha$ increases because $w(I)$ is decreasing. On the other hand, because $w(I)$ is smaller, the operator can raise the price $p_R$ while maintaining the same utility level $r_R$. Therefore, the network effect allows the one with higher market share to enjoy a greater profit $p_R - p_D$. This operator can further exploit this extra profit in a flexible way to drive any competitor out of the market.

(**Remark:** the analysis here is neither rigorous not complete because it ignores the impacts of stochasticity and choice behavior from riders and drivers, and is not able to describe dynamic decisions in competitions. Later, if I have enough time, I would write another blog for models and simulation frameworks that include more details.)

Therefore, for operators with low market shares, it is necessary to both improve operating efficiency at microscopic levels and exploit heterogeneity to provide personalized services. I am not planning to discuss the tools and strategies that can be employed because that might be too specific; however, I am going to give some simple examples that partially indicate what factors practitioners should focus on.

In the first example, let us consider the following scenario: Alice needs to take a ride at the morning peak of a working day. Alice lives in a high-rise building, and it takes a certain amount of time (say, 3 minutes) for her to go downstairs. She now has a variety of time management options: she can go downstairs first, and then request a ride; or she can submit the request at home, and go downstairs when the remaining ETA approaches 3 minutes.

According to her experience, during this period there are generally fewer drivers, and the waiting times for response and pickup are usually large. Waiting at home can bring her more utility than waiting on the roadside because she can do something else during this time.

**Question:** Suppose the response time can be ignored, and the pickup time follows a log-normal distribution. If Alice only considers the average pickup time, which time management option will she choose? What impact will this decision have on the total waiting time for this trip? If instead Alice considers the distribution of pickup time, will the outcome be different?

In the second example, let us consider the operator's dispatch decisions for drivers. Because the driver is an independent individual, the operator cannot force the driver to accept the dispatch; but she can incentivize the driver to accept the dispatch through some incentives. Now, suppose there is a drop in the response rate at the airport. Operator Alice wants to dispatch some drivers to the airport to serve the extra riders. Alice found that driver Bob is available and is in the business district near the airport. She sends him a dispatch request immediately. At the same time, through careful calculations, Alice finds that if Bob travels to the airport, his expected future earnings would increase. Based on this information, she believes that this dispatch is in the interest of the driver, so she only issues a very low incentive to Bob.

Bob receives the dispatch request. According to his past experience at the airport, the queueing time at the airport is usually long, and a large percentage of the requests from the airport have destinations near his current location. If this happens again, he will earn less compared to stay in the current location. Of course, he also knows that he might be assigned a trip with long distances and high incomes at the airport; but he is not able to calculate the accurate probability of such events.

**Question:** Suppose Bob is risk-averse. How will he respond to the dispatch request? If Bob is risk-neutral, but has to rely on his personal experience in the past for decisions, what will happen? Finally, what incentives can Alice use to increase Bob's probability of accepting the dispatch request without increasing the expected cost of incentives?


## Personalized Choice Problem

In the above, I mention the importance of exploiting heterogeneous information. However, in general, the use of this information is complex and requires context-based modeling. Nevertheless, in reality, there are many decision-making problems (not restricted to the ride-hailing context!) that can be modeled in a way that I call "personalized choice problems": the operator's decision is to provide/recommend options to users and the decisions to different users are only loosely dependent. In fact, in some contexts, even if the control variables are in a continuum, the users' limited attention and preference structure require the operator to choose from only a finite set of representative options. (Simply imagine you receive a coupon that can save you $3.14.) In this subsection, I will first introduce basic techniques and major issues to be concerned through a simple problem; then, I will generalize these techniques to some of the more practical contexts.

Deterministic choices from finite sets are generally referred to as SOS1 (special ordered set of type one) constraints: for a user $i$, the decision to choose an option $j$ from the set $\\{1,\cdots,M\\}$ can be modeled as

$$\sum\_j x\_{ij} = 1, \\ x\_{ij} \in \\{0,1\\}. \tag{2.1}$$

In equation (2.1), the decision variable $x\_{ij}$ indicates whether the operator should recommend option $j$ to user $i$. I have discussed this type of constraints in details in [one of my previous posts](https://hanqiu92.github.io/blogs/2016/IP3_201610/). However, in many real-world applications, deterministic decisions are not required because the operator does not need constraints other than the regular ones $\sum\_j x\_{ij} = 1$ to hold in the ex-post sense. For example, suppose the operator wants to control the budget of one resource by the amount $c\_0$: $\sum\_{ij} x\_{ij}c\_{ij}\leq c\_0$, but she faces the same kind of decisions every day during a very long period (say, one month) and she only need to control the total budget. In this case, it is fine to relax the integer program to a linear program and consider stochastic decisions. Moreover, in situations that explorations are encouraged, stochastic decisions are preferred because they have less entropy. Therefore, in the following discussion, I will focus on stochastic decisions and linear programs.

Now, let us start with a very simple case: we want to maximize our total gain $r$ while satisfying budget constraints on $K$ independent resources $c^{k}$:

$$\begin{aligned} \max & \sum\_{ij} x\_{ij}r\_{ij} \\\\  s.t. & \sum\_{ij} x\_{ij}c\_{ij}^{k}\leq 0 \\\\ & \sum\_j x\_{ij} = 1, \ \forall i \\\\ & x\_{ij} \geq 0 \end{aligned} \tag{2.2}$$

To solve the problem (2.2) efficiently in cases when the number of users $N$ is huge, we can consider its dual problem

$$\min\_{\lambda \geq 0} \sum\_i \max\_j \\{ r\_{ij} - \sum\_k \lambda\_k c\_{ij}^{k} \\}, \tag{2.3}$$

which is globally convex and can be solved easily with existing derivative-free methods (e.g., [Powell's method](https://en.wikipedia.org/wiki/Powell%27s_method)).

However, in reality, a serious problem with the above dual approach is that the estimations of $r\_{ij}$ and $c\_{ij}$ are noisy and the decisions $x$ are unstable. To explain why this is a problem, I conduct a simple simulation experiment. Let us consider problem (2.2) with $N = 100,000$ users and a single budget constraint, and the real gains and costs are generated as follows

$$r\_{ij} = \beta\_i j^{\alpha\_i}, \\ c\_{ij} = \gamma\_i (j - 5), \\ j \in \\{0,\cdots,10\\}; $$

$$\alpha\_i \sim B(5,5), \ln(\beta\_i) \sim N(1,0.3), \ln(\gamma\_i) \sim N(1,0.3).$$

To model noises in the estimations, we consider the following $\sim 10\%$ perturbation for each input $r\_{ij}$ or $c\_{ij}$

$$x' = e^z\cdot x, z\sim N(0,0.1).$$

The following figure shows the simulation results averaged over 20 runs; it is shown that the dual approach (2.3), which corresponds the data point above the tick "all", is not robust to estimation noises (with a $2\%$ loss compared with the background optimal solution).

<img width=500 src="/blogs/images/yearly_summaries/opt_result_201906.png">

A simple way to improve the robustness of approach (2.3) is to do clustering. As an illustration, in the same simulation experiment, I first apply PCA to map the $(r\_i,c\_i)$ tuple for each user $i$ into a three-dimensional vector, and then apply k-means clustering to map the users into $N\_G$ groups, where $N\_G \in \\{10,50,100,500,1000,5000\\}$. In the same figure above, we can see that by selecting an appropriate $N\_G$ (in our case, $N\_G = 500$), we can do slightly better than the direct dual approach. (**Remark:** one should notice that this quantitative result on $N\_G$ is case-dependent; for example, one can investigate what would happen if the estimations of costs $c$ are much less noisy, e.g., with a $\sim 1\%$ perturbation.)

Though the above case is simple, the techniques considered can be readily applied to more complex settings. For example, when a dual solution $\lambda$ is determined, the operator can provide personalized recommendations in an online manner: when a user $i$ submits a request, the operator can make a choice from available options immediately by maximizing $r\_{ij} - \sum\_k \lambda\_k c\_{ij}^{k}$. Moreover, if we further notice that the dual approach (2.3) resembles the following stochastic program

$$\min\_{\lambda \geq 0} E\_{r,c \sim F(\cdot)} \max\_j \\{ r\_{j} - \sum\_k \lambda\_k c\_{j}^{k} \\}, \tag{2.4}$$

where $F$ can be interpreted as the populational distribution of gains and costs, we will find that by combining smartly with control methods such as PID and RL, the dual approach (2.3) can be applied to personalized decision-making problems in nonstationary environments.

(**Question:** How to generalize the clustering method in the simulation experiment to the online control settings?)

In some other contexts, the operator has to impose some restrictions on the option set for the users. For example, some options are mutually exclusive and cannot be provided at the same time. Although it seems that the operator needs to design the option set before getting to personalized control step, it is possible and even better to make the two types of decisions at the same time. At the moment of writing this blog, I have come up with two possible approaches: first is to directly consider the dual problem of the relaxed LP formulation, which is tractable when the restriction on the option set has simple structures; second is to develop case-specific [branch and cut](https://en.wikipedia.org/wiki/Branch_and_cut) algorithms, which first relax the restriction on the option set and then correct the solution by adding cuts. Undoubtedly, the knowledge from the dual approach (2.3) is crucial in developing both approaches.


## Personalized Behavior Modeling

In the previous subsection, I discussed how to optimize personalized decisions given estimated inputs. But from the simulation experiment, we know that the estimation accuracy of inputs can have significant impacts on the final performance. In this subsection, I will share my experience (though limited) on how to build models on personalized responses that are beneficial to subsequent controls.

In the following discussion, for notational consistency, I would use terms in RL to describe concepts such as control and response. Specifically, we assume the personalized context is represented by the state $s$, the control variable is represented by the action $a$, and the response is denoted as $r$. Our main objective is to construct a model for $r(s,a)$ given a dataset $D=\\{(s,a,r)\\}$.

Although nonparametric models (such as random forests and multilayer perceptrons) can often capture complex nonlinear relationships among features and can achieve lower estimation errors, in many cases they are not preferred because their predictions are not guaranteed to have the desired structure; this may be due to a limited sample size or other more complex reasons. For instance, in a promotional scenario, the operator would expect a higher discount $a$ for the same user to result in a larger purchase $r$. However, the dataset $D$ is likely to contain not only information about users' heterogeneous responses but also the information of the previous personalized control scheme. That is, the samples in $D$ might be generated according to some control strategy $\pi$

$$a \sim \pi(s), s \sim F(\cdot). \tag{3.1}$$

In this case, using high-capacity nonparametric models may mix up the two kinds of information and provide unintuitive predictions.

When the control variable has simple structures (e.g., prices), we usually have some prior knowledge on its impact on users' responses. For example, a user's purchase probability $r$ with respect to the fare discount $a$ is usually modeled as a sigmoid function

$$r = \frac{1}{1 + e^{- \alpha (u + ka)}}. \tag{3.2}$$

With neural networks, we can encode such prior knowledge in the output layer and consider the following semi-parametric form

$$r(s,a) = \frac{1}{1 + e^{- \alpha(s) (u(s) + k(s)a)}}, \tag{3.3}$$

in which $\alpha$, $u$ and $k$ can be modeled by usual multilayer perceptrons.

But when the control variable has complex structures (e.g., coupon packages), it is no longer analytically and computationally tractable to specify a good prior on its impact. (**Remark:** it is not impossible; in one of my previous work I used dynamic programming to generate features for travelers' coupon redemption behavior. But it is too costly.) However, one may still have some local structural knowledge: for example, even though one cannot tell whether a package of three coupons that each save \$1 is more valuable than a single coupon that saves \$2, we are sure that the same package is more valuable than a single coupon that saves only \$1. When the set of available options has such local structures, we can consider the following approach: we first estimate a nonparametric $r(s,a)$, and then re-fit the prediction $r$ with the local prior on $a$.

In both approaches introduced above, the impact of the previous control strategy $\pi$ has not been considered. This problem is not trivial and has been addressed in the contextual bandit literature. Because of my lack of experience, I am not planning to delve into this topic.

Finally, I want to discuss the uncertainty in the estimates. (One should be aware of the distinction between the **uncertainty of observations** and the **uncertainty of estimates**: the former reveals the stochastic nature of the observation, while the latter implies our lack of sufficient information.) From the simulation experiment in the last subsection, we know that the uncertainty information of our estimates can be useful in designing good clustering strategies. However, it is not trivial to obtain such information accurately.

A commonly used method for estimating the uncertainty of estimation is bootstrapping, which is to generate training samples and obtain prediction models by resampling multiple times, and then use the distribution of predicted values of these models to evaluate the uncertainty. However, the performance of this method is strongly dependent on the model structure, so the modeler needs to choose carefully an appropriate model to balance the generalization capability and the accuracy of the uncertainty estimates. For example, on the one hand, in the classic bootstrap method, the uncertainty of the mean estimate is obtained by calculating the standard deviation of the means of the resampled samples, and this uncertainty estimate is fairly accurate; but at the same time, the generalization of this statistical method is poor: for every possible context $s$, this method requires an independent set of data in the estimation. On the other hand, the linear regression model is known to have strong generalization capability; but because of its low degree of freedom, resampling will not have a great impact on the estimation result, and the uncertainty of the estimate cannot be accurately described. Obviously, to resolve this dilemma there need to be more works in the future.

## Summary and Future Directions

In above, I mentioned:

1. From a macroscopic perspective, a higher market share can lead to a greater price advantage. So, it is critical for new operators to improve microscopic operating efficiency and provide personalized services.
2. For a large part of the decision problems in reality, it is enough to use LP and IP for modeling and optimization; but people should be cautious about the estimation quality of the inputs.
3. Introduction of prior structural information into the behavioral models can be beneficial.

Based on these observations, in the future, I plan to:

1. Keep thinking about optimization under uncertainty. Specifically, I am quite interested in scenarios that need to consider structural decision-making capabilities and online learning capabilities.
2. Keep working on behavioral models within the sequential decision framework and exploring better methods to integrate prior information into high-capacity models or to represent highly complex structural objects/events.


## Reference

[1] Arnott, R. (1996). Taxi travel should be subsidized. Journal of Urban Economics, 40(3), 316-333.

[2] Castillo, J. C., Knoepfle, D., & Weyl, G. (2017, June). Surge pricing solves the wild goose chase. In Proceedings of the 2017 ACM Conference on Economics and Computation (pp. 241-242). ACM.
