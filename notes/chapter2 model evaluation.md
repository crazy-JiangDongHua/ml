# 第二章 模型评估

## 1、评估指标的局限性

1. 准确率(Accuracy)：分类正确的样本占总样本个数的比例。

   当不同类别的**样本比例非常不均衡**时，将准确率作为分类性能的指标非常局限，可以使用更加有效的**平均准确率**(每个类别下的样本准确率的算数平均)作为模型评估的指标。

2. 精确率(Precision)：分类正确的正样本个数占分类器判定为正样本个数的比例。

3. 召回率(Recall)：分类正确的正样本个数占真正的正样本个数的比例。

4. **排序问题**中，通常没有一个确定的阈值把得到的结果判定为正样本或负样本，而是采用Top N返回结果(即模型判定的N个正样本)计算Precision和Recall来衡量排序模型的性能。

5. Precision和Recall是即矛盾又统一的两个指标。通常要绘制P-R曲线，单个点对应的精确率和召回率并不能全面地衡量模型的性能，画曲线能对模型进行更为全面的评估。

6. F1值和ROC曲线也能综合反映一个排序模型的性能。ROC曲线后面一节再说，F1值是精确率和召回率的**调和均值**。
   $$
   F1=\frac{2\times precision\times recall}{precision+recall}
   $$

7. 均方根误差RMSE(Root Mean Square Error)通常用来衡量回归模型的好坏，但是如果存在个别偏离程度特别大的**离群点**(Outlier)，即使离群点非常少，也会让RMSE指标变的很差。
   $$
   RMSE=\sqrt{\frac{\sum_{i=1}^{n}(y_i-{\hat{y}_i})^2}{n}}
   $$
   例如在流量预测问题中，噪声点是很容易产生的，甚至一些相关社交媒体突发事件带来的流量，都很有可能造成离群点。
    针对离群点的解决方案有：

   - 如果认为这些离群点是“噪声点”的话，就需要在数据预处理的阶段将噪声点**过滤**掉。

   - 如果不认为离群点是“噪声点”，就需要进一步提高模型的预测能力，将**离群点产生的机制建模进去**。(这就是一个宏大的话题了)

   - 找一个更合适的指标来评估模型，如**平均绝对百分比误差**(Mean Absolute Percent Error，MAPE)。它将每个点的误差进行了归一化，降低了个别离群点带来的绝对误差的影响。
     $$
     MAPE=\sum_{i=1}^{n}|\frac{y_i-\hat{y}_i}{y_i}|\times\frac{100}{n}
     $$

## 2、ROC曲线

1. ROC(Receiver Operating Characteristic Curve，受试者工作特征曲线)，横坐标为**假阳性率**(False Positive Rate，FPR)，纵坐标为**真阳性率**(True Positive Rate，TPR)。
   $$
   FPR=\frac{FP}{N},TPR=\frac{TP}{P}
   $$
   其中N为真实的负样本个数，P为真实的正样本个数。多个类的混淆矩阵如下图所示。
   
   ![混淆矩阵](C:\Users\Lenovo\Desktop\ml\notes\img\2-1.png)
   
2. ROC曲线的绘制：二值分类问题中，模型的输出一般都是预测样本为正例的概率，概率大于该值则判为正例，小于该值判为负例，计算FPR和TPR，形成ROC曲线上的一点。通过不断移动截断点，则可绘制出ROC曲线。
   其实，还有一种更直观地绘制ROC曲线的方法。首先，根据样本标签统计出正负样本的数量，假设正样本数量为P，负样本数量为N；接下来，把横轴的刻度间隔设置为1/N，纵轴的刻度间隔设置为1/P；再根据模型输出的预测概率对样本进行排序（从高到低）；依次遍历样本，同时从零点开始绘制ROC曲线，每遇到一个正样本就沿纵轴方向绘制一个刻度间隔的曲线，每遇到一个负样本就沿横轴方向绘制一个刻度间隔的曲线，直到遍历完所有样本，曲线最终停在（1,1）这个点，整个ROC曲线绘制完成。

3. AUC：ROC曲线下的面积(沿横轴积分)，能够量化地反映基于ROC曲线衡量出的模型性能。
   
4. ROC曲线与P-R曲线：当正负样本的分布发生变化时，ROC曲线形状能**保持基本不变**，而P-R曲线形状一般会**发生剧烈的变化**。ROC曲线能更加稳定地反映模型本身的好坏，广泛应用于排序、推荐、广告等领域。如果希望更多地看到模型在**特定数据集**上的表现，P-R曲线能更直观地反映其性能。

