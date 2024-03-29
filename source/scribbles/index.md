---
title: Scribbles
mathjax: true
date: 2020-01-01 12:00:00
---

## Modeling

### Methodology

(20/02/24) 建模过程中，要记得分析预测结果在不同context下的稳定性以及对参数的敏感度。
(20/07/13) 很多场景中，模型内存在一些优化问题，不容易进行宏观近似；此时可以考虑通过启发式方法近似求解优化问题来节省计算开销——当然，问题是优化结果对输入数据不一定可导。
(20/09/01) 决策行为建模思考：
1. 对于一个确定的决策任务，给定充分的数据，机器学习模型可以很好地拟合和预测人类行为；
2. 对于meta-task，util-max是一种较好的建模人类行为的思路；
3. 然而，目前的技术手段很难对一个人（的所有决策行为）进行仿真模拟——需要对上述工具采取一种巧妙的平衡；可能从强化学习的应用中获取灵感
4. general modeling heuristic可能比ad-hoc accurate model更合理。特别地，为了模拟agent决策中的heuristic所设计的方案其实也更容易被其他建模者所理解。

(20/12/31) recommendation影响的是attention，而不是choice。
(21/04/15) IRL的一个问题在于如果样本不足，学出来的reward形式容易overfit，无法泛化到样本外的特征上。

### Structure Design

