##A Density-Based Algorithm for Discovering Clusters in Large Spatial Databases with Noise

> Clustering algorithms are attractive for the task of class identification in spatial databases. However, the application to large spatial databases rises the following requirements for clustering algorithms: minimal requirements of domain knowledge to determine the input parameters, discovery of clusters with arbitrary shape and good efficiency on large databases. The well-known clustering algorithms offer no solution to the combination of these requirements. In this paper, we present the new clustcring algorithm DBSCAN relying on a density-based notion of clusters which is designed to discover clusters of arbitrary shape. DBSCAN requires only one input parameter and supports the user in determining an appropriate value for it. We performed an experimental evaluation of the effectiveness and efficiency of DBSCAN using synthetic data and real data of the SEQUOIA 2000 benchmark. The results of our experiments demonstrate that(l) DBSCAN is significantly more effective in discovering clusters of arbitrary shape than the well-known algorithm CLARANS, and that(2) DBSCAN outperforms CLARANS by a factor of more than 100 in terms of efficiency.
> Keywords: Clustering Algorithms, Arbitrary Shape of Clus-
> ters, Efficiency on Large Spatial Databases, Handling Nlj4-
> 275oise.

聚类算法被用于空间数据库中的类型识别任务，然而对于聚类算法有如下的要求：

- 对领域知识的最低要求，用于确定输入参数
- 发现任意形状的聚簇，_may be spherical, drawn-out, linear, elongated etc._
- 在大型数据库中表现良好的性能

作者提出了新的聚类算法DBSCAN，依赖基于密度的聚类的概念，旨在发现任意形状的聚类

###Clustering Algorithms

聚类算法有两种基本类型，分区和分层算法。

分区算法将有n个目标的数据库D的分区，构造成为一个k聚簇。k是算法的输入参数，需要领域知识。分区算法通常从初始分区D开始，使用迭代控制策略优化目标函数。每个聚簇由聚簇的重心来表示_(k-means)_，或者位置上接近中心的一个目标_(k-medoid)_

因此，分区算法有两个步骤。首先，确定k个代表_(representatives)_的最小目标函数，其次，将每个目标聚集到离它最近的代表。第二步意味着一个分区在voronoi图_(voronoi diagram)_中是等价的，并且每个聚簇包含在一个voronoi单元_(voronoi cells)_中。所以，分区算法中所有聚簇的形状都是凸状的，而这种情况是有限的。

CLARANS _(Clustering Large Applications based on RANdomized Search)_是一个改进的k-medoid方法。Ng & Han也讨论了确定数据库中聚簇的”自然“数$k_{nat}$的方法。他们建议每个聚簇从2到n运行CLARANS一次。计算出每个发现的聚簇的轮廓系数，最后，最大轮廓系数的聚簇即为“自然”聚簇。遗憾的是，大量样本时这种方法的运行时间昂贵，因为它属于CLARANS的$O(n)$次调用。

CLARANS假设所有聚簇的目标可以同时驻留在主存中，而这不适用于大型数据库，并且时间代价昂贵。

分层算法创建一个数据库D的层次分解。层次分解以树形图_(dendrogram)_表示，树迭代的将D分割为更小的子集，知道每个子集仅由一个目标组成。

这样一个层次结构中，树的每个结点表示D的一个聚簇。树形图即可以从叶子向上到树根*(凝聚方法agglomerative approach)*形成，也可以从树根向下到树叶*(分裂方法divisive approach)*，通过每一步合并或者分割簇来生成。异于分区算法，层次算法不需要k作为输入。但是，必须定义终止条件，指定何时终止合并或者分割过程。凝聚算法中终止条件的一个例子是所有簇Q之间的临界距离$D_{min}$。