## 3、余弦距离的应用

1. **余弦相似度**和**余弦距离**：余弦相似度取值范围为$[-1,1]$，余弦距离是1减余弦相似度，取值范围为$[0,2]$。

2. **余弦距离**和**欧式距离**：

   - 余弦距离关心的是向量的**角度关系**，并不关心它们的**绝对大小**。在文本、图像、视频领域，特征维度往往很高，余弦相似度在高维情况下依然能保持“相同为1，正交为0，相反为-1”。而欧式距离**受维度影响**，范围不固定，含义也比较模糊。

   - 在一些场景，欧式距离和余弦距离有着**单调**的关系，如果选择距离最小(相似度最大)的近邻，则使用欧式距离和余弦距离的结果是相同的。

   - 总体来说，欧式距离体现**数值上的绝对差异**，余弦距离体现**方向上的相对差异**。
     例如统计用户看剧行为，A为(0,1)， B为(1,0)，此时余弦距离较大，欧式距离较小，我们分析两个用户对不同视频的喜好，更关注相对差异，这里应该用余弦距离。
      再例如分析用户活跃度，以登陆次数和平均观看时长为特征，A为(1,10)， B为(10,100)，则余弦距离会认为他们非常近，但这两个用户的活跃度差别是非常大的，应该选用欧式距离。

3. **余弦距离并不是一个严格定义的距离**。距离的定义是在一个集合中，每一对元素均可唯一确定一个实数，使得**正定性**、**对称性**、**三角不等式**成立，则该实数可称为这对元素的距离。余弦距离满足正定性和对称性，但**不满足三角不等式**。
   在机器学习领域，被称为距离但不满足三条距离公理的还有**KL距离**(Kullback-Leibler Divergence)，也叫做相对熵，常用于计算两个分布之间的差异，它不满足对称性和三角不等式。

## 4、A/B测试的陷阱

1. 在机器学习中A/B测试是验证模型最终效果的主要手段。

2. 已经对模型进行充分的离线评估，还需要进行在线A/B测试的原因有：

   - 离线评估无法完全消除模型**过拟合**的影响。

   - 离线评估无法完全还原线上的工程环境。一般来讲，离线评估往往没有考虑线上环境的延迟、数据丢失、标签数据缺失等情况。也就是说，离线评估是**理想工程环境**下的结果。

   - 离线评估一般是针对模型本身进行评估，线上系统的某些**商业指标**在离线评估中无法计算。如推荐问题中，离线评估关注ROC曲线、P-R曲线，而弦上评估可以全面了解用户点击率、留存时长、PV访问量等变化。

3. 如何进行线上A/B测试：主要手段是进行**用户分桶**，将用户分成实验组和对照组，对实验组用户用新模型，对对照组用户用旧模型。分桶要保证样本的**独立性**和采样方式的**无偏性**。

## 5、模型评估的方法

1. **Holdout检验**：最简单直接的检验方法，它将原始样本数据集**随机划分**成训练集和测试集。
   缺点：验证集上计算出来的最后评估指标原始分组有很大关系。

2. 交叉检验：为了消除Holdout的随机性，则有了交叉验证。

   - **k-fold交叉验证**：将全部样本分成k个大小相等的样本子集，拿出其中一个子集作为验证集，其余k-1个子集作为训练集，依次遍历。通常把k次评估指标的平均值作为最终的评估指标。实际试验中，k经常取10。

   - **留一验证**：每次留下一个样本作为验证集，其余所有样本作为训练集，进行n次验证，再将评估指标求平均值得到最终的评估指标。(n个值的平均)
     留一验证缺点：在样本总数较多的情况下，时间开销极大。留一验证是留p验证的特例，但从n个元素中选择p个元素有$C_n^p$种可能，时间开销比留一验证更多，在实际工程很少应用。

3. 自助法：Holdout和交叉验证都是基于数据集的划分，但是当样本规模较小时，将样本集进行划分会让训练集进一步减小，可能影响模型的训练效果。自助法是基于**自助采样**的检验方法，对于总数为$n$的样本集合，**有放回**地随机抽样$n$次，得到大小为$n$的训练集。其中有的样本会重复，有的样本没有被抽出过，将没有被抽出的样本作为验证集进行验证，就是自助法的验证过程。当样本数很大时，大约有$1/e=36.8\%$的样本从来没有被采样过，这里的计算用到高数中的重要极限。

## 6、超参数调优

