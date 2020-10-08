# 聚类
## cluster.vq
Provides routines for k-means clustering, generating code books from k-means models and quantizing vectors by comparing them with centroids in a code book.

function | introduction
----|----
whiten(obs[, check_finite]) | Normalize a group of observations on a per feature basis.每行元素除以该行的标准差。
vq(obs, code_book[, check_finite]) | Assign codes from a code book to observations.
kmeans(obs, k_or_guess[, iter, thresh, …]) | Performs k-means on a set of observation vectors forming k clusters.
kmeans2(data, k[, iter, thresh, minit, …]) | Classify a set of observations into k clusters using the k-means algorithm.

## cluster.hierarchy

Hierarchical clustering (scipy.cluster.hierarchy)

* These functions cut hierarchical clusterings into flat clusterings or find the roots of the forest formed by a cut by providing the flat cluster ids of each observation.

functions | introduction
----|----
fcluster(Z, t[, criterion, depth, R, monocrit]) | Form flat clusters from the hierarchical clustering defined by the given linkage matrix.
fclusterdata(X, t[, criterion, metric, …]) | Cluster observation data using a given metric.
leaders(Z, T) | Return the root nodes in a hierarchical clustering.

* These are routines for agglomerative clustering.

functions | introduction
----|----
linkage(y[, method, metric, optimal_ordering]) | Perform hierarchical/agglomerative clustering.
single(y) | Perform single/min/nearest linkage on the condensed distance matrix y.
complete(y) | Perform complete/max/farthest point linkage on a condensed distance matrix.
average(y) | Perform average/UPGMA linkage on a condensed distance matrix.
weighted(y) | Perform weighted/WPGMA linkage on the condensed distance matrix.
centroid(y) | Perform centroid/UPGMC linkage.
median(y) | Perform median/WPGMC linkage.
ward(y) | Perform Ward’s linkage on a condensed distance matrix.

* These routines compute statistics on hierarchies.


functions | introduction
----|----
cophenet(Z[, Y]) | Calculate the cophenetic distances between each observation in the hierarchical clustering defined by the linkage Z.
from_mlab_linkage(Z) | Convert a linkage matrix generated by MATLAB(TM) to a new linkage matrix compatible with this module.
inconsistent(Z[, d]) | Calculate inconsistency statistics on a linkage matrix.
maxinconsts(Z, R) | Return the maximum inconsistency coefficient for each non-singleton cluster and its children.
maxdists(Z) | Return the maximum distance between any non-singleton cluster.
maxRstat(Z, R, i) | Return the maximum statistic for each non-singleton cluster and its children.

to_mlab_linkage(Z) | Convert a linkage matrix to a MATLAB(TM) compatible one.

* Routines for visualizing flat clusters.

functions | introduction
----|----
dendrogram(Z[, p, truncate_mode, …]) | Plot the hierarchical clustering as a dendrogram.

* These are data structures and routines for representing hierarchies as tree objects.

functions | introduction
----|----
ClusterNode(id[, left, right, dist, count]) | A tree node class for representing a cluster.
leaves_list(Z) | Return a list of leaf node ids.
to_tree(Z[, rd]) | Convert a linkage matrix into an easy-to-use tree object.
cut_tree(Z[, n_clusters, height]) | Given a linkage matrix Z, return the cut tree.
optimal_leaf_ordering(Z, y[, metric]) | Given a linkage matrix Z and distance, reorder the cut tree.

* These are predicates for checking the validity of linkage and inconsistency matrices as well as for checking isomorphism of two flat cluster assignments.

functions | introduction
----|----
is_valid_im(R[, warning, throw, name]) | Return True if the inconsistency matrix passed is valid.
is_valid_linkage(Z[, warning, throw, name]) | Check the validity of a linkage matrix.
is_isomorphic(T1, T2) | Determine if two different cluster assignments are equivalent.
is_monotonic(Z) | Return True if the linkage passed is monotonic.
correspond(Z, Y) | Check for correspondence between linkage and condensed distance matrices.
num_obs_linkage(Z) | Return the number of original observations of the linkage matrix passed.

* Utility routines for plotting:


functions | introduction
----|----
set_link_color_palette(palette) | Set list of matplotlib color codes for use by dendrogram.

## 原理

K均值聚类是一种在一组未标记数据中查找聚类和聚类中心的方法。 直觉上，我们可以将一个群集(簇聚)看作 - 包含一组数据点，其点间距离与群集外点的距离相比较小。 给定一个K中心的初始集合，K均值算法重复以下两个步骤 -

* 对于每个中心，比其他中心更接近它的训练点的子集(其聚类)被识别出来。
* 计算每个聚类中数据点的每个要素的平均值，并且此平均向量将成为该聚类的新中心。


重复这两个步骤，直到中心不再移动或分配不再改变。 然后，可以将新点x分配给最接近的原型的群集。 SciPy库通过集群包提供了K-Means算法的良好实现。 下面来了解如何使用它。

## 实现

* 导入K-Means
```py
from SciPy.cluster.vq import kmeans,vq,whiten
Python
```
* 数据生成
```py
from numpy import vstack,array
from numpy.random import rand

# data generation with three features
data = vstack((rand(100,3) + array([.5,.5,.5]),rand(100,3)))
```
* 根据每个要素标准化一组观察值。 在运行K-Means之前，使用白化重新缩放观察集的每个特征维度是有好处的。 每个特征除以所有观测值的标准偏差以给出其单位差异。美化数据
```py
# whitening of data
data = whiten(data)
print (data)
```
* 用三个集群计算K均值现在使用以下代码计算三个群集的K均值。
```py
# computing K-Means with K = 3 (2 clusters)
centroids,_ = kmeans(data,3)
```
* 上述代码对形成K个簇的一组观测向量执行K均值。 K-Means算法调整质心直到不能获得足够的进展，即失真的变化，因为最后一次迭代小于某个阈值。 在这里，可以通过使用下面给出的代码打印centroids变量来观察簇。
```py
print(centroids)
```
* 使用下面给出的代码将每个值分配给一个集群。
```py
# assign each sample to a cluster
clx,_ = vq(data,centroids)
```
* vq函数将'M'中的每个观察向量与'N' obs数组与centroids进行比较，并将观察值分配给最近的聚类。 它返回每个观察和失真的聚类。 我们也可以检查失真。使用下面的代码检查每个观察的聚类。