(19/10/11) 图卷积网络(GCN)较适用于建模局部动力系统，似乎不太适用于端到端直接预测。
(19/12/28) 如何保持一个输入特征（动作action）的偏序：
1. 声明强结构、引入隐变量/参数，缺点在于计算复杂，容易mis-specify；
2. data-driven+局部线性化，缺点在于应用时可控空间小；
3. data-driven+加入辅助样本和相应损失函数（例如构造服从偏序关系的样本对）进行辅助训练，缺点在于不能完全保证偏序关系。
4. 对于实数空间$\mathbb{R}^n$上的偏序，可以考虑二次型效用函数$u(a,s,a') = a^T G(s) a'$，然后假设DM的行为$a$服从softmax原则；容易看出此时结果效用是服从$a'$上的偏序的。
5. 如果要进一步考虑多维输出（例如，离散概率分布）上的偏序，一种简单的方式是先映射到一个全序集合（例如实数轴）上，然后作为参数映射到对应的多维空间中。例如，对于离散概率分布，可以考虑一族泊松分布，其位置参数$\lambda$是全序的。这种做法的问题是仍需手动指定输出的结构，容易mis-specify。
6. 进一步考虑拆分real prob和subjective prob来引入optimistic因素；参考saliency theory。
(21/04/15) attention机制是sparse的；attention权重可以作为rl中的reward.

### Human Decision Behavior

(20/10/01) 网约车用户调研：
1. 高频基本与通勤挂钩 —— 需要通过结构化的mode choice模型判断，似乎公交的preference很低
2. 忠诚度与体验有关 —— 稳定的体验可以保证暂时ignore other options；可通过其他app的安装特征判断是否与其他option有较强的绑定
3. 价格敏感度与支付方式有关 —— 公司报销的相对不太敏感；高价车型部分反映供给紧张情况
4. 其他 —— 乘客希望有自定义偏好设定；对于福利主要关注应答时间；新司机由于业务不熟练，提供的体验更差
(21/04/15) 劳动经济学基础假设：
1. 劳动者的资源禀赋包括：时间，以及周期性的固定盈余（租金等）。劳动者通过工作时长换取收入。
2. 劳动者的效用和支出是关于消费水平和休息水平的函数。决策问题是固定效用优化支出，或者固定支出优化效用。决策问题的解也可以看成是效用无差异曲线与支出无差异曲线的切点。
(21/04/15) 使用causal graph来分析人类决策行为的一个问题点在于，忽略了人类对于未来的推演能力（心智理论），即实际情况可能是双向因果关系。

## Inference

### Machine Learning

(20/07/04) 树/SVM等基于分类的模型是以split为核心（桶内数据点直接平均作为预测值），复杂度与桶数量有关；boosting进一步提供了高效交叉分桶的方式。LR/NN等基于回归的模型是以fit为核心（每个数据点贡献自己的loss影响整体模型），复杂度与模型参数有关。**问题：**是否从duality角度存在优化空间？例如，adaptive complexity/model selection？
(20/10/04) split（VC维）与fit的一个区别在于，前者类似于分析学中的simple function，后者则类似于逼近理论中的FFT、小波等。对于split，表示复杂度 = 模型复杂度 * 局部定义数；特别地，如果原数据有复杂的非线性，那么局部线性需要定义非常多局部

### Causal Inference

(20/01/01) 反事实(counterfactual)误差分析的套路是将$l(y_1-y_2,\hat{y}_1-\hat{y}_2)$拆分为$l(y_1,\hat{y}_1)$和$l(y_2,\hat{y}_2)$，再通过原有模型误差分析套路生成bound。主要区别在于需要分析决策倾向性(propensity score) $p(a|s)$对于样本分布的影响。
(20/07/01) PSM是一种常用方法，但是效果相当于做complete random实验而不是block random实验，因此当达到了full random效果后，随着过滤样本数增大（差异评估阈值下降）时，使用PSM反而增加了数据集之间（在特征层面上）的差异，因为样本量下降了。相反，直接对特征维度进行matching可以直接刻画数据集间的差异。（但是在数据量充分的情况下这种现象不太可能发生）
(20/07/01) dragonnet: 用propensity score来估计ATE，可以降低估计方差，但是牺牲了个人维度属性，个性化估计会有偏
(20/10/11) causal nn：fit的角度，通过权重分布将单点估计转化为整体加权估计，然后利用分部积分进行转化。实际应用可能比较tricky。
(20/12/31) 在多treatment场景中，做单treatment的实验结果分析会有什么效率损失？噪声更大，实验结果置信度下降
(21/04/15) 在估计treatment effect时，如果无法得到准确估计，可以退而求其次，根据一些正则性假设，生成上下界估计，并进行敏感度分析。
(21/04/15) 当treatment effect较弱时，使用传统的树模型可能会导致估计的treatment effect偏向于0，因为很难产生有效的split。
(21/04/15) 合成控制中，系数的稀疏性是一个重要性质。
(21/04/15) 合成控制的可靠性可以通过“extended pre-intervention period”的拟合准确度评估。特别地，根据理论分析，合成控制主要适用于样本量少+趋势强/噪声弱的场景；如果样本量大+噪声大，合成控制本身可能会overfit，引入更多noise。另一方面，可进一步通过类似于重采样的方式，评估效果的显著性：如果实验元素的估计效果显著差异于对照组元素的效果分布，则置信。这种评估更多依赖人工经验，因为样本少时p-value不置信。
(21/04/15) 通过观测数据和实验挖掘因果关系：
* 目标：根据数据和实验，找到因果图的结构。
* 关键假设：faithfulness，即概率分布与因果图关于随机变量条件独立的判断的一致性。基于这一假设，可以通过条件独立性推断因果图的方向，例如PC算法。若要进一步通过增量假设（例如，噪声的结构）得到更多关于因果图的结构信息，则需要对具体情况进行具体分析，没有简单结论。
* 应用场景：医药、物理等基础研究场景；社会科学&政策研究（可能存在hidden confounder）；AGI
(21/04/15) mediator & moderator: 在现有因果关系估计基础上，试图进一步拆解不同因果路径的影响。可以使用因果图或者结构模型进行分析。常用于基于实验设计的因果关系挖掘中。
(21/04/15) honest做法迁移nn的思路：结构+权重拆分训练。可以与超参数的优化结合。
* 简单方法：将模型结构切分为两部分，先用第一部分初始化全部权重，然后用第二部分数据fine-tune输出层权重。
* general方法：一部分数据做nas或者sparse connection search，另一部分训练权重。
(21/12/25) 当特征X维度少时，可以直接分层统计计算ATE；否则，需要通过模型简化X->y的复杂度。

### Time Series

(20/10/01) 宏观时序分析的缺点：如果外部存在持续影响因素，那么在评估一个shock的效果时置信度和时效性存在trade-off
(20/12/31) 通过观察bias和loss随时间的演化，可以一定程度上判断系统的非平稳性。
(20/12/31) 时间序列中，对之前时间段信息的利用可以通过多种时间粒度（小时、天、周）来提取不同层次的信息。在此之上，通过引入时间段的encoding特征，可以使用attention来自动化地根据不同场景（例如是否节假日）选取适当的建模信息。

## Decision & Optimization

### Methodology

(19/06/02) 在predict-then-optimize框架中，优化/控制器的迭代频率最好与预测器的迭代频率一致。例如，如果决策是实时的，那么模型最好保持相对实时的更新；否则一段时间过后数据分布可能会变化，导致模型预估精度下降。

(19/07/02) OR在实际应用中的思考：
1. 输入模型的准确度、决策的敏感度评估非常重要
2. 对一个具体问题，先明确关键结构和核心变量，尽快建立从核心变量到决策的映射。业务直觉的建立与持续迭代很重要。
3. 决策影响面越大，越需要谨慎，引入更仔细的仿真评估和灵敏度分析。

(20/10/02) perception - decision - execution框架更多是人机交互的逻辑，可以extend到更多场景中

(20/12/31) random action的优点:
1. 增加噪声，降低自身的exploitability，同时保持对方的attention
2. 增加从对方的反馈行为中能获取的信息量

(21/01/10) task-specific方法存在数据利用的效率瓶颈，在数据量少的场景中较难取得好的效果。因此，一个重点的研究方向，是如何从不同任务下的数据中，归纳出可复用的决策思路（与meta-learning类似）。下面是一些可尝试的方向：
* 通过半监督和表示学习，判断数据样本的可复用性，然后加以利用；
* 从人类决策行为中，抽象出通用的inductive bias，然后设计相应框架。例如，模块化/stitching。

### Integer Programming & Combinatorial Optimization

(19/11/14) primal/dual方法不一定非要在simplex上做局部搜索。如果问题本身或局部有更好的结构（例如组合性质），则应当利用局部结构加速优化进程。例子：二分图匹配中的KM算法。
(20/02/24) 一些formulation即使在计算上不可行(例如，指数多个约束)，也可能对理论分析、设计新的算法有帮助：SOS system等。
(20/07/01) 对于单个整数可行点，可以利用可行域的组合结构加速搜索效率。**问题：**很多问题的求解效率只是在单个整数解处高，一旦同时考虑多个整数解，就不如LP，此时怎么利用组合结构？
(20/07/01) QAOA的基本思路：考虑解{0,1}^n的复合态（由n个qubit表示），通过（参数化的）约束和解维度的酉矩阵对复合态作用，使得最终演化结果在期望上收益接近最优。演化/仿真过程是量子的，而参数的求解是经典的（低维求解）。**问题：**为什么可以用酉矩阵进行作用？
(20/02/24) 组合优化一些常用思路：primal-dual；Lagranian dual guided greedy；原始空间的局部搜索heuristic；LP + randomization/sampling
(20/02/24) CFT for vertex cover:
      1. subgradient优化的效率很重要，决定整体收敛速度；
      2. 每步迭代中，用pricing来筛选column可以有效提高subgradient优化速度，但是需要更多迭代次数，应整体判断是否使用；
      3. 如果做column fixing，则dual的上界估计（LP最优解）会逐步下降（因为fixed primal variable不一定是最优，导致对应的primal LP最优解下降），此时该上界估计不能用来评估整体效果，只能用于辅助subgradient算法收敛。不过，column fixing的确能够提高优化效果，因为基础的dual-guided思路本质上与最优解还是有gap的。

(20/02/24) bipartite matching：

        给每个node加上一个fix-value varible（该变量可以看成node与source或target相连的边）；
        KM可以看成是利用组合方法对这个extend problem做dual simplex？
        其中，"对等边找增广路径提高primal目标值"对应"令fix-value variable离开basis"；"调整dual增加等边"对应"令original variable进入basis"。

(20/02/24) lapmod for bipartite matching:

        step 1a: (cr) 对extended formulation，令I（对应row的col）进入basis，然后在原col集合中，对每个col j选一个row（连边），col j的对偶值由该边决定。
        step 1b: (rt) 对于只连了一条边的row i，对应的j的对偶值完全由i决定，因此，如果令i_b离开B，将第二大的值对应的row加入B，并对对偶值u和v进行更新，则对对偶值进行了优化。
        step 2: (rr) 选一个free row i, 算出两个最小的slack和对应的边，并令i_b离开B，将这两条边加入B，对应最小slack的col j的一条边离开B。（等于是B的2换2操作，而且通过augment可以进一步降低i_b）
        step 3: (ag) 选一个free row i，找shortest path，path上的都进入B。  

(20/10/06) 内点法和单纯形法的比较：
内点法：几何角度，完全从可行域内部出发，迭代时不仅需要注意优化，还需要注意不要离边界太近，否则后续很难离开边界。
单纯形法：组合+几何角度，完全从顶点出发；这导致只在少数场景，例如LP中可以使用
(20/10/06) matriod和submodular的概念可以帮助理解为何简单的greedy+rounding能取得较好的效果

(21/04/15) 多目标优化方法：
1. 序列带约束求解。该方法的一大问题是对约束的设定敏感，约束的微弱变化导致解的大幅波动。另外，目标多了以后，顺序影响很大：排在后面的目标要么优化空间小，要么大幅削弱此前的优化效果。
2. 加权。可以利用已有解来初始化对偶变量，从而提速：先对单目标求解并存储dual变量，然后用这些变量进行加权作为初始值。
3. 加权+约束求解兜底：从全局做加权优化，然后对违反约束的个体重新求解。加权相比序列约束更可控，故应将约束作为静态配置，加权权重作为主要控制优化点。

### Robust Optimization

(20/01/01) 
1. 若每个单独的决策重要，但是预测的confidence不够，那么可以使用局部平均的方式提高预测精度。可以使用kNN、随机森林等方法。
2. DB提出的data-driven生成uncertainty set的思路是：假设参数服从某一分布，再通过假设检验给出分布的置信区间。个人感觉这种方式更适用于宏观决策、没有明确反馈的场景，不适用于实时控制场景。
3. Xu论文中讨论的robust DP是转化为LP求解，只能处理reward不确定的情况，无法处理模型不确定的情况。
4. Adaptive opt: 忽略state、action对后续state的影响，直接考虑后续state的uncertainty。更加适用于长周期的宏观决策(例如电网控制)，而非前后state强相关的高频场景

### Exploration & Exploitation

(20/10/05) 在复杂函数的bandit分析中，需要用到reward function所处的函数空间的维度来评估一处样本的信息量对函数整体的估计准确度的影响。此时需要一些通用的维度定义。其中，可以考虑
1. eluder dimension: 一种函数空间连续性的维度评估，可以用在任意函数空间中
2. state-action matrix rank

### Reinforcement Learning

(20/12/25) self-imitation：对于random exploration中产生的好trajectory，可以将其当做expert illustration，通过BC方法来引导后续训练。

### Game Theory

(21/04/15) correlated equilibrium：作为nash equilibrium的扩展概念，主要差异是将决策空间从独立概率分布的乘积拓展为联合概率分布，因此允许更多均衡的存在性。
(21/04/15) blackwell approachability: iteratively solve the minmax problem w.r.t. a projected (half-space) sub-problem, then it is guaranteed that it will converge to the set. for equilibrium, the projection happens to be "regret matching".
(21/04/15) 在评估中，AIVAT需要遍历所有可能发生的历史，因此计算量可能很大，是否能够仅考虑部分历史？
(21/04/15) online policy searching：可以使用alternative payoff（值估计）来进行引导，但是过度利用反而会增加了exploitability。一个有趣的观察是，抽象后，policy会变弱，但是value通常仍然很准，因此使用粗糙的value来引导精细的policy是较合理的。
(21/05/15) Single-deep CFR：在deep CFR的基础上，通过对历史regret模型的采样来替代对平均policy模型的学习。
(21/05/15) DREAM：在deep CFR的基础上，考虑outcome sampling以降低单次迭代的计算开销。然后，进一步通过增加模型预估的baseline来降低采样误差：使用expected SARSA，训练以下等式$Q(s(h),a) = R(h,a) + \mathbb{E}_{h'|\pi_{-i}} \sum_{a'} \pi(a'|s(h')) Q(s(h'),a')$。
(21/05/15) 怎么做depth-limited搜索？需要对信息集$I$进行扩充，以囊括自身策略$\pi$所产生的信息量：
* CFR-D: 增加其他agent可能处于的信息集和相应到达概率，以在求解subgame策略时，考虑其他agent的deviation动机，从而得到更robust的subgame策略；
* ReBeL: 增加其他agent对自身策略的belief（PBS=关于隐藏信息的概率分布），从而在求解trunk策略时，考虑自身策略对其他agent的belief的影响，从而得到更robust的trunk策略。
  * 应用ReBeL所需的先决条件：1. 实现belief与history/info set的相互转化方式；2. 实现CFR-D/CFR-AVG，用于robust求解subgame。
(21/05/15) (deep) cfr的regret计算提效思路整理：
1. 做多次outcome sampling rollout得到更准确的regret估计值。问题在于rollout时仍需要调用模型（生成策略），计算效率没有明显提升。
2. 结合outcome sampling和external sampling，在浅层用OS，深层用ES，以平衡计算成本和误差。
3. 使用类似于mcts的方式，在node处做多次有限深度搜索估计regret。问题在于如何保证模型预估与实际的Q值（受当前策略影响）是一致的。
(21/05/20) 进化博弈论（EGT）相比于传统博弈论，更关心群体中不同策略占比的演化过程。evolutionary stable strategy（ESS）是一个与纳什均衡（NE）不完全一致的概念。
(21/05/20) fictitious play（FP）: 每一步中，固定其他agent的策略，计算自己的最优响应策略（BR）；长期来看，这些最优响应策略的平均值（在一定条件下）收敛到NE。deep variants是用RL等方法求解最优响应策略。

## Transportation & Urban Studies

(19/09/13) 城市出行中，regular需求更关注服务稳定性，而MaaS的优势虽然灵活和可定制化(也可能有更好的服务)，但无法保证稳定，因此MaaS的主要空间在于irregular需求。由于需求的高支付意愿来源于lack of alternatives，MaaS概念内部的不同mode将会形成强替代效应。
(19/11/20) 宏观基本图对应“稳态”，因此由于突发事件导致的交通流变化不应该根据宏观基本图进行判断，而应该基于动力学模型(PDE)甚至是仿真进行判断。
(20/02/26) 物流场景与出行场景的本质区别是需求形态上的，这种差异导致了运行模式的差异，从而产生了不同的优化问题。
(20/05/30) 在供需在空间上均匀分布时，batching可以降低pickup distance；但如果需求或供给为单点，则没有明显效果。这意味着规模收益需要通过车辆deadheading实现，存在tradeoff！
(20/07/01) 区域/路网转换：
* 长方形、六边形体系：用于分析，优点是计算与转换方便，便于数据聚合和展示；缺点是不能反映区域间路网的连通性
* 路网体系：可以准确描述微观行为，缺点在于计算成本高
(21/04/15) 城市交通新业态：热点从shared mobility到mobility service的转变，即从强调去中心化的资源共享（个体主导=C2C）回到中心化的服务上（企业主导=B2C）。

## Code Reading & Development

### Optimization Solvers

(20/01/30) 新算法的实现往往需要引入新的状态变量，而主流程在以往的开发中并未留下足够的接口，从而导致随着时间迁移，代码越来越难改；不断重构是必要的。(特别是在solver的设计中)

* SCIP:
  * 线性domain prop的实现在cons_linear.c的propagateCons方法中
* CLP:
  * dual row/chuzr:
        用row pivot类实现，更灵活；实现中包括dantzig方法（原始），DSE方法（现有），PE方法（新）。核心逻辑一致，增加了partial scan提速，与论文一致。还引入了一些heuristic控制，比如先处理free nonbasic的变量（理论上这些应该一直在basic中，但由于不区分P1/P2，会来回出入）
  * dual col/chuzc:
        col0预处理：先筛选feasible slack；如果有free的变量，直接引入。
        4107 找对偶步长\theta（分两步，不断乘2，与K的论文中基本一致），增加控制流
        4469-4575 调整对偶的步长\theta，然后做shifting
        4576-4717 计算最优的pivot大小，帮助后续计算进行精度控制（相当于引入辅助信息，帮助控制流更好地进行跳转）
* SoPlex:
        spxsolver.h 主入口，给出各种定义
        pr后缀为pricing，rt后缀为ratio test
        区分rational和real，因为rational的计算精度实际要求更高
        流程步骤较多，可能是为了兼顾scip的IP流程，计算效率可能变差

        spxsolve.cpp -> solve 求解主流程
        spxpricer.h,spxratiotester.h
        enter/leave区分primal/dual，分别进enter.cpp和leave.cpp；dual simplex是先leave再enter。
        具体关注steeppr和boundflippingrt

## Misc

(19/07/02) 大数据的作用有以下几点：
1. 由于存储原始数据变得可行，可以对历史数据进行更多事后的多维度的分析，数据的利用率得到提升。
2. 具有多源数据和实时数据流的处理能力，意味着可以进行更实时的决策。
本质上，仍然依赖于良好的直觉和业务认知。

(19/12/21) (S)PDE的基本求解思路：
1. 对于空间问题，在值域逼近的话用有限差分(FDM)；在函数域逼近则用有限元(FEM)或者谱分解。
2. 如果有时间上的演化，则可将实数域上的PDE转化为函数域上(关于时间)的ODE。ODE中，每个无穷小时间片的演化对应一个函数域上的变换算子$A$，该算子在时间$t$的演化下组成算子半群$\\{e^{tA}\\}_t$。进而可以通过算子半群理论对算法的收敛性进行分析。
3. 对SPDE，需要掌握大规模随机采样的方式，特别是如何高效率对BM/brownian sheet进行采样。一种方式是通过circulant embedding+FFT进行精确采样。另一种方式是用Karhunen–Loève分解目标过程，然后分别采样来逼近。

(21/04/15) CTM模型：
* 基本思路：next cell inflow = min(demand = current cell density, supply1 = next cell flow capacity, supply2 = next cell jam density)
* jam density通过backward wave speed（对应两个不同密度的流体merge时，边界移动的速度）定义，可以任意指定，也可以根据上下游流密速情况推导
* 实践中，ctm可以很好地逼近lwr流体模型，并可进一步扩展到一般网络场景

(21/04/15) CAV+V2X套路整理：
1. 微观：CAV+V2X控制方案设计。重要综述： An overview of vehicular platoon control under the four-component framework。
   1. 如何设计更安全和鲁棒的“分布式”跟弛行为：考虑周边车辆的信息输入，以及行驶过程中的自然扰动。通常考虑控制论方法，并分析稳定性。 -- 思考题：如果是不同标准的CAV，应该如何进行交互？
   2. 如何设计更好的platoon控制方法：取决于路侧单元提供哪种形式的控制信号。可以用传统的控制方法做，也可以考虑RL。同样，稳定性和鲁棒性很重要。
   3. 如何设计更高效的CAV信息交互方案：information flow topology（IFT）设计。是一个MIP问题，但是可以根据问题结构设计高效的启发式算法。
   4. 如何设计可实时部署的控制策略：预留计算时间，提前进行计算，然后在实际控制时，根据具体情况，对控制方案进行一阶修正。     
2. 宏观：基于CAV+V2X的混合交通流的控制

## Paper Notes

* 1912.05500: 通过extrinsic reward -> intrinsic reward -> policy, 能够更好地在非平稳环境中迭代策略
* 2011.06118: 在拟合choice行为时，通过局部扰动生成候选选项，而不是考虑全部选项，类似于在决策前增加一层attention
* 1908.09874: 对于类别特征的encoding，使用该类别的连续特征均值，或者给定联系特征、样本属于该类别的条件概率，相比于使用one-hot要更高效。
* 2006.13570: 通过对不同的超参数（学习率、dropout率等）进行smart的deep-ensemble，可以得到hyper-deep-ensemble，进而增加预测的diversity。问题：这些工作是否真正地捕捉了模型的uncertainty？
* 2006.08063: 在VAE等概率建模中，对于普通离散分布，一种重参数近似方式是gumbel-softmax；这一工作将该方法延拓到对复杂组合对象的分布上。具体来说是先仿照gumbel-max构建优化问题，然后考虑convex relaxation做近似。
* 2010.06610: 文章指出，直接multi-head很难产生具有差异化的预测结果。但是为什么MIMO就能work？
* [preference learning along multiple criteria](https://papers.nips.cc/paper/2020/file/52f4691a4de70b3c441bca6c546979d9-Paper.pdf): 很多决策问题，有多个维度的偏好，此时优化问题可以formulate成一个approachbility的问题，min max与target set之间的距离；这一问题是凸、但非线性的，因此和一维偏好上的min max有一定差异。
* 2004.00603: CFR中的action regret推广为policy regret（称为“trigger regret”），然后再通过localize转化，形成可以局部计算的regret，进而产生相关算法。有点像在原版CFR上做的理论改进。
* 2007.05864: NTKGP extends random prior with NTK info, better than random prior.
* 2102.00815: 提出Bellman-Eluder dimension，作为一种结合Bellman Rank和Eluder dimension的用于评估复杂度的指标。
* 2102.09488: 在information-directed sampling中，分析和比较了两种不同entropy定义所带来的影响，并根据数值实验结果，提出了一种新的regret bound理论分析框架。
* 2102.12769: 考察了长尾reward情况下的RL理论，提出使用robust mean estimator（包括reward clipping）替代sample mean。
* 2102.08087: 考虑连续时间optimal task acceptance问题，其中task的时长和收益未知，需要探索；作者先分析完全信息情况下的最优策略，然后基于此提出了数种predict-then-optimize算法，并分析了相应的regret bound。
* 2103.04289: IRL with ql-k model，即对于任意reward函数，从myopic开始，多次根据opponent的当前最优策略更新自身最优策略（k层深度的优化）；然后根据相应的loss更新reward函数
* 2110.13973: （平衡对环境复杂度的理解程度和策略的优化效果）在信息传输带宽有限的情况下，通过Blahut-Arimoto等信息论算法对未知环境的信息进行压缩，找到有限带宽下最值得关注的优化方向，能够提升单位信息密度的增量收益。
* 2112.06054: 提出一种IRL方法，通过policy和state occupancy dist之间的联系，将学习目标转为expert state occupancy dist，而这可以通过td的方式实现。该方法的好处是不需要stationary的expert数据，要求更低（意味着可以在arbitrary init state dist下获取expert response）。