到目前为止，层次聚类算法的主要问题是难以获得终止条件的适当参数。例如，$D_{min}$的值足够小以分离所有的自然簇，并且同时足够大以使得没有簇被分割为两部分。最近，在信号处理领域，已经提出了分层算法Ejcluster（García，Fdez-Valdivia，Cortijo＆Molina 1994)自动导出终止条件。它的关键思想是，如果一个足够小的步骤可以从第一个点走到第二个点，那么两个点属于同一个簇。 Ejcluster遵循分裂的方法。它不需要任何域知识输入。此外，实验表明它在发现非凸簇方面非常有效。然而，由于每对点的距离计算，Ejcluster的计算成本是$O(n^2)$。这对于诸如字符识别之类中等程度的n值的应用程序是可接受的，但是对于大型数据库上的应用程序来说这是代价高昂的。 Jain（1988)探索了一种基于密度的方法来识别k维点集中的聚类。数据集被划分为许多非重叠单元格_(nonoverlapping cells)_，并构造直方图。具有相对高频率计数的单元格是潜在的聚类中心，并且聚类之间的边界落入直方图的“谷”中。

### A Density Based Notion of Clusters 

对聚簇的认识：每个聚簇有一个典型的密度，远远高于簇的外部。此外，噪声区域的密度低于任何的簇。

关键思想是对于聚簇的每个点，给定半径的邻域必须包含至少最小数量的点，即邻域的密度必须超过某个阈值。邻域的形状通过选择两个点p和q的距离函数$dist(p,q)$来确定。例如，在2D空间使用曼哈顿距离，邻域的形状是矩形。DBSCAN适用于任何距离函数，可以在给定的应用中选择适当的函数。

**Definition 1**:(Eps-neighborhood of a point) The Eps-neighborhood of a point p, denoted by $N_{Eps}(P)$, is defined by $N_{Eps}(P)= {q \in D | dist(p,q) \le Eps}$.

一种朴素的方法可能要求簇中的每个点，在该点的Eps邻域中至少存在最小数量（MinPts)的点。然而这种方法失败了，因为簇中有两种点，簇内部的点 _(core points)_和簇边界上的点_(border points)_。通常，边界点的Eps邻域包含比核心点的Eps邻域少得多的点。因此不得不设置相对较小的点的最小值来包含属于同一聚簇的所有点。然而，这个值不是各个簇的特征，特别是存在噪音时。所以，我们要求对于簇C中的每个点p，在C中存在点q，使得p在q的Eps邻域内，并且NEps（q)包含至少MinPts点。

**Definition 2**:(directly density-reachable) A point p is directly density-reachable from a point q wrt. Eps, MinPts if 

1) $p \in N_{Eps}(q)$

2) $|N_{Eps}(q)|\ge  MinPts$(core point condition). 

显然，直接密度可达对于核心点对是对称的。然而，一般而言，如果涉及一个核心点和一个边界点，则它不是对称的。

**Definition 3**:(density-reachable) A point P is density-reachable from a point q wrt. Eps and MinPts if there is a chain of points $p_1 ,\dots ,p_n, p_1=q, p_n= p$ such that $p_{i+1}$ is directly density-reachable from $p_i$.

密度可达性是直接密度可达性的规范扩展。这个关系是可传递的，但不是对称的

**Definition 4**:(density-connected) A point p is density connected to a point q wrt. Eps and MinPts if there is a point o such that both, p and q are density-reachable from o wrt. Eps and MinPts. 

密度连通性是对称的，随遇密度可达的点，密度连通性是自反的。

簇被定义为一组密度连接点，具有最大的密度可达性。想对给定的一组的簇的集合定义噪声。噪声只是D中不属于任何一个簇的一组点集。

**Definition 5**:(cluster) Let D be a database of points. A cluster C wrt. Eps and MinPts is a non-empty subset of D satisfying the following conditions: 

1) $\forall \ p, q: if\ p \in C$ and q is density-reachable from p wrt. Eps and MinPts, then $q \in C$.(Maximality) 

2) $\forall \ p,q \ \in C$ is density-connected to q wrt. EPS and MinPts.(Connectivity)

**Definition 6**:(noise) Let $C_1,..., C_k$ be the clusters of the database D wrt. parameters $Eps_i$ and $MinPts_i$,i= 1,....k. Then we define the noise as the set of points in the database D not belonging to any cluster $C_i$. i.e. noise=$(p\in D|\forall i:p \notin C_i)$.  

