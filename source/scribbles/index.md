---
title: Scribbles
mathjax: true
date: 2020-01-01 12:00:00
---

## Modeling

(19/10/11) 图卷积网络(GCN)较适用于建模局部动力系统，似乎不太适用于端到端直接预测。

(19/11/20) 仿真是一种成本较高的建模方式，因此在进行仿真时也需要考虑建模中的核心权衡：应该考虑哪些要素、抛弃哪些要素？

(19/12/08) batching的作用：在供需都在平面上均匀分布时，可以降低pickup distance；但如果需求为单点，则batching不能降低pickup distance。

(19/12/08) model-driven：通常需要使用非线性优化校准(calibrate)，容易mis-specify；data-driven：对数据的分布有较强依赖。

(19/12/28) 如何保持一个输入特征（动作action）的偏序：
1. 声明强结构、引入隐变量/参数，缺点在于计算复杂，容易mis-specify；
2. data-driven+局部线性化，缺点在于应用时可控空间小；
3. data-driven+加入辅助样本和相应损失函数（例如构造服从偏序关系的样本对）进行辅助训练，缺点在于不能完全保证偏序关系。
4. 对于实数空间$\mathbb{R}^n$上的偏序，可以考虑二次型效用函数$u(a,s,a') = a^T G(s) a'$，然后假设DM的行为$a$服从softmax原则；容易看出此时结果效用是服从$a'$上的偏序的。
5. 如果要进一步考虑多维输出（例如，离散概率分布）上的偏序，一种简单的方式是先映射到一个全序集合（例如实数轴）上，然后作为参数映射到对应的多维空间中。例如，对于离散概率分布，可以考虑一族泊松分布，其位置参数$\lambda$是全序的。这种做法的问题是仍需手动指定输出的结构，容易mis-specify。
6. 进一步考虑拆分real prob和subjective prob来引入optimistic因素；参考saliency theory。

(20/02/12) 建模本身可以看成是一种heuristic：相对于考虑全部信息，筛选出最重要的元素，形成tractable formulation。很多工作讨论的问题考虑因素过多，所提出的方法很难robust地应用到相应的实际问题中。

(20/02/24) 建模过程中，一定要分析求解结果在不同context下的稳定性以及对参数的敏感度。



## Inference & Control & Decision

(19/06/02) 在predict-then-optimize框架中，优化/控制器的迭代频率最好与预测器的迭代频率一致。例如，如果决策是实时的，那么模型最好保持相对实时的更新；否则一段时间过后数据分布可能会变化，导致模型预估精度下降。

(19/07/02) OR在实际应用中的思考：
1. 输入模型的准确度、决策的敏感度评估非常重要！
2. 拿到问题，第一步永远是明确关键结构和核心变量，尽快建立从核心变量到决策的映射。业务直觉的建立与持续迭代很重要。
3. 决策影响面越大，越需要谨慎，引入更仔细的仿真评估和灵敏度分析。

(19/07/29) 树模型的工作原理是找最优的分桶，而桶内数据点不做区分直接平均。因此可以认为是一种特征提取方式，分桶结果可以用作其他强表达模型的输入。

(20/01/01) 反事实(counterfactual)误差分析的套路是将$l(y_1-y_2,\hat{y}_1-\hat{y}_2)$拆分为$l(y_1,\hat{y}_1)$和$l(y_2,\hat{y}_2)$，再通过原有模型误差分析套路生成bound。主要区别在于需要分析决策倾向性(propensity score) $p(a|s)$对于样本分布的影响。

## Optimization

(19/11/14) primal/dual方法不一定非要在simplex上做局部搜索(local search)。如果问题本身或局部有更好的结构（例如组合性质），则应当利用局部结构加速优化进程。例子：二分图匹配中的KM算法。

(20/01/01) robust optimization思路整理:
1. 若每个单独的决策重要，但是预测的confidence不够，那么可以使用局部平均的方式提高预测精度。可以使用kNN、随机森林等方法。
2. DB提出的data-driven生成uncertainty set的思路是：假设参数服从某一分布，再通过假设检验给出分布的置信区间。个人感觉这种方式更适用于宏观决策、没有明确反馈的场景，不适用于实时控制场景。
3. model bootstrapping很难生成可以解析的uncertainty set，还是需要人工设计->怎么处理这个gap？
4. Xu论文中讨论的robust DP是转化为LP求解，只能处理reward不确定的情况，无法处理模型不确定的情况。
5. Adaptive opt: 忽略state、action对后续state的影响，直接考虑后续state的uncertainty。更加适用于长周期的宏观决策(例如电网控制)，而非前后state强相关的高频场景
6. 近期一些adversial的尝试可能能达到好的robust效果

(20/01/30) 新算法的实现往往需要引入新的状态变量，而主流程在以往的开发中并未留下足够的接口，从而导致随着时间迁移，代码越来越难改；不断重构是必要的。(特别是在solver的设计中)

(20/02/24) 整数规划review：
1. 设计求解算法/使用solver时，明确目的是要prove optimality还是search for feasible solution；
2. 一些formulation即使在计算上不可行(例如，指数多个约束)，也可能对理论分析、设计新的算法有帮助；特别地，很多组合优化问题可以通过加入大量的约束变成LP。
3. 很多heuristic的benchmark没有将solver tune得足够好，虽然仍然是有优势的。

(20/02/24) 组合优化review：
1. 一些常用思路：primal-dual；Lagranian dual guided greedy；原始空间的局部搜索；LP + smart randomization/sampling
2. vertex cover：尝试了CFT，效果较好。
    1. subgradient优化的效率很重要，决定整体收敛速度；
    2. 每步迭代中，用pricing来筛选column可以有效提高subgradient优化速度，但是需要更多迭代次数，应整体判断是否使用；
    3. 如果做column fixing，则dual的上界估计（LP最优解）会逐步下降（因为fixed primal variable不一定是最优，导致对应的primal LP最优解下降），此时该上界估计不能用来评估整体效果，只能用于辅助subgradient算法收敛。不过，column fixing的确能够提高优化效果，因为基础的dual-guided思路本质上与最优解还是有gap的。
3. bipartite matching：给每个node加上一个fix-value varible（该变量可以看成node与source或target相连的边）；KM可以看成是利用组合方法对这个extend problem做dual simplex？其中，"对等边找增广路径提高primal目标值"对应"令fix-value variable离开basis"；"调整dual增加等边"对应"令original variable进入basis"。
4. 如果子问题有很好的组合解决方法，可以考虑Branch and Bound + 子问题。怎么变得更general？

## Transportation

(19/09/13) 城市出行中，regular需求更关注服务稳定性，而MaaS的优势虽然灵活和可定制化(也可能有更好的服务)，但无法保证稳定，因此MaaS的主要空间在于irregular需求。由于需求的高支付意愿来源于lack of alternatives，MaaS概念内部的不同mode将会形成强替代效应。

(19/11/20) 宏观基本图对应“稳态”，因此由于突发事件导致的交通流变化不应该根据宏观基本图进行判断，而应该基于动力学模型(PDE)甚至是仿真进行判断。

(20/02/26) 物流场景与出行场景的本质区别是需求形态上的，这种差异导致了运行模式的差异，从而产生了不同的优化问题。

## Misc

(19/07/02) 大数据的作用有以下几点：
1. 由于存储原始数据变得可行，可以对历史数据进行更多事后的多维度的分析，数据的利用率得到提升。
2. 具有多源数据和实时数据流的处理能力，意味着可以进行更实时的决策。
本质上，仍然依赖于良好的直觉和业务认知！

(19/08/26) 与推荐/广告业务相比，城市服务业务的用户使用频率更低，因此数据量相对有限。于此同时，偏好结构更清晰：价格、时间、服务、稳定性。因此，structural modeling/stated preference仍有一定的必要性。

(19/12/21) (S)PDE的基本求解思路：
1. 对于空间问题，在值域逼近的话用有限差分(FDM)；在函数域逼近则用有限元(FEM)或者谱分解。
2. 如果有时间上的演化，则可将实数域上的PDE转化为函数域上(关于时间)的ODE。ODE中，每个无穷小时间片的演化对应一个函数域上的变换算子$A$，该算子在时间$t$的演化下组成算子半群$\\{e^{tA}\\}_t$。进而可以通过算子半群理论对算法的收敛性进行分析。
3. 对SPDE，需要掌握大规模随机采样的方式，特别是如何高效率对BM/brownian sheet进行采样。一种方式是通过circulant embedding+FFT进行精确采样。另一种方式是用Karhunen–Loève分解目标过程，然后分别采样来逼近。
