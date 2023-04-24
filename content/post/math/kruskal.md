+++
author = "柏华"
title = "图论-图解-克鲁斯卡尔 (Kruskal) 算法"
date = "2022-05-19"
description = "图论-图解-克鲁斯卡尔 (Kruskal) 算法"
featured = true
tags = [
    "图论",
    "图觖算法",
]
isCJKLanguage = true
+++

在图论中，克鲁斯卡尔 (Kruskal) 算法是用于求解图的最小生成数的典型算法。有很多代表性的应用。

<!--more-->
# 克鲁斯卡尔 (Kruskal)  算法
## 概念
设图 <font color='red'>G=<V,E></font> 是一个具有 n 个顶点的带权连通无向图。图 <font color='red'>T=<V, TE></font> 是图 G 的最小生成树。
- V 是图 T 的顶点集。
- TE 是图 T 的边集。

如何由图 G 生成图 T，这就是该算法要解决的问题。 

## 算法
1. 将图 G 中的边按照权重从小到大排序。
2. 依次选取每条边，若选权的边<font color='red'>未使图 T 形成回路</font>，则加入到边集 TE 中； 否则舍弃，直到 TE 中包含 n - 1 条边为止。 

## 示例图解
- 带权连通无向图 G
![](/images/math/h1.png)

### 图解步骤
- 按权从小到大排序

| 边       | 权  |
|---------|----|
| (V0,V1) | 1  |
| (V0,V7) | 2  |
| (V1,V4) | 3  |
| (V4,V7) | 4  |
| (V1,V2) | 5  |
| (V4,V5) | 6  |
| (V2,V5) | 7  |
| (V2,V3) | 8  |
| (V3,V6) | 9  |
| (V5,V6) | 10 |
| (V0,V3) | 11 |
| (V6,V7) | 12 |
- 取第一条边
![](/images/math/h2.png)
- 取第二条边
![](/images/math/h3.png)
- 取第三条边
![](/images/math/h4.png)
- 取第四条边，构成了回路，舍弃
![](/images/math/h5.png)
- 取第五条边
![](/images/math/h6.png)
- 取第六条边
![](/images/math/h7.png)
- 取第七条边，构成了回路，舍弃
![](/images/math/h8.png)
- 取第八条边
![](/images/math/h9.png)
- 取第九条边
![](/images/math/h10.png)

此时的边数 = n(节点个数) - 1 = 8 - 1 = 7 条了，结束。