1. 超参数搜索算法包括的几个要素：

   - 目标函数，即算法需要最大化/最小化的目标。

   - 搜索范围，一般通过上限和下限来确定。

   - 算法的其他参数，如搜索步长。

2. 超参数调优方法有：

   - **网格搜索**：最简单、应用最广泛的超参数搜索算法。通过查找搜索范围内所有的点来确定最优值。
     优点：如果采用较大的搜索范围以及较小的步长，则很大概率能找到**全局最优值**。
     缺点：十分**消耗资源和时间**，特别是需要调优的超参数比较多的时候。
     实际应用中会先使用较大的搜索范围和较大的搜索步长，寻找全局最优值可能的位置，然后逐渐缩小搜索范围和步长来寻找更精确的最优值。优点是可以降低所需时间和计算量，缺点是目标函数一般是**非凸**的，可能会错过全局最优值。

   - **随机搜索**：随机搜索不再测试上界和下界之间的所有值，而是在搜索范围内**随机选取**样本点。其理论依据是，如果样本足够多，那么随机采样也能大概率找到全局最优值，或其近似值。
     优点：比网格搜索快。
     缺点：并不能保证找到全局最优值。
     注：随机搜索有非常多种优化方法，如爬山算法、模拟退火(Simulated Annealing，SA)、遗传算法(Genetic Algorithm，GA)等。

3. **贝叶斯优化算法**：
   网格搜索和随机搜索在测试新点时会忽略前一个点的信息，而贝叶斯优化算法**充分利用了之前的信息**，通过对目标函数形状进行学习，找到使目标函数向全局最优值提升的参数。
   贝叶斯优化算法学习目标函数形状的方法：首先，根据先验分布，假设一个**搜集函数**；然后，每一次使用新的采样点来测试目标函数时，利用这个信息来更新目标函数的**先验分布**；最后，算法测试由**后验分布**给出全局最优值可能出现的位置。
   优点：比网格搜索、随机搜索更**高效**。
   缺点：贝叶斯优化算法一旦找到了一个局部最优值，会在该区域不断采样，很**容易陷入局部最优**。
   为了弥补贝叶斯优化的的缺陷，需要在搜索和利用之间找到一个**平衡**点，“搜索”即是在未取样的区域获取采样点，“利用”则是根据后验分布在最可能出现全局最优值的区域进行采样。

## 7、过拟合与欠拟合

1. **过拟合**：模型对训练数据拟合过当。如模型过于复杂，把**噪声**数据的特征也学习到模型中，导致模型泛化能力下降。反映到评估指标上，就是模型在训练集上表现很好，但测试集和新数据上表现较差。

2. **欠拟合**：模型没有很好地捕捉到数据的特征，不能很好地拟合数据。反映到评估指标上，就是模型训练和预测表现都不好。

3. 过拟合的原因：

   * 数据太少或抽样错误，导致训练数据不能有效代表业务场景的真实分布
   * 模型过于复杂，拟合了噪声的特征

4. 降低过拟合风险的方法：

   - 从数据入手：获得更多的训练数据，更多的训练数据是解决过拟合最有效的手段。一般实验数据有限，可以通过一定的规则来**扩充**训练集，如图像分类中可以用数据增强，用GAN来合成大量的新训练数据。

   - 降低模型复杂度。数据较少时，模型过于复杂是产生过拟合的主要因素，适当降低模型复杂度可以避免模型拟合过多的采样噪声。例如神经网络模型中**减少网络层数**、**神经元个数**等；决策树模型中降低树的**深度**、进行**剪枝**等。
   - 减少特征数量，删去组合特征

   - 正则化方法。给模型的参数加上一定的**正则约束**，例如将权值的大小加入到损失函数中，如L1/L2正则化。

   - 集成学习方法。将多个模型集成在一起，来降低单一模型的过拟合风险，如Bagging方法。
   - 早停
   - dropout
   - batch normalization

1. 降低欠拟合风险的方法：

   - **添加新特征**。当特征不足或者现有特征与样本标签的相关性不强时，模型容易出现欠拟合。通过挖掘“上下文特征”“ID类特征”“组合特征”等新特征，往往能取得更好的效果。在深度学习潮流中，由很多模型可以帮助完成特征工程，如**因子分解机**、**梯度提升决策树**、**Deep-crossing**等都可以成为丰富特征的方法。

   - 增加模型复杂度。例如在线性模型中**添加高次项**，在神经网络模型中增加网络层数或神经元个数等。

   - 减小正则化系数。正则化是用来防止过拟合的，但当模型出现欠拟合现象时，需要有针对性地减少正则化系数。