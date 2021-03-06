# 异常检测——基于相似度的方法

[本文github地址](https://github.com/ArriettySong/DataWhale_LearningNote/blob/main/AnomalyDetection/Task4-基于相似度的方法.md)

**主要内容包括：**

- 基于距离的度量
- 基于密度的度量

[TOC]

## 1、概述

​	“异常”的定义要结合业务背景来判定，“异常值”通常指具有特定业务意义的那一类特殊的异常值。

## 2、基于距离的度量 

​		基于距离的方法是一种常见的适用于各种数据域的异常检测算法，它基于最近邻距离来定义异常值。 此类方法不仅适用于多维数值数据，在其他许多领域，例如分类数据，文本数据，时间序列数据和序列数据等方面也有广泛的应用。

​		基于距离的异常检测有这样一个前提假设，即异常点的 $k$ 近邻距离要远大于正常点。解决问题的最简单方法是使用嵌套循环。 第一层循环遍历每个数据，第二层循环进行异常判断，需要计算当前点与其他点的距离，一旦已识别出多于 $k$ 个数据点与当前点的距离在 $D$ 之内，则将该点自动标记为非异常值。 这样计算的时间复杂度为$O(N^{2})$，当数据量比较大时，这样计算是极不划算的。 因此，需要**修剪方法**以加快距离计算。

​		下面是对修剪优化的几种方法：

### 2.1 基于单元的方法

#### 2.1.1 网格距离的由来

​	在基于单元格的技术中，数据空间被划分为单元格，单元格的宽度是阈值$D$和数据维数$d$的函数。具体地说，每个维度被划分成宽度最多为 $\frac{D}{{2 \cdot \sqrt d }}$ 单元格。

> 其中：
>
> 1. 阈值距离 $D$，点与点之间距离超过$D$，则不再认为这两个点为“近邻”。
>
> 2. $d$为dimensionality，维度。

![@图 1 基于单元的数据空间分区 | center| 500x0](img/UWiX5C7kCHx5yX7O9yQm9F1ndg-QgMqS3BAwIWPB40k.original.fullsize-1609839833441.png)

​	网格距离$\frac{D}{{2 \cdot \sqrt d }}$是为了达成以下目标而设定的：

> - $L1$邻居中的所有点与当前网格所有点的距离均小于等于阈值距离$D$，即均为"近邻"。
>
> - $L_2$以外的邻居与当前网格所有点的距离均大于阈值距离$D$，即均为"远邻"
> - 只有$L_2$中的邻居节点，无法判断与当前网格中点到距离与阈值距离的关系，需要进行显式距离计算。

#### 2.1.2 以二维为例，看各个邻居与当前单元格的距离

以二维情况为例，此时网格间的距离为 $\frac{D}{{2 \cdot \sqrt d }}$ =$\frac{D}{{2 \cdot \sqrt 2 }}$。

对于给定的单元格：

 $L_{1}$ 邻居被定义为通过最多1个单元间的边界可从该单元到达的单元格的集合。请注意，在一个角上接触的两个单元格也是 $L_{1}$ 邻居。 

 $L_{2}$ 邻居是通过跨越2个或3个边界而获得的那些单元格。 

上图中显示了标记为 $X$的特定单元格及其 $L_{1}$ 和 $L_{2}$ 邻居集。 显然，内部单元具有8个 $L_{1}$ 邻居（3\*3-1=8）和40个 $L_{2}$ 邻居（7\*7-3*3=40）。 

然后，可以立即观察到以下性质：

> - 单元格中两点之间的距离最多为$\frac{D}{{2 \cdot \sqrt 2 }}$*$\sqrt 2$ =  $D/2$。 **($d$=2，二维空间两点之间最长距离为对角线，为$\sqrt 2$网格距离)。**
> - 一个点与 $L_{1}$ 邻接点之间的距离最大为 $D$。**（道理同上）**
> - 一个点与它的 $Lr$ 邻居(其中$r$ > 2)中的一个点之间的距离至少为$D$。 **(距离最短为 $3*\frac{D}{{2 \cdot \sqrt 2 }}$ >$D$)**
>

满足预设。

#### 2.1.3 基于单元格计算异常值、非异常值（简单快捷）

在给定的单元以及相邻的单元中存在的数据点满足某些特性，这些特性可以让数据被更有效的处理。这些基于单元格非常简单快捷的计算异常值、非异常值的规则是：

> 1. 如果单元A及其相邻$L_{1}$ 中包含超过 $k$ 个数据点，那么单元A及其相邻$L_{1}$ 中的所有数据点都不是异常值。
> 2. 如果单元 $A$ 及其相邻 $L_{1}$ 和 $L_{2}$ 中包含少于 $k$ 个数据点，则单元A中的所有点都是异常值。

1.  利用规则1，标记非异常值；

2.  利用规则2，标记异常值；

3.  对于此时仍未标记为异常值或非异常值的单元格中的数据点需要明确计算其 $k$ 最近邻距离。

对于3的计算注意事项：

​	3.1  对于那些在  $L_{1}$ 和  $L_{2}$ 中不超过 $k$ 个且距离小于 $D$ 的数据点，则声明为异常值。

​	3.2  仅需要对单元 $A$ 中的点到单元$A$的$L_{2}$邻居中的点执行显式距离计算。这是因为已知 $L_{1}$ 邻居中的所有点到 $A$ 中任何点的距离都小于 $D$，并且已知 $Lr$ 中 $(r> 2)$ 的所有点与 $A$上任何点的距离至少为 $D$），减少计算开销。



#### 2.1.4 注意事项

- 网格单元的数量基于数据空间的分区，并且与数据点的数量无关。

- 该方法在低维数据上的效率很高，不适用于高维数据。
- 超参数：网格距离$D$, 近邻数量$k$



### 2.2 基于索引的方法（暂不深入研究）

​		对于一个给定数据集，基于索引的方法利用多维索引结构(如 $\mathrm{R}$ 树、$k-d$ 树)来搜索每个数据对象 $A$ 在半径 $D$ 范围 内的相邻点。设 $M$ 是一个异常值在其 $D$ -邻域内允许含有对象的最多个数，若发现某个数据对象 $A$ 的 $D$ -邻域内出现 $M+1$ 甚至更多个相邻点， 则判定对象 $A$ 不是异常值。该算法时间复杂度在最坏情况下为 $O\left(k N^{2}\right),$ 其中 $k$ 是数据集维数， $N$ 是数据集包含对象的个数。该算法在数据集的维数增加时具有较好的扩展性，但是时间复杂度的估算仅考虑了搜索时间，而构造索引的任务本身就需要密集复杂的计算量。

> 构建索引需要的计算量比较昂贵，搜索计算复杂度看起来也不低，不太适用于自己的业务场景，暂时先略过了。



## 3、基于密度的度量（LOF）          

​		基于距离的检测适用于各个集群的密度较为均匀的情况，基于密度的算法则可以较好地适应密度不同的集群情况。如Figure1所示，基于距离的检测中，只有$o1$会被判断为离群点，而在基于密度的检测中，$o1$ 和$o2$均会被识别为离群点。

​		基于密度的算法主要有局部离群因子(LocalOutlierFactor,LOF)，以及LOCI、CLOF等基于LOF的改进算法。下面我们以LOF为例来进行详细的介绍和实践。

![image-20210120110117675](D:\WorkSpace\GitHub\DataWhale_LearningNote\AnomalyDetection\img\image-20210120110117675.png)



&emsp;&emsp; 那么，这个基于密度的度量值是怎么得来的呢？还是要从距离的计算开始。类似$k$近邻的思路，首先我们也需要来定义一个“$k$-距离”。

### 3.1 $k$-距离（$k-distance(p)$）：    

数据集$D$，给定对象$p$，距离对象$p$距离第$k$近的点为$o$

$d(p,o)$：点$p$与$o$之间的距离

$k-distance(p)$：对象$p$的第$k$距离$d_k(p)$

$d(p,o)$=$k-distance(p)$且满足：

+ 在集合D中至少有不包括$p$在内的$k$个点 $o'$，其中$o'∈D\backslash\{p\}$，满足$d(p,o')≤d(p,o)$
+ 在集合D中最多有不包括$p$在内的$k-1$个点$o'$，其中$o'∈D\backslash\{p\}$，满足$d(p,o')<d(p,o)$    

&emsp;&emsp;直观一些理解，就是以对象$p$为中心，对数据集$D$中的所有点到$p$的距离进行排序，距离对象$p$第$k$近的点$o$与$p$之间的距离就是$k$-距离。

![图3](https://img-blog.csdn.net/20160618160343957)

### 3.2 k-邻域（k-distance neighborhood）：    

对象$p$的第$k$距离邻域 $N_k(p)$: 	$N_{k − d i s t a n c e ( p )}( P ) = \{ q ∈ D \backslash\{q\} ∣ d ( p , q ) ≤ k − d i s t a n c e ( p )\}$

就是$p$的第$k$距离即以内的所有点，包括第$k$距离点。

因此$p$的第$k$邻域点的个数 $|N_k(p)|≥k$。

### 3.3 可达距离（reachability distance）：    

首先，可达距离的设计同样是为了减少距离的计算开销，$o$的$k$-邻域的所有$p$的$d(p,o)$的统计波动可以被显著降低。**这种平滑效果的强度可以通过参数$k$来控制，$k$的值越高，表示同一邻域内物体的可达距离越接近。**

对象$p$到点$o$的可达距离定义：$r e a c h−d i s t_ k ( p , o ) = m a x \{k−distance( o ) , d ( p , o )\}$ 

<font color="red">**注意：这里用的是$o$的$k$-距离，不是$p$的$k$-距离！**</font>(这里把我绕晕了)

因此：

+ 若$p_i$在对象$o$的$k$-邻域内，则可达距离就是给定点$p$关于对象$o$的$k$-距离 $k−distance( o )$；
+ 若$p_i$在对象$o$的$k$-邻域外，则可达距离就是给定点$p$关于对象$o$的实际距离$ d ( p , o )$。  
  

这样的分类处理可以简化后续的计算，同时让得到的数值区分度更高。

### 3.4 局部可达密度（local reachability density）：

有两个参数可以定义密度的概念：(i)一个参数$MinPts$指定（--对象/区域的邻域）对象的最小数量; (ii)一个参数指定量。这两个参数决定了聚类算法操作的密度阈值。也就是说，如果对象或区域的邻域密度超过给定的密度阈值，那么它们就被连接起来，构成一个新的区域（--且均为非异常值）。

基于密度检测异常值，有必要比较不同对象集合的密度，这意味着我们必须动态地确定对象集合的密度。因此，我们保留$MinPts$作为唯一的参数，并使用对象$p$到$o∈N_{MinPts}(p)$的可达距离$reach-dist_{MinPts}(p, o)$作为度量对象$p$邻域的密度的值。

给定点$p$的局部可达密度计算公式为：$lrd_{MinPts}(p)=1/(\frac {\sum\limits_{o∈N_{MinPts}(p)} reach-dist_{MinPts}(p,o)} {\left\vert N_{MinPts}(p) \right\vert})$



![image-20210120162047426](D:\WorkSpace\GitHub\DataWhale_LearningNote\AnomalyDetection\img\image-20210120162047426.png)

直观上，对象$p$的局部可达密度是根据$p$的$MinPts$个最近邻的平均可达距离的倒数。

注意，如果所有可达距离之和为0，则局部密度为∞。这种情况发生在：对于对象$p$，存在至少$MinPts$个对象，不同于$p$，但和$p$共享相同的空间坐标。也就是说，数据集中有至少$MinPts$个$p$的重合点。为简单起见，我们将不显式地处理这种情况，而是简单地假设没有重复项。(为了处理重复，我们可以将邻域的概念建立在一个$k-distinct-distance$上，类似于前面定义的$k-distance$，附加条件是至少有$k$个具有不同空间坐标的对象。)

> --理解不一定绝对准确，可能会删掉
>
> *注意：是$p$的邻域点 $N_k(p)$到$p$的可达距离，不是$p$到 $N_k(p)$的可达距离，一定要弄清楚关系。*
>
> *如果是$p$到$N_k(p)$的可达距离，那可达距离均为$p$的k-距离。*
>
> *而$o∈N_k(p)$到$p$的可达距离是不一定的，要么是o的k-距离，要么是o与$p$的实际距离。*

由公式可以看出，这里是对给定点$p$进行度量，计算定点$p$到其邻域内的所有对象的可达距离平均值。给定点$p$的局部可达密度越高（也就是平均可达距离越短），越可能与其邻域内的点属于同一簇；密度越低（也就是平均可达距离越长），越可能是离群点.



### 3.5 局部异常因子：    

对象$p$的局部异常因子定义为：

![局部异常因子公式.png](img/局部异常因子公式-1609839840015.png)

对象$p$的局部异常因子刻画了$p$的异常程度。它是$p$的$MinPts$近邻的局部可达密度与对象$p$的局部可达密度的比值的均值。

不难看出，$p$的局部可达密度越低，$p$的$MinPts$近邻的局部可达密度越高，则p的LOF值越高。

如果这个比值越接近1，说明o的邻域点密度差不多，$o$可能和邻域同属一簇；如果这个比值小于1，说明$o$的密度高于其邻域点密度，$o$为密集点；如果这个比值大于1，说明$o$的密度小于其邻域点密度，$o$可能是异常点。

最终得出的LOF数值，就是我们所需要的离群点分数。



<img src="https://pic1.zhimg.com/80/v2-1d6429de1b0b7784ee148c5102eb746c_720w.jpg" alt="img" style="zoom:80%;" />

![img](https://pic3.zhimg.com/80/v2-577c9aff21d4fb77dd782eb8f978ad5e_720w.jpg)

（摘自知乎[LOF离群因子检测算法及python3实现](https://zhuanlan.zhihu.com/p/37753692)的两张图片，非常形象的解释了凌乱的$o$和$p$之间的各种倒腾）

PS：$ρ_2(O2)$的计算里有一点点笔误，看出来了吗？（偷笑



## 4、练习

sklearn中LocalOutlierFactor库可以用于**对单个数据集进行无监督的离群检测**，也可以**基于已有的正常数据集对新数据集进行新颖性检测**。

### 4.1 生成数据，调用sklearn的LOF算法进行单个数据集的无监督离群检测，并可视化

[Task4-相似度-LOF（无监督离群检测）.ipynb](https://github.com/ArriettySong/DataWhale_LearningNote/blob/main/AnomalyDetection/Task4-%E7%9B%B8%E4%BC%BC%E5%BA%A6-LOF%EF%BC%88%E6%97%A0%E7%9B%91%E7%9D%A3%E7%A6%BB%E7%BE%A4%E6%A3%80%E6%B5%8B%EF%BC%89.ipynb)

1. 构造一个含有集群和离群点的数据集。该数据集包含两个密度不同的正态分布集群和一些离群点。（PS：我们手工对数据点的标注其实是不准确的，可能有一些随机点会散落在集群内部）
2. 使用LocalOutlierFactor库对构造数据集进行训练，得到训练的标签和训练分数（局部离群值）。
3. 将训练分数（离群程度）用圆直观地表示出来，并对构造标签与训练标签不一致的数据用不同颜色的圆进行标注。

​		可以看出，模型成功区分出了大部分的离群点，一些因为随机原因散落在集群内部的“离群点”也被识别为集群内部的点，但是一些与集群略为分散的“集群点”则被识别为离群点。
​		同时可以看出，模型对于不同密度的集群有着较好的区分度，对于低密度集群与高密度集群使用了不同的密度阈值来区分是否离群点。

​		因此，我们从直观上可以得到一个印象，即基于LOF模型的离群点识别在某些情况下，可能比基于某种统计学分布规则的识别更加符合实际情况。

### 4.2 生成数据，调用sklearn的LOF算法进行新颖性检测，并可视化

参考官方文档：https://scikit-learn.org/stable/auto_examples/neighbors/plot_lof_novelty_detection.html?highlight=lof

[Task4-相似度-LOF（新颖性检测）.ipynb](https://github.com/ArriettySong/DataWhale_LearningNote/blob/main/AnomalyDetection/Task4-%E7%9B%B8%E4%BC%BC%E5%BA%A6-LOF%EF%BC%88%E6%96%B0%E9%A2%96%E6%80%A7%E6%A3%80%E6%B5%8B%EF%BC%89.ipynb)

注意：当LOF用于新颖性检测时，不可以在训练集上使用predict、decision和score样本（这将导致错误的结果），只能对新的不可见数据（不在训练集中）使用这些方法。

```python
clf = LocalOutlierFactor(n_neighbors=20, novelty=True, contamination=0.1)
```



### 4.3 信用卡欺诈数据，分别用PyOD库和sklearn调用LOF算法

[Task4-相似度-LOF（信用卡欺诈数据）.ipynb](https://github.com/ArriettySong/DataWhale_LearningNote/blob/main/AnomalyDetection/Task4-%E7%9B%B8%E4%BC%BC%E5%BA%A6-LOF%EF%BC%88%E4%BF%A1%E7%94%A8%E5%8D%A1%E6%AC%BA%E8%AF%88%E6%95%B0%E6%8D%AE%EF%BC%89.ipynb)

precision@rank n远低于前几天的PCA、HBOS等算法，该数据可能不太适合LOF算法。

![image-20210120180517381](D:\WorkSpace\GitHub\DataWhale_LearningNote\AnomalyDetection\img\image-20210120180517381.png)



## 5. 参考资料

1. LOF: Identifying Density-Based Local Outliers
2. sklearn上关于LOF的新颖性检测 https://scikit-learn.org/stable/auto_examples/neighbors/plot_lof_novelty_detection.html?highlight=lof 

3. 异常检测几种常用模型比较 https://scikit-learn.org/stable/auto_examples/miscellaneous/plot_anomaly_comparison.html#sphx-glr-auto-examples-miscellaneous-plot-anomaly-comparison-py 

4. [LOF离群因子检测算法及python3实现](https://zhuanlan.zhihu.com/p/37753692)