集群C至少包含一个点p，p必须通过某个点o（可能等同于p)与其自身密度连接。因此，o至少需要满足核心点的条件，因此o的eps领域至少包含MinPts个点。

**Lemma 1**: Let p be a point in D and $|N_{Eps}(p)| \ge MinPts$. Then the set $O=\{o|o \in D$ and o is density-reachable from p wrt. Eps and MinPts} is a cluster wrt. Eps and MinPts. 

聚簇C并不明显的由任何核心点决定。但是C中的每个点密度可达任何一个核心点，所以C恰好包含从C的任意核心点可以密度可达的点。

**Lemma 2**: Let C be a cluster wrt. Eps and MinPts and let p be any point in C with $|N_{Eps}(p)| \ge MinPts$. Then C equals to the set $O=\{o|o \in D$ and o is density-reachable from p wrt. Eps and MinPts}

### 4.DBSCAN: Density Based Spatial Clustering of Applications with Noise

基于密度的噪声空间聚类的应用

根据定义5和6发现空间数据库中的聚簇和噪声

简单有效的启发式方法可以确定“最薄”（即最低密度)簇的参数Eps和MinPts的值

##### 4.1算法

DBSCAN从任意点p开始，检索所有密度可达的点，根据Eps和MinPts。如果p是核心点，此过程产生一个聚（Lemma 2)。如果p是边缘点，没有点可以从p密度可达，那么DBSCAN访问下一个点。

由于我们使用Eps和MinPts的全局值，如果两个不同密度的簇彼此接近，则DBSCAN可以根据定义5将两个簇合并为一个簇。设两组点$S_1$和$S_2$之间的距离定义为$dist(S_1,S_2)=min\{dist(p,q)|lp \in S_1, q \in S_2 \}$。然后，仅当两组之间的距离大于Eps时，具有至少最薄簇的密度的两组点将彼此分离。因此，对于具有较高MinPts值的簇，可能需要递归调用DBSCAN。

```
DBSCAN的基本版本

DBSCAN(SetOfPoints, Eps, MinPts) 
// SetOfPoints is UNCLASSIFIED
 	ClusterId := nextId(NOISE);
 	FOR i FROM 1 TO SetOfPoints.size DO 
		Point := SetOfPoints.get(i);
 		IF Point.ClId = UNCLASSIFIED THEN 
			IF ExpandCluster(SetOfPoints, ClusterId, Eps, MinPts) THEN 
				ClusterId := nextId(ClusterId) 
			END IF 
		END IF 
	END FOR 
END; // DBSCAN 

```

SetOfPoints是整个数据库或先前运行中发现的群集。 Eps和MinPts是手动或根据4.2节中介绍的启发法确定的全局密度参数。函数SetOfPoints.get(i)返回SetOfPoints的第i个元素。 DBSCAN使用的最重要的功能是ExpandCluster，如下所示：

```
ExpandCluster(SetOfPoints, Point, CiId, Eps, MinPts) : Boolean;
	seeds:=SetOfPoints.regionQuery(Point, Eps)
	IF seeds.size<MinPts THEN // no core point
		SetOfPoint.changeClId(Point, NOISE)
		RETURN False;
	ELSE // all points in seeds are density-
			// reachable from Point 
		SetOfPoints.changeClIds(seeds, ClId);
		seeds.delete(Point); 
		WHILE seeds <> Empty DO
			currentP := seeds.first();
			result := setofPoints.regionQuery(currentP,Eps);
			IF result.size >= MinPts THEN 
				FOR i FROM 1 TO result.size DO
					resultP := result.get(i) 
					IF resultP.ClId IN {UNCLASSIFIED, NOISE} THEN
						IF resultP.ClId = UNCLASSIFIED THEN
							seeds, append(resultP)
						END IF;
						SetOfPoints. changeClId(resultP, ClId)
					END IF; // UNCLASSIFIED or NOISE 
				END FOR ;
			END IF; // result.size >= MinPts
			seeds, delete (currentP)
		END WHILE; // seeds <> Empty
		RETURN True; 
	END IF
END; // ExpandCluster
```

