---
title: "複数直線の近傍点を求める"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [math, geometry]
published: true
---

# その一（愚直に）
$\bm c$を複数直線の近傍点，$\bm u_i$を直線上のある点，$\bm v_i$を直線の方向ベクトルとする．

![](/images/20210915/fig1.png =250x)

複数直線の近傍点を求めるには，以下の式を最小化すればよい．

$$ f(\bm c, k_i) = \sum_i ||\bm c - (\bm u_i + k_i \bm v_i)||^2$$

上式を偏微分して解く．

$$
\begin{align}
\frac{df}{d\bm c} = 0 &\Leftrightarrow \bm c = \frac{1}{n} \sum_i^n (\bm u_i + k_i \bm v_i) \\
\frac{df}{dk_i} = 0 &\Leftrightarrow k_i = \frac{\bm c^T \bm v_i - \bm u_i^T \bm v_i}{\bm v_i^T \bm v_i}
\end{align}
$$

式(1)に式(2)を代入し，$\bm c$について解けば近傍点が求まる．

# その二（少しお洒落に）
複数直線の近傍点$\bm c$と直線の距離$d_i$は，以下の式で表すことができる．

$$
\begin{align*}
d_i &= ||\bm c - \bm u_i||\sin \theta \\
&= ||\bm v \times (\bm c - \bm u_i)||
\end{align*}
$$

![](/images/20210915/fig2.png =250x)

複数直線の近傍点は，以下の式で表される．

$$ \bm c = \mathop{\rm arg~min} \sum_i d_i^2$$

$\sum_i d_i^2$を$\bm c$の成分ごとに偏微分して解けば，近傍点が求まる．

# 参考文献
[Point cloud to point cloud rigid transformations](https://www.cs.jhu.edu/cista/455/Lectures/Rigid3D3DCalculations.pdf)
[Nearest approaches to multiple lines in n-dimensional space](https://www.crewes.org/Documents/ResearchReports/2010/CRR201032.pdf)