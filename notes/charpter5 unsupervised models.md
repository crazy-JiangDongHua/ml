# 第五章 非监督学习

## 引导语

非监督学习期望机器通过学习找到数据中存在的某种**共性**特征或结构，亦或是数据之前存在的某种**关联**。
 非监督学习主要包含两大类学习方法：**数据聚类**和**特征变量关联**。其中，聚类算法往往通过多次迭代来找到数据的**最优分割**，而特征变量关联则是利用各种**相关性分析**方法来找到变量之间的**关系**。

## 1、K均值聚类

1. K均值聚类的基本思想是，通过迭代方式寻找$K$个簇(Cluster)的一种划分方案，使得聚类结果对应的代价函数最小。特别的，代价函数可以定义为各个样本距离所属簇中心点的**误差平方和**$J(c,\mu)=\sum_{i=1}^{M}||x_i-\mu_{c_i}||^2$其中$x_i$代表第$i$个样本，$c_i$是$x_i$所属于的簇，$\mu_{c_i}$代表簇对应的中心点，$M$是样本总数。

2. K均值聚类算法步骤：

   1. 数据**预处理**，如归一化、离群点处理等。(不归一化会影响误差平方和的计算，K均值对离群点敏感，影响簇中心点的计算)

   2. **随机**选取$K$个簇中心，记为$\mu_1^{(0)},\mu_2^{(0)},...,\mu_K^{(0)}$(初始化簇中心)。

   3. 定义代价函数：$J(c,\mu)=\min_\mu\min_c\sum_{i=1}^{M}||x_i-\mu_{c_i}||^2$

   4. 令$t=0,1,2,...$为迭代步数，**重复下列过程**直到$J$收敛。

      1. 对于每一个样本$x_i$，将其分配到距离最近的簇：$c_i^{(t)}\leftarrow\mathop{\arg\min}_{k}||x_i-\mu_k^{(t)}||^2$对

      2. 于每一个类簇$k$，重新计算该簇的中心：$\mu_k^{(t+1)}\leftarrow\mathop{\arg\min}_\mu\sum_{i:c_i^{(t)}=k}||x_i-\mu||^2$
      
   
   K均值在迭代时，假设当前代价函数没有达到最小值，则首先固定簇中心(最开始是随便选择$K$个中心)，调整每个样例数据所属的类别，选择最近的中心点，使代价函数减小；然后定下当前每个样例数据的类别后，调整簇中心使代价函数减小。交替进行，代价函数单调递减，最后中心点和样例数据所属的类别都收敛。

3. K均值的优缺点：

   - 优点：对于大数据集，K均值高效且可伸缩，它的复杂度是$O(NKt)$，接近于**线性**。其中$N$是数据对象的数目，$K$是聚类簇数，$t$是迭代轮数。

   - 缺点：
      - 需要人工预设K值，且该值和真实数据分布未必吻合；
      - 受**初值**和**离群点**的影响，每次的**结果不稳定**；
      - 受初值影响，结果通常是**局部最优**；
      - 无法很好地解决数据簇**分布差别比较大的情况**(如一类是另一类样本数量的100倍)；
      - 不太适用于**离散分布**；
      - 样本点只能被**划分到单一的类中**。


4. K均值算法调优的方法：

   - **预处理**，数据归一化和离群点处理：K均值本质是一种基于欧式距离度量的数据划分方法，均值和方差大的维度将对数据聚类结果产生决定性影响。离群点或少量噪声会对均值产生影响导致中心偏移。

   - **合理选择K值**：K值是事先设置的，这是一个主要缺点。K值的选择一般基于经验和多次试验结果。
      * 可以采用**手肘法**，尝试不同K值对应的损失，画出折线图，其拐点就是手肘法认为的K的最佳值。
      * 手肘法是一个经验方法，不够自动化，有一个**Gap Statistic**方法，不需要肉眼判断，只需要找到最大的Gap Statistic对应的K即可。(Gap Statistic书中有具体介绍)
      * 采用**核函数**：面对**非凸**的数据分布形状时，可能需要引入核函数来优化，成为核K均值算法，是核聚类方法的一种。(通过非线性映射讲输入空间数据点映射到高维的特征空间中进行聚类)


5. K均值算法的改进模型：

   - K-means++：K-means最开始是随机选取数据集中的K个点作为聚类中心，K-means++**改进了初始值的选择**，会尽量使聚类中心越远越好。

   - ISODATA算法(迭代自组织数据分析法)：当遇到高维度、海量数据时，很难准确估计K的大小。其基本思想是当某个类别的样本数过少时，去除该类别；当某个类别的样本过多、分散程度较大时，分为两个子类别(加入了**分裂操作**和**合并操作**)。其缺点是需要指定的**参数比较多**，除了一个参考的$K$，还有三个阈值用于分裂合并的判断。

   - 为解决样本只能属于一个类，可以使用**模糊C**或**高斯混合**模型。


6. K均值的收敛性：K均值的迭代算法实际上是一种EM算法(Expectation-Maximization algorithm)。EM算法只能保证收敛到**局部最优解**。K均值算法等价于用EM算法求解以下含隐变量的最大似然问题：
   $$
   P\left(x, z \mid \mu_{1}, \mu_{2}, \ldots, \mu_{k}\right) \propto\left\{\begin{array}{c}
   \exp \left(-\left\|x-\mu_{z}\right\|_{2}^{2}\right),\left\|x-\mu_{z}\right\|_{2}=\min _{k}\left\|x-\mu_{k}\right\|_{2} ; \\
   0 \quad,\left\|x-\mu_{z}\right\|_{2}>\min _{k}\left\|x-\mu_{k}\right\|_{2}
   \end{array}\right.
   $$
   其中$z \in \{1,2,3,...,k\}$是模型的隐变量。

   在E步骤，计算：
   $$
   Q_{i}\left(z^{(i)}\right)=P\left(z^{(i)} \mid x^{(i)}, \mu_{1}, \mu_{2}, \ldots, \mu_{k}\right) \propto\left\{\begin{array}{l}
   1,\left\|x^{(i)}-\mu_{z^{(i)}}\right\|_{2}=\min _{k}\left\|x-\mu_{k}\right\|_{2} \\
   0,\left\|x^{(i)}-\mu_{z^{(i)}}\right\|_{2}>\min _{k}\left\|x-\mu_{k}\right\|_{2}
   \end{array}\right.
   $$
   这等同于在K均值算法中对于每一个点$x^{(i)}$找到当前最近的簇$z^{(i)}$。

   在M步骤，找到最优的参数$\theta\{\mu_1,\mu_2,...,\mu_k\}$，使得$Q$函数最大：
   $$
   \theta=\underset{\theta}{\operatorname{argmax}} \sum_{i=1}^{m} \sum_{z^{(i)}} Q_{i}\left(z^{(i)}\right) \log {P\left(x^{(i)}, z^{(i)} \mid \theta\right)}
   $$
   经过推导可得:
   $$
   \sum_{i=1}^{m} \sum_{(i)} Q_{i}\left(z^{(i)}\right) \log \frac{P\left(x^{(i)}, z^{(i)} \mid \theta\right)}{Q_{i}\left(z^{(i)}\right)}=\sum_{i=1}^{m}\left\|x^{(i)}-\mu_{z^{(i)}}\right\|^{2}
   $$
   因此，这一步骤等同于找到最优的中心点$\mu_1,\mu_2,...,\mu_k$，使得损失函数达到最小，此时每个样本$x^{(i)}$对应的簇$z^{(i)}$已确定，因此每个簇k对应的最优中心点$\mu_k$可以由该簇中所有点的平均计算得到，这与K均值算法中根据当前簇的分配更新聚类中心的步骤是等同的。

   ## 2、EM算法

   假如我们有一个估计问题（estimation problem），其中由训练样本集 $\{x^{(1)}, ..., x^{(m)}\}$ 包含了 $m$ 个独立样本。我们用模型 $p(x, z)$ 对数据进行建模，拟合其参数（parameters），其中的似然函数（likelihood）如下所示：

   $$
   \begin{aligned}
   l(\theta) &= \sum_{i=1}^m\log p(x;\theta) \\
   &= \sum_{i=1}^m\log\sum_z p(x,z;\theta)
   \end{aligned}
   $$
   
   然而，确切地找到对参数 $\theta$ 的最大似然估计（maximum likelihood estimates）可能会很难。此处的 $z^{(i)}$ 是一个潜在的随机变量（latent random variables）；通常情况下，如果 $z^{(i)}$ 事先得到了，然后再进行最大似然估计，就容易多了。
   
   这种环境下，使用期望最大化算法（EM algorithm）就能很有效地实现最大似然估计（maximum likelihood estimation）。明确地对似然函数$l(\theta)$进行最大化可能是很困难的，所以我们的策略就是使用一种替代，在 $E$ 步骤 构建一个 $l$ 的下限（lower-bound），然后在 $M$ 步骤 对这个下限进行优化。
   
   对于每个 $i$，设 $Q_i$ 是某个对 $z$ 的分布（$\sum_z Q_i(z) = 1, Q_i(z)\ge 0$）。则有下列各式$^1$：
   
   $$
   \begin{aligned}
   \sum_i\log p(x^{(i)};\theta) &= \sum_i\log\sum_{z^{(i)}}p(x^{(i)},z^{(i)};\theta)&(1) \\
   &= \sum_i\log\sum_{z^{(i)}}Q_i(z^{(i)})\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})} &(2)\\
   &\ge \sum_i\sum_{z^{(i)}}Q_i(z^{(i)})\log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}&(3) 
   \end{aligned}
   $$
   
   >1 如果 $z$ 是连续的，那么 $Q_i$ 就是一个密度函数（density），上面讨论中提到的对 $z$ 的求和（summations）就要用对 $z$ 的积分（integral）来替代。
   
   上面推导（derivation）的最后一步使用了 Jensen 不等式（Jensen’s inequality）。其中的 $f(x) = log x$ 是一个凹函数（concave function），因为其二阶导数 $f''(x) = -1/x^2 < 0$ 在整个定义域（domain） $x\in R^+$ 上都成立。
   
   此外，上式的求和中的单项： 
   
   $$
   \sum_{z^{(i)}}Q_i(z^{(i)})[\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}]
   $$
   
   是变量（quantity）$[p(x^{(i)}, z^{(i)}; \theta)/Q_i(z^{(i)})]$ 基于 $z^{(i)}$ 的期望，其中 $z^{(i)}$ 是根据 $Q_i$ 给定的分布确定。然后利用 Jensen 不等式（Jensen’s inequality），就得到了：
   
   $$
   f(E_{z^{(i)}\sim Q_i}[\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}])\ge
   E_{z^{(i)}\sim Q_i}[f(\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})})]
   $$
   
   其中上面的角标 $“z^{(i)}\sim Q_i”$ 就表明这个期望是对于依据分布 $Q_i$ 来确定的 $z^{(i)}$ 的。这样就可以从等式 $(2)$ 推导出等式 $(3)$。
   
   接下来，对于任意的一个分布 $Q_i$，上面的等式 $(3)$ 就给出了似然函数 $l(\theta)$ 的下限（lower-bound）。那么对于 $Q_i$ 有很多种选择。咱们该选哪个呢？如果我们对参数 $\theta$ 有某种当前的估计，很自然就可以设置这个下限为 $\theta$ 这个值。也就是，针对当前的 $\theta$ 值，我们令上面的不等式中的符号为等号。（稍后我们能看到，这样就能证明，随着 $EM$迭代过程的进行，似然函数 $l(\theta)$ 就会单调递增（increases monotonically）。）
   
   为了让上面的限定值（bound）与 $\theta$ 特定值（particular value）联系紧密（tight），我们需要上面推导过程中的 Jensen 不等式这一步中等号成立。要让这个条件成立，我们只需确保是在对一个常数值随机变量（“constant”-valued random variable）求期望。也就是需要：
   
   $$
   \frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}=c
   $$
   
   其中常数 $c$ 不依赖 $z^{(i)}$。要实现这一条件，只需满足：
   
   $$
   Q_i(z^{(i)})\propto p(x^{(i)},z^{(i)};\theta)
   $$
   
   实际上，由于我们已知 $\sum_z Q_i(z^{(i)}) = 1$（因为这是一个分布），这就进一步表明：
   
   $$
   \begin{aligned}
   Q_i(z^{(i)}) &= \frac{p(x^{(i)},z^{(i)};\theta)}{\sum_z p(x^{(i)},z;\theta)} \\
   &= \frac{p(x^{(i)},z^{(i)};\theta)}{p(x^{(i)};\theta)} \\
   &= p(z^{(i)}|x^{(i)};\theta)
   \end{aligned}
   $$
   
   因此，在给定 $x^{(i)}$ 和参数 $\theta$ 的设置下，我们可以简单地把 $Q_i$ 设置为 $z^{(i)}$ 的后验分布（posterior distribution）。
   
   接下来，对 $Q_i$ 的选择，等式 $(3)$ 就给出了似然函数对数（log likelihood）的一个下限，而似然函数（likelihood）正是我们要试图求最大值（maximize）的。这就是 $E$ 步骤。接下来在算法的 $M$ 步骤中，就最大化等式 $(3)$ 当中的方程，然后得到新的参数 $\theta$。重复这两个步骤，就是完整的 $EM$ 算法，如下所示：
   
   &emsp;重复下列过程直到收敛（convergence）:  {
   
   &emsp;&emsp;（$E$ 步骤）对每个 $i$，设 
   　　
   $$
   Q_i(z^{(i)}):=p(z^{(i)}|x^{(i)};\theta)
   $$
   
   &emsp;&emsp;（$M$ 步骤） 设 
   
   $$
   \theta := arg\max_\theta\sum_i\sum_{z^{(i)}}Q_i(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
   $$
   
   } 
   
   怎么才能知道这个算法是否会收敛（converge）呢？设 $\theta^{(t)}$ 和 $\theta^{(t+1)}$ 是上面 $EM$ 迭代过程中的某两个参数（parameters）。接下来我们就要证明一下 $l(\theta^{(t)})\le l(\theta^{(t+1)})$，这就表明 $EM$ 迭代过程总是让似然函数对数（log-likelihood）单调递增（monotonically improves）。证明这个结论的关键就在于对 $Q_i$ 的选择中。在上面$EM$迭代中，参数的起点设为 $\theta^{(t)}$，我们就可以选择 $Q_i^{(t)}(z^{(i)}): = p(z^{(i)}|x^{(i)};\theta^{(t)})$。之前我们已经看到了，正如等式 $(3)$ 的推导过程中所示，这样选择能保证 Jensen 不等式的等号成立，因此：
   
   $$
   l(\theta^{(t)})=\sum_i\sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta^{(t)})}{Q_i^{(t)}(z^{(i)})}
   $$
   
   参数 $\theta^{(t+1)}$  可以通过对上面等式中等号右侧进行最大化而得到。因此： 
   
   $$
   \begin{aligned}
   l(\theta^{(t+1)}) &\ge \sum_i\sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta^{(t+1)})}{Q_i^{(t)}(z^{(i)})} &(4)\\
   &\ge \sum_i\sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta^{(t)})}{Q_i^{(t)}(z^{(i)})} &(5)\\
   &= l(\theta^{(t)}) &(6)
   \end{aligned}
   $$
   
   上面的第一个不等式推自：
   
   $$
   l(\theta)\ge \sum_i\sum_{z^{(i)}}Q_i(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
   $$
   
   上面这个不等式对于任意值的 $Q_i$ 和 $\theta$ 都成立，尤其当 $Q_i = Q_i^{(t)}, \theta = \theta^{(t+1)}$。要得到等式 $(5)$，我们要利用 $\theta^{(t+1)}$ 的选择能够保证：
   
   $$
   arg\max_\theta \sum_i\sum_{z^{(i)}}Q_i(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
   $$
   
   这个式子对 $\theta^{(t+1)}$ 得到的值必须大于等于 $\theta^{(t)}$ 得到的值。最后，推导等式$(6)$ 的这一步，正如之前所示，因为在选择的时候我们选的 $Q_i^{(t)}$ 就是要保证 Jensen 不等式对 $\theta^{(t)}$ 等号成立。
   
   因此，$EM$ 算法就能够导致似然函数（likelihood）的单调收敛（单调有界必收敛）。在我们推导 $EM$ 算法的过程中，我们要一直运行该算法到收敛。得到了上面的结果之后，可以使用一个合理的收敛检测（reasonable convergence test）来检查在成功的迭代（successive iterations）之间的 $l(\theta)$ 的增长是否小于某些容忍参数（tolerance parameter），如果 $EM$ 算法对 $l(\theta)$ 的增大速度很慢，就声明收敛（declare convergence）。
   
   **备注。** 如果我们定义
   
   $$
   J(Q, \theta)=\sum_i\sum_{z^{(i)}}Q_i(z^{(i)})log\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
   $$
   
   通过我们之前的推导，就能知道 $l(\theta) ≥ J(Q, \theta)$。这样 $EM$ 算法也可看作是在 $J$ 上的坐标上升（coordinate ascent），其中 $E$ 步骤在 $Q$ 上对 $J$ 进行了最大化（自己检查哈），然后 $M$ 步骤则在 $\theta$ 上对 $J$ 进行最大化。
   
   在一般条件下EM算法是收敛的，但不能保证收敛到全局最优。
   
   EM算法应用极其广泛，主要应用于含有隐变量的概率模型的学习。高斯混合模型的参数估计是EM算法的一个重要应用。
   
   EM算法还可以解释为F函数的极大-极大算法。EM算法有许多变形，如GEM算法。GEM算法的特点是每次迭代增加F函数值（并不一定是极大化F函数），从而增加似然函数值。
   
   ## 3、高斯混合模型
   
   1. 高斯混合模型(Gaussian Mixed Model，GMM)也是一种常见的聚类算法，与K-means类似，都使用了EM算法进行迭代计算。高斯混合模型假设每个簇的数据都是符合高斯分布的，当前数据呈现的分布就是各个簇的高斯分布叠加在一起的结果。理论上，高斯混合模型可以**拟合出任意类型的分布**。
   
   2. 高斯混合模型的核心思想是，假设数据可以看作从多个高斯分布中生成出来的。在该假设下，每个单独的分模型都是标准高斯模型，其**均值**$\mu_i$和**方差**$\Sigma_i$，此外，每个分模型都还有一个参数$\pi_i$，可以理解为**权重或生成数据的概率**。高斯混合模型公式如下：$p(x)=\sum_{i=1}^K\phi_i N(x|\mu_i,\Sigma_i)$，其中$\phi_j \ge 0, \sum_{j=1}^K \phi_j=1$，$K$是高斯模型的个数。
   
   3. 高斯混合模型是一个**生成式模型**。
   
   4. 求解高斯混合模型的参数可以用EM算法框架。
   
      我们希望能够获得一个联合分布 $p(x^{(i)},z^{(i)}) = p(x^{(i)}|z^{(i)})p(z^{(i)})$ 来对数据进行建模。其中的 $z^{(i)} \sim Multinomial(\phi)$ （即$z^{(i)}$ 是一个以 $\phi$ 为参数的多项式分布，其中 $\phi_j \ge 0, \sum_{j=1}^k \phi_j=1$，而参数 $\phi_j$ 给出了 $p(z^{(i)} = j)$），另外 $x^{(i)}|z^{(i)} = j \sim N(μ_j,\Sigma_j)$ **（$x^{(i)}|z^{(i)} = j$是一个以 $μ_j$ 和 $\Sigma_j$ 为参数的正态分布）**。我们设 $k$ 来表示 $z^{(i)}$ 能取的值的个数。因此，我们这个模型就是在假设每个$x^{(i)}$ 都是从$\{1, ..., k\}$中随机选取$z^{(i)}$来生成的，然后 $x^{(i)}$ 就是服从$k$个高斯分布中的一个，而这$k$个高斯分布又取决于$z^{(i)}$。这就叫做一个**混合高斯模型（mixture of Gaussians model）。** 此外还要注意的就是这里的 $z^{(i)}$ 是**潜在的**随机变量（latent random variables），这就意味着其取值可能还是隐藏的或者未被观测到的。这就会增加这个估计问题（estimation problem）的难度。
   
      我们这个模型的参数也就是 $\phi, \mu$ 和 $\Sigma$。要对这些值进行估计，我们可以写出数据的似然函数（likelihood）：
   
      $$
      \begin{aligned}
      l(\phi,\mu,\Sigma) &= \sum_{i=1}^m \log p(x^{(i)};\phi,\mu,\Sigma) \\
      &= \sum_{i=1}^m \log \sum_{z^{(i)}=1}^k p(x^{(i)}|z^{(i)};\mu,\Sigma)p(z^{(i)};\phi)
      \end{aligned}
      $$
   
      然而，如果我们用设上面方程的导数为零来尝试解各个参数，就会发现根本不可能以闭合形式（closed form）来找到这些参数的最大似然估计（maximum likelihood estimates）。（不信的话你自己试试咯。）
   
      随机变量 $z^{(i)}$表示着 $x^{(i)}$ 所属于的 $k$ 个高斯分布值。这里要注意，如果我们已知 $z^{(i)}$，这个最大似然估计问题就简单很多了。那么就可以把似然函数写成下面这种形式：
   
      $$
      l(\phi,\mu,\Sigma)=\sum_{i=1}^m \log p(x^{(i)}|z^{(i)};\mu,\Sigma) + \log p(z^{(i)};\phi)
      $$
   
      对上面的函数进行最大化，就能得到对应的参数$\phi, \mu$ 和 $\Sigma$：
   
      $$
      \begin{aligned}
      &\phi_j=\frac 1m\sum_{i=1}^m 1\{z^{(i)}=j\}, \\
      &\mu_j=\frac{\sum_{i=1}^m 1\{z^{(i)}=j\}x^{(i)}}{\sum_{i=1}^m 1\{z^{(i)}=j\}}, \\
      &\Sigma_j=\frac{\sum_{i=1}^m 1\{z^{(i)}=j\}(x^{(i)}-\mu_j)(x^{(i)}-\mu_j)^T}{\sum_{i=1}^m 1\{z^{(i)}=j\}}.
      \end{aligned}
      $$
   
      事实上，我们已经看到了，如果 $z^{(i)}$ 是已知的，那么这个最大似然估计就几乎等同于用高斯判别分析模型（Gaussian discriminant analysis model）中对参数进行的估计，唯一不同在于这里的 $z^{(i)}$ 扮演了高斯判别分析当中的分类标签$^1$的角色。
   
      >1 这里的式子和高斯判别分析的方程还有一些小的区别，这首先是因为在此处我们把 $z^{(i)}$ 泛化为多项式分布（multinomial），而不是伯努利分布（Bernoulli），其次是由于这里针对高斯分布中的每一项使用了一个不同的 $\Sigma_j$。
   
      然而，在密度估计问题里面，$z^{(i)}$ 是不知道的。这要怎么办呢？
      期望最大化算法（EM，Expectation-Maximization）是一个迭代算法，有两个主要的步骤。针对我们这个问题，在 $E$ 这一步中，程序是试图去“猜测（guess）” $z^{(i)}$ 的值。然后在 $M$ 这一步，就根据上一步的猜测来对模型参数进行更新。由于在 $M$ 这一步当中我们假设（pretend）了上一步是对的，那么最大化的过程就简单了。下面是这个算法：
   
   
      &emsp;重复下列过程直到收敛（convergence）: {
   
      &emsp;&emsp;（$E$-步骤）对每个 $i, j$, 设 
   
      $$
      w_j^{(i)} := p(z^{(i)}=j|x^{(i)};\phi,\mu,\Sigma)
      $$
   
      &emsp;&emsp;（$M$-步骤）更新参数：
   
      $$
      \begin{aligned}
      &\phi_j=\frac 1m\sum_{i=1}^m w_j^{(i)}, \\
      &\mu_j=\frac{\sum_{i=1}^m w_j^{(i)}x^{(i)}}{\sum_{i=1}^m w_j^{(i)}}, \\
      &\Sigma_j=\frac{\sum_{i=1}^m w_j^{(i)}(x^{(i)}-\mu_j)(x^{(i)}-\mu_j)^T}{\sum_{i=1}^m w_j^{(i)}}.
      \end{aligned}
      $$
   
      } 
   
      在 $E$ 步骤中，在给定 $x^{(i)}$ 以及使用当前参数设置（current setting of our parameters）情况下，我们计算出了参数 $z^{(i)}$ 的后验概率（posterior probability）。使用贝叶斯规则（Bayes rule），就得到下面的式子：
   
      $$
      p(z^{(i)}=j|x^{(i)};\phi,\mu,\Sigma)=
      \frac{p(x^{(i)}|z^{(i)}=j;\mu,\Sigma)p(z^{(i)}=j;\phi)}
      {\sum_{l=1}^k p(x^{(i)}|z^{(i)}=l;\mu,\Sigma)p(z^{(i)}=l;\phi)}
      $$
   
      上面的式子中，$p(z^{(i)}=j|x^{(i)};\phi,\mu,\Sigma)$ 是通过评估一个高斯分布的密度得到的，这个高斯分布的均值为 $\mu_i$，对$x^{(i)}$的协方差为$\Sigma_j$；$p(z^{(i)} = j;\phi)$ 是通过 $\phi_j$ 得到，以此类推。在 $E$ 步骤中计算出来的 $w_j^{(i)}$ 代表了我们对 $z^{(i)}$ 这个值的“弱估计（soft guesses）”$^2$。
   
      >2 这里用的词汇“弱（soft）”是指我们对概率进行猜测，从 $[0, 1]$ 这样一个闭区间进行取值；而与之对应的“强（hard）”值得是单次最佳猜测，例如从集合 $\{0,1\}$ 或者 $\{1, ..., k\}$ 中取一个值。
   
      另外在 $M$ 步骤中进行的更新还要与 $z^{(i)}$ 已知之后的方程式进行对比。它们是相同的，不同之处只在于之前使用的指示函数（indicator functions），指示每个数据点所属的高斯分布，而这里换成了 $w_j^{(i)}$。
   
      $GMM$算法也让人想起 $K$ 均值聚类算法，而在 $K$ 均值聚类算法中对聚类重心 $c(i)$ 进行了“强（hard）”赋值，而在 $GMM$ 算法中，对$w_j^{(i)}$ 进行的是“弱（soft）”赋值。与 $K$ 均值算法类似，$GMM$ 算法也容易导致局部最优，所以使用不同的初始参数（initial parameters）进行重新初始化（reinitializing），可能是个好办法。
   
   5. 高斯混合模型与K均值算法：
      相同点：
   
      * 都是聚类算法；
   
      * 都需要指定K值；
   
      * 都用EM算法来求解；
   
      * 往往只能收敛于局部最优。
   
      高斯混合模型的优点：
      
      * 可以给出一个样本属于某类的概率是多少（样本可以属于多个类）；
      * 不仅用于聚类，还可以用于概率密度估计；
      * 可以用于生成新的样本点。
   
   ## 4、自组织映射神经网络
   
   1. 自组织映射神经网络(Self-Organizing Map，SOM)是无监督学习方法中的一类重要方法，可以用作**聚类**、**高维可视化**、**数据压缩**、**特征提取**等多种用途。
   
   2. 自组织神经网络本质上是一个两层的神经网络，包括输入层和输出层(竞争层)。输入层模拟感知外界输入信息的视网膜，输出层模拟做出相应的大脑皮层。输出层中神经网的个数通常是聚类的个数。
   
      训练时采用“竞争学习”的方式，每个输入样例在输出层中找到一个最匹配的节点，成为激活节点；紧接着用梯度下降法更新激活节点的参数；同时，激活节点临近的点也根据他们距离激活节点的远近适当地更新参数(模拟神经细胞的侧抑制现象，越远更新程度越打折扣，但更远的则表现弱激励作用)。这种竞争可以通过神经元之间的横向抑制连接(负反馈路径)来实现。
   
   3. 自组织映射神经网络的自组织学习过程可以归纳为以下几个子过程：
   
      1. 初始化：所有连接权重都用**小的随机值**初始化。
      2. 竞争：神经元计算每个输入模式各自的判别函数值，并宣布具有最小判别函数值的特定神经元为胜利者。
      3. 合作：获胜神经元决定了兴奋神经元拓扑领域的空间位置。确定激活节点后，更新节点，距离越远，更新程度越打折扣，但更远的则表现弱激励作用。
      4. 适应：适当调整相关兴奋神经元的连接权重，使得获胜神经元对相似输入模式的后续应用的响应增强。
      5. 迭代：回到(2)竞争步骤，迭代直到特征映射趋于稳定。
   
   4. 自组织映射神经网络具有**保序映射**的特点，可以将任意维度的输入在输出层映射为一维或二维图形，并保持拓扑结构不变。
   
   5. 自组织映射神经网络(SOM)与K-means：
   
      - SOM**受K值影响要小一点**，隐藏层中的某些节点可以没有任何输入数据属于它，因此聚类结果的簇数可能会小于神经元的个数。将K设大一点即可。
   
      - SOM为每个输入数据找到一个最相似的类后，也会更新临近的节点，而K-means只更新这个类的参数。这样做有优点也有缺点，优点是SOM**受噪声影响较小**，**缺点是准确性**可能比K-means低(因为也更新了临近节点)。
   
      - SOM的**可视化比较好**，而且具有优雅的**拓扑关系**图。
   
   1. 如何设计自组织映射神经网络并设定网络训练参数：
   
      - 设定输出层神经元的数量：如果不清楚类别数，可以尽量设置较多的节点数。如果分类过细则酌情减少输出节点。产生的从未更新过的“死节点”可以通过重新初始化权值来解决。
   
      - 设计输出层节点的排列：对于一般的分类问题，一个输出节点能代表一个模式类，用一维线阵既结构简单又意义明确。对于颜色空间或旅行路径类问题，用二维平面比较直观。
   
      - 初始化权重：可以随机初始化，但为避免大量初始“死节点”，可以从训练集中随机抽取![m](https://math.jianshu.com/math?formula=m)个输入样本作为初始权值。
   
      - 设计拓扑领域：领域形状可以是正方形、六边形或者菱形。
   
      - 设计学习率：一个递减函数。
   
   ## 5、聚类算法的评估
   
   1. 常见数据簇的特点：
   
      - 以**中心**定义的数据簇：这类数据集合倾向于球形分布，通常中心被定义为质心(所有点的平均值)。
   
      - 以**密度**定义的数据簇：这类数据集合呈现和周围数据簇明显不同的密度。当数据簇不规则或相互盘绕，并且有噪声和离群点时，常常基于密度的簇定义。
   
      - 以**连通**定义的数据簇：这类数据集合中的数据之间有连通关系，整个数据簇表现为图结构。对不规则形状或者缠绕的数据簇有效。
   
      - 以**概念**定义的数据簇：这类数据集合中具有某种共同性质。
   
   1. 聚类评估的任务是估计在数据集上进行聚类的可行性，以及聚类方法产生的结果的质量。这一过程分为三个子任务：
   
      - 估计聚类趋势：这一步骤是检测数据分布中是否存在**非随机**的簇结构。如果数据是基本随机的，那么聚类结果毫无意义。我们可以观察聚类误差是否随聚类类别数量的增加而单调变化，如果数据是基本随机的，即不存在非随机簇结构，那么聚类误差随聚类类别数量增加而变化的幅度应该不限制，并且找不到合适的K。还可以用**霍普金斯统计量**(Hopkins Statistic)来判断数据在空间上的随机性。
   
      - 判定数据簇数：找到与真实数据分布最吻合的簇数，据此判断聚类结果的质量。可手肘法或Gap Statistic等方法。需要说明是，用于评估的最佳数据簇数可能与程序输出的簇数是不同的。例如有些聚类算法可以自动确定K，但可能与我们通过其他方法得到的最优簇数有差别。
   
      - 测定聚类质量：在无监督的情况下，我们可以通过考察簇的分离情况和紧凑情况来评估，常用的度量指标有：
        * **轮廓系数**：与簇中数据紧凑程度和该簇与其他簇的分离程度有关。
        * **均方根标准偏差**(Root-mean-square standard deviation，RMSSTD)：衡量聚类结果的同质性，即紧凑程度。
        * **R方**(R-Square)：可以用来衡量聚类的差异度。**改进的Hubert$\Gamma$统计**：通过数据对的不一致性来评估聚类的差异。此
        * 外，为了更加合理地评估不同聚类算法的性能，通常还需要**人为构造**不同类型的数据集，观察聚类算法在这些数据集上的效果。
   
   