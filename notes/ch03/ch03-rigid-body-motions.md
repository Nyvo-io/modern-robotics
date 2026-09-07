# Modern Robotics 第 3 章：刚体运动（Rigid-Body Motions）

来源：本地教材 PDF，第 3 章从书页 59／本地 PDF 第 78 页开始。

## 本章路线图

1. 区分物理几何对象与它在特定参考系中的坐标表示。
2. 从平面刚体运动过渡到三维旋转矩阵与角速度。
3. 用齐次变换矩阵表示空间刚体的位置与姿态。
4. 用螺旋轴（screw axis）、扭旋（twist）与指数坐标（exponential coordinates）表示运动。
5. 将力矩与力组合成力旋（wrench），为后续运动学与动力学奠基。

## 章前导读：关于向量与参考系

第 2 章回答了“描述空间刚体最少需要 6 个数”；第 3 章接着解决“这些几何量如何在不同参考系中表示和转换”。

### 本节路线图

1. 先区分物理对象与坐标数组。
2. 再用 Figure 3.1 观察同一点在两个参考系中的不同表示。
3. 最后区分空间参考系（space frame）与物体参考系（body frame）。

### 物理对象与坐标表示

自由向量（free vector）是具有长度和方向的几何对象；它本身不必固定在某一个起点。例如线速度的箭头方向表示运动方向，长度表示速率。

选定参考系（reference frame）和长度尺度后，才能把自由向量写成数值向量 $v\in\mathbb{R}^n$。更换参考系后，数值 $v$ 通常改变，但底层的物理向量不变。因此：

- **坐标无关对象**：物理空间中的点或向量。
- **坐标表示**：它在某个已选参考系中的数值坐标。

### Figure 3.1：同一物理点的两种坐标

![教材图 3.1：同一物理点在不同参考系中的坐标](../../attachments/ch03/fig-3-1-point-coordinates.png)

*来源：教材 Figure 3.1，书页 61／本地 PDF 第 80 页；局部截取。*

图中只有一个物理点 $p$，但有两个参考系 $\{a\}$ 和 $\{b\}$。两个参考系的原点、轴方向和长度尺度不同，所以它们给同一点分别配上

$$
p_a=(1,2),
\qquad
p_b=(4,-2).
$$

$\{a\}$、$\{b\}$ 表示参考系；$\hat{x}_a,\hat{y}_a$ 和 $\hat{x}_b,\hat{y}_b$ 是它们的单位坐标轴。下标 $a,b$ 说明坐标是用哪个参考系表示的。

==加了一个hat表示他们是单位向量==


两个坐标数组不相等，不代表存在两个点。这就像用“相对校门”和“相对图书馆”描述同一座建筑：描述的数值不同，建筑本身没有改变。第 3.1 节会把这种坐标转换写成矩阵运算。

### 空间参考系与物体参考系

- 空间参考系（space frame）$\{s\}$ 是全书唯一的固定参考系，可以想成固定在房间一角。
- 物体参考系（body frame）$\{b\}$ 固连于刚体，用来追踪刚体的位置与姿态；其原点不必在刚体实体内，只要它相对刚体的位姿恒定。

> 教材的一项特别约定：进行瞬时运动学分析时，它把“物体参考系”理解为当下瞬间与运动刚体固连坐标系重合的静止惯性参考系（inertial frame）。这是本书的记号与分析约定，后续不应与动力学中的旋转非惯性系混淆。

### Figure 3.2：右手参考系与正转方向

![教材图 3.2：右手参考系与正转方向](../../attachments/ch03/fig-3-2-right-hand-rule.png)

*来源：教材 Figure 3.2，书页 62／本地 PDF 第 81 页；局部截取。*

本书的所有参考系都采用右手系（right-handed frame）。右手食指、中指和拇指依次对应 $\hat{x}$、$\hat{y}$ 和 $\hat{z}$ 轴，因而有

$$
\hat{x}\times\hat{y}=\hat{z}.
$$

$\times$ 表示叉积（cross product），不是点积！！ 带帽符号的 $\hat{x},\hat{y},\hat{z}$ 表示单位向量。前两根轴的方向一旦确定，第三根轴必须满足右手约定，不能任意选择。

这里叉积是找同时垂直于$\hat{x}$ 和 $\hat{y}$ 的方向,这个方向可能是正负z，所以通过右手定则选择哪一个。

判断绕某轴的正旋转（positive rotation）时，让右手拇指指向该轴的正方向；其余四指自然弯曲的方向就是正转方向。旋转矩阵、角速度和螺旋轴的正负号都依赖这个约定。

## 3.1 平面刚体运动（Rigid-Body Motions in the Plane）

章前导读区分了物理对象与坐标表示。本节开始把这种区分用于平面刚体：只要描述固连在刚体上的物体参考系，就能描述整个刚体的构型。

### 本节路线图

1. 用位置向量和朝向表示平面刚体构型。
2. 把朝向写成旋转矩阵。
3. 用矩阵和向量完成参考系转换与刚体位移。
4. 从平面螺旋运动引出扭旋和指数坐标。

### 用位置与朝向描述物体参考系

![教材图 3.3：平面刚体的固定参考系与物体参考系](../../attachments/ch03/fig-3-3-planar-body-frame.png)

*来源：教材 Figure 3.3，书页 63／本地 PDF 第 82 页；局部截取。*

图中 $\{s\}$ 是固定的空间参考系，$\{b\}$ 固连在灰色刚体上。平面刚体有 3 个自由度，它们在图中分成：

- 物体参考系原点的位置 $(p_x,p_y)$：它是 $\{b\}$ 的原点相对于 $\{s\}$ 原点的位置，并且用 $\{s\}$ 的坐标表示；
- 物体参考系相对 $\{s\}$ 的朝向角 $\theta$。

位置向量 $p$ 可用 $\{s\}$ 的坐标轴写成

$$
p=p_x\hat{x}_s+p_y\hat{y}_s.
$$

这个式子表示：从 $\{s\}$ 的原点出发，沿 $\hat{x}_s$ 走 $p_x$，再沿 $\hat{y}_s$ 走 $p_y$，就到达 $\{b\}$ 的原点。它只确定位置，还不能确定灰色刚体朝向哪里。

朝向可以用 $\theta$ 描述，也可以把 $\{b\}$ 的两根单位轴用 $\{s\}$ 的坐标表示：

$$
\begin{aligned}
\hat{x}_b &= \cos\theta\,\hat{x}_s+\sin\theta\,\hat{y}_s,\\
\hat{y}_b &= -\sin\theta\,\hat{x}_s+\cos\theta\,\hat{y}_s.
\end{aligned}
$$

第一式给出 $\hat{x}_b$ 在 $\{s\}$ 中的方向；第二式给出与它垂直、仍满足右手约定的 $\hat{y}_b$。因此，$(p_x,p_y,\theta)$ 一起才能完整指定平面刚体的构型。

### 平面旋转矩阵

将 $\hat{x}_b$ 和 $\hat{y}_b$ 在 $\{s\}$ 中的坐标向量作为两列排在一起，得到旋转矩阵（rotation matrix）

$$
P
=
[\hat{x}_b\ \hat{y}_b]
=
\begin{bmatrix}
\cos\theta & -\sin\theta\\
\sin\theta & \cos\theta
\end{bmatrix}.
$$

这个矩阵的用途是表示 $\{b\}$ 相对 $\{s\}$ 的朝向。它的列有明确的几何含义：

- 第一列 $(\cos\theta,\sin\theta)^T$ 是 $\hat{x}_b$ 用 $\{s\}$ 坐标表示后的数值；
- 第二列 $(-\sin\theta,\cos\theta)^T$ 是 $\hat{y}_b$ 用 $\{s\}$ 坐标表示后的数值。

虽然 $P$ 包含 4 个数，但它们不能独立选择：两列都必须是单位向量，而且彼此正交。因此平面旋转矩阵仍只有由 $\theta$ 描述的 1 个自由度。

配对的 $(P,p)$ 则描述 $\{b\}$ 相对 $\{s\}$ 的完整构型：$P$ 负责朝向，$p$ 负责位置。后续参考系转换将直接使用这个矩阵—向量对。

教材在本节使用 $P$ 表示这个旋转矩阵。为了显式标明“从物体参考系 $\{b\}$ 到空间参考系 $\{s\}$ 的表示关系”，也常将同一个矩阵记作 $R_{sb}$：

$$
R_{sb}\equiv P
=
\begin{bmatrix}
\cos\theta & -\sin\theta\\
\sin\theta & \cos\theta
\end{bmatrix}
=
\begin{bmatrix}
\vert & \vert\\
\hat{x}_b & \hat{y}_b\\
\vert & \vert
\end{bmatrix}.
$$

因此，刚体的位姿（pose）或构型可写成 $(R_{sb},p)$：$R_{sb}$ 表示朝向，$p=[p_x\ p_y]^T$ 表示位置。下标 $sb$ 可读作“用 $\{s\}$ 描述 $\{b\}$”。为与当前教材公式一致，本节后续仍以 $P$ 为主记号。

若点的 $\{b\}$ 坐标为 $q_b$，则其 $\{s\}$ 坐标为

$$
q_s=R_{sb}q_b+p.
$$

其中左乘 $R_{sb}$ 是将坐标表示从 $\{b\}$ 转换到 $\{s\}$，再加 $p$ 处理两个参考系原点之间的偏移；这不表示物理点真的先旋转再平移。

#### $\theta=90^\circ$ 的数值检查

当物体参考系相对固定参考系逆时针旋转 $90^\circ$ 时，

$$
\cos 90^\circ=0,
\qquad
\sin 90^\circ=1,
$$

因此

$$
P
=
\begin{bmatrix}
0 & -1\\
1 & 0
\end{bmatrix}.
$$

第一列 $(0,1)^T$ 说明 $\hat{x}_b$ 已经指向 $+\hat{y}_s$；第二列 $(-1,0)^T$ 说明 $\hat{y}_b$ 已经指向 $-\hat{x}_s$。这和把整个物体坐标系逆时针转四分之一圈的几何图像一致。

### 参考系复合：位置

![教材图 3.4：三个平面参考系的复合](../../attachments/ch03/fig-3-4-frame-composition.png)

*来源：教材 Figure 3.4，书页 64／本地 PDF 第 83 页；局部截取。*

图中的三个位置向量分别是：

- $p$：从 $\{s\}$ 原点到 $\{b\}$ 原点，用 $\{s\}$ 坐标表示；
- $q$：从 $\{b\}$ 原点到 $\{c\}$ 原点，用 $\{b\}$ 坐标表示；
- $r$：从 $\{s\}$ 原点到 $\{c\}$ 原点，用 $\{s\}$ 坐标表示。

我们想把“$\{s\}\to\{b\}$”和“$\{b\}\to\{c\}$”两段路程接起来，但 $p$ 和 $q$ 分别使用不同参考系的坐标，不能直接相加。旋转矩阵 $P$ 将 $q$ 从 $\{b\}$ 坐标转换为用 $\{s\}$ 坐标表示，因此

$$
r=Pq+p.
$$

计算顺序可以理解为：

1. 先算 $Pq$，得到“从 $\{b\}$ 到 $\{c\}$”这段位移在 $\{s\}$ 中的坐标，也就是改变 $q$ 的坐标表示；
2. 再加 $p$，将起点从 $\{b\}$ 原点移回 $\{s\}$ 原点。

整个公式的核心不是“先乘后加”的记忆顺序，而是：只有用同一参考系表示的向量才能直接相加。

### 参考系复合：朝向

Figure 3.4 中，$Q$ 表示 $\{c\}$ 相对 $\{b\}$ 的朝向，$P$ 表示 $\{b\}$ 相对 $\{s\}$ 的朝向。要得到 $\{c\}$ 相对 $\{s\}$ 的朝向，需要按参考系链条逐层转换：

$$
R=PQ.
$$

这个公式的用途是把两段相对朝向复合成一段：$Q$ 先把用 $\{c\}$ 表示的方向转换为用 $\{b\}$ 表示，$P$ 再把结果转换为用 $\{s\}$ 表示。矩阵作用顺序从右向左，所以 $Q$ 先作用，$P$ 后作用。

### 完整位姿复合

把朝向与位置放在一起，$\{c\}$ 相对 $\{s\}$ 的构型为

$$
(R,r)=(PQ,\,Pq+p).
$$

这对公式分别处理同一个参考系复合问题的两个部分：

- **朝向**：$R=PQ$，只需要复合两段旋转；
- **位置**：$r=Pq+p$，先将相对位移 $q$ 转换到 $\{s\}$，再加上 $\{b\}$ 原点的位置 $p$，得到 $\{c\}$ 原点在 $\{s\}$ 中的位置。

一个容易混淆的地方是：平移 $p$ 不会改变坐标轴的朝向，所以它只出现在 $r$ 中，不出现在 $R$ 中。

例如，当

$$
P=
\begin{bmatrix}
0 & -1\\
1 & 0
\end{bmatrix},
\quad
p=
\begin{bmatrix}
2\\
3
\end{bmatrix},
\quad
Q=I,
\quad
q=
\begin{bmatrix}
1\\
0
\end{bmatrix},
$$

有


$$
R=PQ=P,
\qquad
r=Pq+p
=
\begin{bmatrix}
2\\
4
\end{bmatrix}.
$$

### 坐标变换与刚体位移

![教材图 3.5：平面刚体位移及等效定点旋转](../../attachments/ch03/fig-3-5-rigid-body-displacement.png)

*来源：教材 Figure 3.5，书页 66／本地 PDF 第 85 页；局部截取。*

Figure 3.5(a) 中，灰色椭圆表示同一个刚体移动前后的实际位置。$\{d\}$ 固连在刚体上，初始时与固定参考系 $\{s\}$ 重合；经过变换 $(P,p)$ 后，它移动到与固定参考系 $\{b\}$ 重合的 $\{d'\}$。同样固连在刚体上的 $\{c\}$ 也随之移动到 $\{c'\}$。

图中的运动分为两步：

1. 变换 ① 用 $P$ 绕 $\{s\}$ 的原点实际旋转整个刚体；
2. 变换 ② 用 $p$ 在 $\{s\}$ 中实际平移整个刚体。

因此，固连参考系 $\{c\}$ 的新位姿满足

$$
R'=PR,
\qquad
r'=Pr+p.
$$

它们在代数形式上与 Figure 3.4 的复合公式相似，但物理含义不同：Figure 3.4 是同一个点或参考系没有移动，只改变其坐标表示；Figure 3.5 则是刚体和固连于它的参考系真的发生了运动。

### 等效的定点旋转

Figure 3.5(b) 给出了同一次刚体运动的另一种描述：不再把它拆成“绕 $\{s\}$ 原点旋转，再平移 $p$”，而是让整个刚体直接绕另一个固定点 $s$ 旋转角度 $\beta=90^\circ$。

这两种运动描述的中间过程不同，但起始位姿和最终位姿完全相同：

- Figure 3.5(a)：绕 $\{s\}$ 原点旋转，再平移；
- Figure 3.5(b)：绕固定点 $s$ 一次旋转到位。

点 $s$ 在这次等效描述中是旋转中心，因此运动前后保持不动。教材把这种运动看作平面螺旋运动（planar screw motion）的例子；此处先抓住“同一刚体位移可以有不同但等效的运动描述”，下一步再定义它的参数。


### 平面螺旋坐标

绕固定点 $s$ 的平面旋转可以用三个螺旋坐标（screw coordinates）表示：

$$
(\beta,s_x,s_y).
$$

三个量分工明确：

- $\beta$：刚体绕固定点转过的角度；
- $(s_x,s_y)$：固定旋转中心 $s$ 在空间参考系 $\{s\}$ 中的坐标。

注意斜体小写点 $s$ 与带花括号的参考系 $\{s\}$ 不是同一个对象：前者是旋转中心，后者是用来测量其坐标的固定参考系。

在 Figure 3.5(b) 中，

$$
(\beta,s_x,s_y)=\left(\frac{\pi}{2},0,2\right).
$$

它表示刚体绕 $\{s\}$ 中坐标为 $(0,2)$ 的固定点，沿正方向旋转 $\pi/2$。从三维视角看，对应的旋转轴穿过点 $s$ 并垂直于图面，也就是z轴。

### 螺旋轴的速度表示

同一个平面螺旋运动也可以通过“瞬时怎样运动”来描述，利用螺旋轴的速度。上面是直接给出旋转点的坐标。令角速度（angular velocity）归一化为

$$
\omega=1\ \text{rad/s},
$$

它是归一化后的，在二维中通常为 +1，-1，用于表达运动轴的旋转方向

角速度方向就是螺旋轴方向

再记录固定参考系原点 在这一旋转下的瞬时线速度（linear velocity）

$$
v=(v_x,v_y).
$$
这个v 是单位角速度所对应的线速度分量

将两者组合，得到螺旋轴（screw axis）

$$
S=(\omega,v_x,v_y).
$$

这里的 $v$ 不是固定旋转中心 $s$ 的速度；点 $s$ 的速度为零。

$v$ 指的是刚体 位于空间参考系 $\{s\}$ 原点的那个点，在该瞬间因绕 $s$ 旋转而具有的线速度。

角速度只告诉我们：

> 刚体“转多快、绕哪个方向转”，方向是旋转轴方向，大小事转动快慢。

线速度 v 描述某个点此刻往哪个方向移动，有多快。他可以描述出绕哪里转

对于纯旋转，刚体上某个点 q 的线速度是：

$$\boxed{ \dot q=\omega\times r }$$

其中  r 是从旋转轴指向这个点的位置向量

它结合w，还可以告知 旋转轴（平面中就是旋转中心）的位置

$$ \boxed{ \text{twist 中的 }(\omega,v)\text{ 一起决定瞬时螺旋轴的位置和运动性质} } $$

比如：

“绕原点转”和“绕点 (10,0) 转”都有同样的 $\omega=1$, 这时还需要一个 v 来编码轴的位置。因为不同点的v是不一样的，它还取决于点到旋转中心的距离



对于绕固定点 s (旋转中心) 的纯旋转，任意点q 的线速度部分为：
$$
\dot q=\omega\times(q-s)
$$

假如旋转中心是原点的话，那么就直接： $\dot q=\omega\times q$

这里的x是叉乘，在二维里，因为旋转轴是z轴，所以sin也不用了，在二维里可以直接写的像乘法一样。但后面引入三维就会产生区别


在二维平面中，把角速度写成三维形式：

$$
\boldsymbol{\omega}
=
\begin{bmatrix}
0\\
0\\
\omega
\end{bmatrix},
\qquad
q-s
=
\begin{bmatrix}
q_x-s_x\\
q_y-s_y\\
0
\end{bmatrix}.
$$

因此：

$$
\dot{q}
=
\boldsymbol{\omega}\times(q-s)
=
\begin{bmatrix}
-\omega(q_y-s_y)\\
\omega(q_x-s_x)
\end{bmatrix}.
$$

展开后得到：

$$
\dot{q}
=
\underbrace{
\begin{bmatrix}
\omega s_y\\
-\omega s_x
\end{bmatrix}
}_{v}
+
\underbrace{
\begin{bmatrix}
-\omega q_y\\
\omega q_x
\end{bmatrix}
}_{\boldsymbol{\omega}\times q}.
$$
它对总线速度，进行了一个分解

所以刚体的速度可以写成：

$$
\boxed{
\dot{q}=v+\boldsymbol{\omega}\times q
}
$$

其中平移速度分量为：

$$
\boxed{
v
=
-\boldsymbol{\omega}\times s
=
\begin{bmatrix}
\omega s_y\\
-\omega s_x
\end{bmatrix}
}
$$
得到 vx 和 vy

总结：
$$\underbrace{\text{绕 }s\text{ 旋转}}_{\dot q=\omega\times(q-s)} \quad\Longleftrightarrow\quad \underbrace{\text{绕原点旋转}}_{\omega\times q} + \underbrace{\text{线速度项}}_{v=-\omega\times s}.$$
在公式层面，将刚体绕旋转中心旋转 拆分成：

$$ \boxed{ \text{绕 }s\text{ 旋转} \equiv \text{绕原点旋转的速度} + \text{整体平移速度} } $$



旋转中心 s 有一个决定性特征：**位于旋转中心的点速度为零**

所以，在二维中，给你(v,w)：

$$v= \begin{bmatrix} v_x\\ v_y \end{bmatrix} = \begin{bmatrix} \omega s_y\\ -\omega s_x \end{bmatrix}. $$

只要 $\omega\neq 0$，你就可以反推出旋转中心：

$$ \boxed{ s_x=-\frac{v_y}{\omega}, \qquad s_y=\frac{v_x}{\omega} }$$

这也就是上面说的 在二维中，线速度能告诉你螺旋轴的位置

二维螺旋轴需要这三个参数，这个坐标也可以称为**平面刚体的 twist（速度旋量）**

从他也可以看出，二维刚体一瞬间可以做三种独立运动：

$$\boxed{ x\text{ 方向平移} + y\text{ 方向平移} + \text{平面内旋转} }$$

这和二维刚体构型有 3个自由度是对应的，只不过$((x,y,\theta)$ 描述的是位置和朝向，这里描述它们变化得多快


$(v,\omega)$ 描述了整个刚体的**瞬时速度场**

你给我一个刚体上的任意点 \(q\)，我只需要知道：$(v,\omega)$

就能算出它此刻的速度：

$$\dot q=v+\omega\times q$$



Figure 3.5(b) 中，旋转中心为 $s=(0,2)$。当 $\omega=1\ \text{rad/s}$ 且沿正方向旋转时，位于 $\{s\}$ 原点的刚体点初始向 $+\hat{x}_s$ 方向运动，速率为 $2$，因此

$$
v=(2,0),
\qquad
S=(1,2,0).
$$

$S$ 描述的是单位角速度下的运动方向与轴的位置关系，而不是这次运动最终转过的总角度。

令 $x$ 是刚体上所研究的点，$s$ 是旋转中心；在 Figure 3.5 的例子中，所研究的 $x$ 取在 $\{s\}$ 原点。令

$$
r=x-s
=
\begin{bmatrix}
x_x-s_x\\
x_y-s_y
\end{bmatrix}
.
$$

所以二维旋转的线速度公式是

$$
v=
\begin{bmatrix}
v_x\\
v_y
\end{bmatrix}
=
\omega
\begin{bmatrix}
-r_y\\
r_x
\end{bmatrix}
.
$$

所以

$$
v_x=-\omega r_y,
\qquad
v_y=\omega r_x,
$$

也可以直接写成

$$
v_x=-\omega(x_y-s_y),
\qquad
v_y=\omega(x_x-s_x).
$$


而到了三维：

线速度变成：

$$v= \begin{bmatrix} v_x\\ v_y\\ v_z \end{bmatrix}$$

角速度也变成三维：

$$\omega= \begin{bmatrix} \omega_x\\ \omega_y\\ \omega_z \end{bmatrix}$$

于是完整的 twist 有 **6 个分量**：

$$\boxed{ V= \begin{bmatrix} \omega\\ v \end{bmatrix} = \begin{bmatrix} \omega_x\\ \omega_y\\ \omega_z\\ v_x\\ v_y\\ v_z \end{bmatrix} }$$

这正好对应空间刚体的 6 DOF：

$$\boxed{ 3\text{ 个旋转速度} + 3\text{ 个平移速度} }$$


### 有限位移的指数坐标

螺旋轴 $S$ 只规定单位角速度下“沿哪个运动方向走”。要表示实际完成的一次有限位移，还要乘以转角 $\theta$：

$$
S\theta.
$$

教材把 $S\theta$ 称为平面刚体位移的指数坐标（exponential coordinates）。这里：

- $S=(\omega,v_x,v_y)$ 描述归一化的运动轴与运动方向；
- $\theta$ 描述沿该运动轴实际前进多少，也就是最终转角；
- $S\theta$ 将“方向”与“大小”组合成一次有限位移的三个参数。

Figure 3.5 中 $S=(1,2,0)$，实际转角为 $\theta=\pi/2$。这个例子里 $\theta$ 与前面几何描述使用的 $\beta$ 表示同一个转角，因此

$$
S\theta
=
(1,2,0)\frac{\pi}{2}
=
\left(\frac{\pi}{2},\pi,0\right).
$$

这三个数不是刚体最终的 $(x,y,\theta)$ 位姿坐标；它们是生成这次位移的指数坐标。


### 扭旋：瞬时运动速度

要描述刚体此刻运动得多快，不再乘有限转角 $\theta$，而是乘 旋转速率 $\dot\theta$。角速度与线速度的组合称为扭旋（twist）：

$\theta=\theta(t)$   $\dot\theta=\frac{d\theta}{dt}$

$$
V=S\dot\theta.
$$

若

$$
S=(\omega,v_x,v_y),
$$

则

$$
V
=
(\omega\dot\theta,\,v_x\dot\theta,\,v_y\dot\theta).
$$

这个公式. 用途是把归一化的运动方向 $S$ 按实际速率 $\dot\theta$ 缩放。其分量含义为：

- $\omega\dot\theta$：刚体的实际角速度；（因为w只不过是个单位角速度，用于表示方向，乘上速率，才是最终的实际角速度）
- $(v_x\dot\theta,v_y\dot\theta)$：空间参考系原点处刚体点的实际线速度。

v是单位角速度对应的线速度分量

因此，$S\theta$ 与 $S\dot\theta$ 的核心区别是：

- $S\theta$ 描述一段累计的有限位移；
- $V=S\dot\theta$ 描述某一瞬间的运动速度。

若以恒定速率 $\dot\theta=\theta$ 运动一秒，那么一秒内累计的运动量与 $S\theta$ 对应；这是教材用来联系两种表示的特殊情形。

### 3.1 小结

- 平面刚体构型由位置与朝向组成，可写成 $(P,p)$。
- $(P,p)$ 可以表示构型、改变坐标表示，也可以表示实际刚体位移；公式相似时必须结合物理语境判断。
- 参考系复合满足 $(R,r)=(PQ,Pq+p)$，矩阵从右向左作用。
- 平面螺旋运动可用固定旋转中心和角度描述，也可用螺旋轴 $S$ 描述。
- $S\theta$ 是有限位移的指数坐标，$V=S\dot\theta$ 是瞬时扭旋。

## 3.2 三维空间中的旋转与角速度（Rotations and Angular Velocities）

第 3.1 节在平面中用两根物体坐标轴构造旋转矩阵；本节把同一思路推广到三维，并进一步研究旋转矩阵的约束、运算和角速度。

### 本节路线图

1. 用三根物体坐标轴构造三维旋转矩阵。
2. 理解旋转矩阵的约束、逆矩阵和复合运算。
3. 用轴—角与指数坐标表示三维旋转。
4. 将旋转随时间的变化连接到角速度。

### 3.2.1 三维旋转矩阵的列

![教材图 3.6：三维刚体的位置与朝向](../../attachments/ch03/fig-3-6-spatial-frames.png)

*来源：教材 Figure 3.6，书页 68／本地 PDF 第 87 页；局部截取。*

图中 $\{s\}$ 是固定的空间参考系，$\{b\}$ 固连于刚体。位置向量 $p$ 从 $\{s\}$ 的原点指向 $\{b\}$ 的原点，描述刚体的位置；三根物体坐标轴 $\hat{x}_b,\hat{y}_b,\hat{z}_b$ 描述刚体的朝向。

将三根物体坐标轴(单位方向向量)分别用 $\{s\}$ 的坐标表示：

$$
\begin{aligned}
\hat{x}_b &= r_{11}\hat{x}_s+r_{21}\hat{y}_s+r_{31}\hat{z}_s,\\
\hat{y}_b &= r_{12}\hat{x}_s+r_{22}\hat{y}_s+r_{32}\hat{z}_s,\\
\hat{z}_b &= r_{13}\hat{x}_s+r_{23}\hat{y}_s+r_{33}\hat{z}_s.
\end{aligned}
$$


把这三个坐标向量依次放入矩阵的三列，得到三维旋转矩阵

$$
R
=
[\hat{x}_b\ \hat{y}_b\ \hat{z}_b]
=
\begin{bmatrix}
r_{11} & r_{12} & r_{13}\\
r_{21} & r_{22} & r_{23}\\
r_{31} & r_{32} & r_{33}
\end{bmatrix}.
$$

每一列的几何含义与二维情况完全一致：

- 第一列是 $\hat{x}_b$ 在 $\{s\}$ 中的坐标(这个单位向量箭头端点的坐标)；
- 第二列是 $\hat{y}_b$ 在 $\{s\}$ 中的坐标；
- 第三列是 $\hat{z}_b$ 在 $\{s\}$ 中的坐标。

所以 $R$ 描述 $\{b\}$ 相对 $\{s\}$ 的朝向，而 $(R,p)$ 一起描述三维刚体相对 $\{s\}$ 的完整位姿。虽然 $R$ 有 9 个元素，但三根轴必须保持单位长度、相互垂直并满足右手规则，因此只有 3 个独立自由度；这些约束将在下一小步展开。

#### 单位长度与正交约束

旋转不能拉伸坐标轴，也不能改变坐标轴之间的直角。因此 $R$ 的三列必须满足两类条件：

- **单位长度（unit norm）**：每一列与自身的点积为 $1$；
- **正交性（orthogonality）**：任意两列之间的点积为 $0$。

令

$$
R=[\hat{x}_b\ \hat{y}_b\ \hat{z}_b],
$$

则

$$
R^TR
=
\begin{bmatrix}
\hat{x}_b^T\hat{x}_b & \hat{x}_b^T\hat{y}_b & \hat{x}_b^T\hat{z}_b\\
\hat{y}_b^T\hat{x}_b & \hat{y}_b^T\hat{y}_b & \hat{y}_b^T\hat{z}_b\\
\hat{z}_b^T\hat{x}_b & \hat{z}_b^T\hat{y}_b & \hat{z}_b^T\hat{z}_b
\end{bmatrix}
=I.
$$

这个式子的结构是：转置 $R^T$ 的每一行依次取出原矩阵的一列，再与 $R$ 的各列做点积。因此：

- 对角线元素是某根轴与自身的点积，等于 $1$；
- 非对角线元素是两根不同轴的点积，等于 $0$。

所以一个紧凑公式

$$
R^TR=I
$$

同时包含三条单位长度约束和三组两两正交约束。它保证三列构成一组标准正交基，但暂时还没有排除左手系；右手性将在下一步用行列式判断。

#### 行列式与右手性

由 $R^TR=I$ 可得

$$
\det(R^TR)
=\det(R)^2
=1,
$$

所以

$$
\det R=\pm1.
$$

这说明单位长度和正交约束只保证三根轴彼此垂直，却不能区分右手系与左手系。

真正的旋转必须保持坐标系的手性，不允许通过镜像把右手系翻成左手系，因此还要要求

$$
\det R=1.
$$

例如，矩阵

$$
F=
\begin{bmatrix}
1&0&0\\
0&1&0\\
0&0&-1
\end{bmatrix}
$$

满足 $F^TF=I$，因为三列仍是单位正交向量；但它把 $z$ 轴翻转，且 $\det F=-1$，所以表示镜像反射（reflection），不是三维旋转。

因此三维旋转矩阵必须同时满足

$$
R^TR=I,
\qquad
\det R=1.
$$

#### 特殊正交群 $SO(3)$

所有 三维旋转矩阵 组成的集合称为特殊正交群（special orthogonal group）$SO(3)$：

$$
SO(3)
=
\left\{
R\in\mathbb{R}^{3\times3}
\mid
R^TR=I,\ \det R=1
\right\}.
$$

名称中的三个部分分别表示：

- **正交（orthogonal）**：$R^TR=I$，三列是单位正交基；
- **特殊（special）**：$\det R=1$，排除行列式为 $-1$ 的镜像反射；
- **群（group）**：这些矩阵在矩阵乘法下可以复合，并满足封闭性、结合律、单位元与逆元等性质；后续再逐项讨论。

$SO(3)$ 中的 $3$ 指矩阵作用于三维空间，而不是说矩阵有三个元素。类似地，平面旋转矩阵组成 $SO(2)$：

$$
SO(2)
=
\left\{
R\in\mathbb{R}^{2\times2}
\mid
R^TR=I,\ \det R=1
\right\}.
$$

一个 $3\times3$ 矩阵有 9 个数；三条单位长度约束与三条两两正交约束去掉 6 个连续自由度，因此三维旋转只剩 3 个连续自由度。$\det R=1$ 是从 $\pm1$ 中选择右手分支，不再减少连续自由度。

$∥r1​∥=1,∥r2​∥=1,∥r3​∥=1$, 还有就是三列两两垂直


3个自由度也就是绕x轴，绕y轴，绕z轴转。 所以这就是为什么一个刚体的朝向我们用9个数的三维矩阵去表示，但它只有3个自由度

这就和空间刚体有6个自由度对上了：

一个空间刚体的构型需要有 位置 + 朝向  也就是前面说的：（p,R）

位置是3个自由度

朝向：R∈SO(3)，也是3个

所以总共有6个：

- 沿 x 平移
- 沿 y 平移
- 沿 z 平移
- 绕 x 转
- 绕 y 转
- 绕 z 转


#### 3.2.1.1 群性质与逆矩阵

$SO(3)$ 在矩阵乘法下构成一个群，意味着它满足：

1. **封闭性（closure）**：若 $A,B\in SO(3)$，则 $AB\in SO(3)$；
2. **结合律（associativity）**：$(AB)C=A(BC)$；
3. **单位元（identity element）**：$I\in SO(3)$，且 $AI=IA=A$；
4. **逆元（inverse element）**：每个 $R\in SO(3)$ 都存在 $R^{-1}\in SO(3)$，使 $RR^{-1}=R^{-1}R=I$。

旋转矩阵的正交条件已经给出

$$
R^TR=I.
$$

对方阵而言，能在左侧与 $R$ 相乘得到 $I$ 的矩阵就是 $R$ 的逆矩阵，因此

$$
R^{-1}=R^T.
$$

它的几何意义是：$R$ 将坐标从一个参考系转换到另一个参考系，$R^T$ 则沿相反方向转换回来。换句话说，转置矩阵 撤销 原旋转。

这也是旋转矩阵在计算上的重要优点：求逆不需要一般的消元过程，只需交换行和列。

##### 旋转复合的封闭性

若 $A,B\in SO(3)$，它们分别表示两个合法旋转。复合后的矩阵是 $AB$。要证明 $AB$ 仍属于 $SO(3)$，需要检查定义中的两个条件。

首先检查正交性。注意乘积转置会反转顺序：

$$
(AB)^T=B^TA^T.
$$

因此

$$
\begin{aligned}
(AB)^T(AB)
&=B^TA^TAB\\
&=B^T(A^TA)B\\
&=B^TIB\\
&=B^TB\\
&=I.
\end{aligned}
$$

再检查行列式：

$$
\det(AB)
=\det A\det B
=1\times1
=1.
$$

两个条件都成立，所以

$$
AB\in SO(3).
$$

这就是封闭性（closure）的具体含义：连续执行两个真实旋转，结果仍是一个真实旋转。

##### 三维旋转的非交换性

矩阵乘法满足结合律，但通常不满足交换律（commutativity）。对三维旋转而言，通常有

$$
AB\ne BA.
$$

原因是右侧矩阵先作用：$AB$ 表示先做 $B$、再做 $A$，而 $BA$ 表示先做 $A$、再做 $B$。绕不同轴旋转时，第一次旋转会改变物体各轴的方向，因此第二次旋转面对的是不同状态。

例如，令 $R_x$、$R_y$ 分别表示绕 $+x$、$+y$ 轴正向旋转 $90^\circ$：

$$
R_x=
\begin{bmatrix}
1&0&0\\
0&0&-1\\
0&1&0
\end{bmatrix},
\qquad
R_y=
\begin{bmatrix}
0&0&1\\
0&1&0\\
-1&0&0
\end{bmatrix}.
$$

作用于 $\hat{z}=(0,0,1)^T$ 时，

$$
R_xR_y\hat{z}=\hat{x},
\qquad
R_yR_x\hat{z}=-\hat{y}.
$$

结果不同，所以 $R_xR_y\ne R_yR_x$。

二维旋转是特殊例外：所有旋转都绕同一根垂直于平面的轴进行，只需把角度相加，$\alpha+\beta=\beta+\alpha$，所以 $SO(2)$ 中的旋转彼此可交换。

##### 旋转保持向量长度

旋转只改变向量的方向，不改变其长度。若

$$
y=Rx,
$$

则通过平方范数可以直接证明：

$$
\begin{aligned}
\|y\|^2
&=y^Ty\\
&=(Rx)^T(Rx)\\
&=x^TR^TRx\\
&=x^Tx\\
&=\|x\|^2.
\end{aligned}
$$

由于长度非负，因此

$$
\|Rx\|=\|x\|.
$$

推导中的关键一步是使用旋转矩阵的正交条件 $R^TR=I$。直觉上，$R$ 只是把向量刚性转向，没有拉伸或压缩它。

同一推理还说明旋转保持两向量之间的点积：

$$
(Rx)^T(Rz)=x^TR^TRz=x^Tz,
$$

所以旋转也保持夹角。

#### 3.2.1.2 用途一：表示朝向

旋转矩阵有三种主要用途：表示朝向、改变坐标表示，以及实际旋转向量或参考系。Figure 3.7 先帮助我们理解第一种，它是最根本的几何含义

![教材图 3.7：同一点在三个不同朝向参考系中的表示](../../attachments/ch03/fig-3-7-three-oriented-frames.png)

*来源：教材 Figure 3.7，书页 72／本地 PDF 第 91 页；局部截取。*

图中的 $\{a\},\{b\},\{c\}$ 表示同一个三维空间，原点相同但朝向不同。图上分开绘制只是为了避免坐标轴重叠；黑点 $p$ 始终是同一个物理点。固定参考系 $\{s\}$ 没有画出，并与 $\{a\}$ 对齐。

表示朝向时，记号

$$
R_{sc}
$$

表示“参考系 $\{c\}$ 相对于参考系 $\{s\}$ 的朝向”。下标读取规则是：

> 第二个下标指出被描述的参考系，第一个下标指出用来描述它的参考系。

c 的三根轴，用 s 坐标系表示。

因此：

- $R_{sc}$：$\{c\}$ 相对 $\{s\}$ 的朝向；
- $R_{bc}$：$\{c\}$ 相对 $\{b\}$ 的朝向；
- $R_{cb}$：$\{b\}$ 相对 $\{c\}$ 的朝向。

交换两个下标意味着反向描述同一对参考系，所以

$$
R_{cb}=R_{bc}^{-1}=R_{bc}^T.
$$

在这一用途下，$R$ 被看成一个参考系朝向的“数据表示”，而不是正在对某个物理对象执行旋转。


##### 用途二：改变坐标表示

同一个矩阵 $R_{ab}$ 也可以看成坐标转换 算子：它把用 $\{b\}$ 表示的坐标转换成用 $\{a\}$ 表示的坐标。 该表同一个向量的坐标系表示

对一个物理点 $p$，若其 $\{b\}$ 坐标为 $p_b$，则

$$
p_a=R_{ab}p_b.
$$

这里物理点没有移动，改变的只是它的数值表示。下标可以像单位一样检查方向：

$$
R_{a\cancel{b}}p_{\cancel{b}}=p_a.
$$

同样，如果 $R_{bc}$ 描述 $\{c\}$ 相对 $\{b\}$ 的朝向，那么将它改写成相对 $\{a\}$ 的朝向需要

$$
R_{ac}=R_{ab}R_{bc}.
$$

中间相邻的 $b$ 下标消去，留下外侧的 $a,c$：

$$
R_{a\cancel{b}}R_{\cancel{b}c}=R_{ac}.
$$

这个下标消去规则不仅帮助记忆乘法顺序，也能检查维度正确但参考系方向错误的公式。若中间下标不能相邻匹配，就不能直接这样相乘完成坐标转换。

##### 用途三：主动旋转参考系

![教材图 3.8：围绕单位轴主动旋转坐标系](../../attachments/ch03/fig-3-8-rotation-about-axis.png)

*来源：教材 Figure 3.8，书页 74／本地 PDF 第 93 页；局部截取。*

这一用途把旋转矩阵 看成**真正施加的旋转操作**，而不是把同一对象改用另一组坐标写出来。图中初始坐标系的轴为 $\{\hat{x},\hat{y},\hat{z}\}$；

让它围绕单位旋转轴（unit rotation axis）$\hat{\omega}$ 转过角度 $\theta$ 后，得到浅灰色的新坐标系 $\{\hat{x}',\hat{y}',\hat{z}'\}$。图中特别画出 $\hat{y}$ 与 $\hat{y}'$ 重合，而 $\hat{x},\hat{z}$ 则转到了新方向。

把这个操作记为

$$
R=\operatorname{Rot}(\hat{\omega},\theta).
$$

它的整体意思是：从与原坐标系对齐的朝向 $I$ 出发，绕空间中的单位轴 $\hat{\omega}$ 主动转动 $\theta$，得到由 $R$ 表示的最终朝向。这里 $\hat{\omega}$ 给出旋转轴的方向，$\theta$ 的正负由右手定则决定；$R$ 的三列分别就是最终 $\hat{x}',\hat{y}',\hat{z}'$ 用原坐标系表示的坐标。

与用途二的分界是：

- **用途二**：点或向量在空间中没有动；$p_a=R_{ab}p_b$ 只替换数值表示所用的坐标轴。
- **用途三**：参考系（或向量）在空间中实际改变朝向；$R=\operatorname{Rot}(\hat{\omega},\theta)$ 描述这一几何旋转的结果。

同一个数值矩阵可以出现在两种叙述中，但必须先说清楚：它是在转换坐标表示，还是在描述或施加真实的旋转。

##### 绕坐标轴的基本旋转

Figure 3.8 中若旋转轴恰好选为原坐标系的一根单位轴，教材把主动旋转写成三种基本矩阵：

$$
\operatorname{Rot}(\hat{x},\theta)=
\begin{bmatrix}
1 & 0 & 0\\
0 & \cos\theta & -\sin\theta\\
0 & \sin\theta & \cos\theta
\end{bmatrix},
\qquad
\operatorname{Rot}(\hat{y},\theta)=
\begin{bmatrix}
\cos\theta & 0 & \sin\theta\\
0 & 1 & 0\\
-\sin\theta & 0 & \cos\theta
\end{bmatrix},
$$

$$
\operatorname{Rot}(\hat{z},\theta)=
\begin{bmatrix}
\cos\theta & -\sin\theta & 0\\
\sin\theta & \cos\theta & 0\\
0 & 0 & 1
\end{bmatrix}.
$$

它们的目的，是把“绕指定坐标轴转 $\theta$”变成可计算的线性操作。每个矩阵保留旋转轴本身：例如 $\operatorname{Rot}(\hat{z},\theta)\hat{z}=\hat{z}$，所以第三列为 $(0,0,1)^T$；其余两根轴在与旋转轴垂直的平面内按**右手定则**旋转。这也是二维平面旋转矩阵嵌入三维后的形式。

##### 任意单位轴的轴角表示

不必把旋转轴限制为 $\hat{x},\hat{y},\hat{z}$。对任意单位向量

$$
\hat{\omega}=(\hat{\omega}_1,\hat{\omega}_2,\hat{\omega}_3),
\qquad
\|\hat{\omega}\|=1,
$$

教材用同一个记号 $\operatorname{Rot}(\hat{\omega},\theta)$ 表示绕它旋转 $\theta$。它的意义比逐项公式更重要：任何 $R\in SO(3)$ 都可以看成从单位矩阵 $I$ 出发，围绕某条单位轴转过某个角度得到的朝向。

同一个旋转也可写成

$$
\operatorname{Rot}(\hat{\omega},\theta)
=
\operatorname{Rot}(-\hat{\omega},-\theta).
$$

原因是同时反转旋转轴方向与角度正负，按照右手定则产生的实际转向不变。这个等价性说明轴角参数化（axis-angle parameterization）并非对每个旋转都唯一。

##### 空间轴与物体轴旋转

![教材图 3.9：固定参考系轴与物体参考系轴的旋转](../../attachments/ch03/fig-3-9-fixed-and-body-frame-rotation.png)

*来源：教材 Figure 3.9，书页 75／本地 PDF 第 94 页；局部截取。*

设 $R_{sb}$ 表示物体参考系 $\{b\}$ 相对空间参考系 $\{s\}$ 的当前朝向，且 $R=\operatorname{Rot}(\hat{\omega},\theta)$。同一个数值矩阵 $R$ 放在不同一侧，描述的是不同的几何操作：

- **空间参考系旋转（fixed-frame rotation）**：旋转轴 $\hat{\omega}_s$ 用固定的 $\{s\}$ 表示，绕这条空间中不随物体移动的轴转动。新朝向为

  $$
  R_{sb'}=R R_{sb}.
  $$

- **物体参考系旋转（body-frame rotation）**：旋转轴 $\hat{\omega}_b$ 用随物体转动的 $\{b\}$ 表示，轴会随物体而改变其空间方向。新朝向为

  $$
  R_{sb''}=R_{sb}R.
  $$

判断是哪个参考系旋转：
	主要看旋转轴是用哪个坐标系表示的，也就是等价的看旋转矩阵R是乘在Rsb的坐标还是右边

左乘是固定空间旋转

假设现在有一个物体坐标系 {b}，它当前相对于空间坐标系 {s\} 的朝向是 $R_{sb}$

也就是：

> b 的三根轴，用 s 坐标表示。

当你在左边乘一个旋转矩阵 \(R\) 时：$R_{sb}'=R\,R_{sb}$

实际上是在对这三根“已经用 s 坐标表示的向量”同时做一次旋转 R。

>把 b 的三根轴，都按照一个**在 s 坐标系中定义的旋转 R** 去转。


右乘：

$R_{sb}'=R_{sb}R$

为什么表示绕自身轴？

因为右边的 R 不是直接对 s 坐标中的三根向量做旋转，而是在重新组合 $R_{sb}$ 的三列。

+ 左乘：直接在“世界坐标”里转这些向量
+ 右乘：在“当前自身基底”里重新组合三根轴

在 Figure 3.9 中，上支路绕 $\hat{z}_s$ 转 $90^\circ$，所以左乘并得到 $\{b'\}$；下支路绕 $\hat{z}_b$ 转 $90^\circ$，所以右乘并得到 $\{b''\}$。当 $\{s\}$ 和 $\{b\}$ 未对齐时，即使两个矩阵都写成同一数值 $R$，它们指向的空间旋转轴也不同，因此最终朝向通常不同。


##### 主动旋转一个向量

若要主动旋转的是向量（vector）$v$，而不是某个参考系的朝向，则只涉及一个参考系。把旋转轴 $\hat{\omega}$、向量 $v$ 和旋转后的向量 $v'$ 都用这同一个参考系表示，旋转为

$$
v'=Rv,
\qquad
R=\operatorname{Rot}(\hat{\omega},\theta).
$$

这个式子的目的，是直接计算向量经过真实旋转后的新方向；它与用途二的 $p_a=R_{ab}p_b$ 不同，后者并没有移动物理对象。此处不存在“左乘还是右乘 $R_{sb}$”的选择，因为没有第二个参考系朝向矩阵：$R$ 直接作用在列向量 $v$ 上。 虽然他们样子一样，但一个是旋转向量，一个是改变了向量的坐标参考系

### 3.2.2 角速度（Angular Velocities）

![教材图 3.10：瞬时角速度与坐标轴导数](../../attachments/ch03/fig-3-10-instantaneous-angular-velocity.png)

*来源：教材 Figure 3.10，书页 76／本地 PDF 第 95 页；局部截取。*

对于固连在转动刚体上的一组单位轴 $\{\hat{x},\hat{y},\hat{z}\}$，比较时刻 $t$ 与 $t+\Delta t$：它们之间的朝向差可视为绕某条瞬时单位轴 $\hat{\omega}$ 转过小角度 $\Delta\theta$。当 $\Delta t\to 0$ 时，$\Delta\theta/\Delta t$ 变成角速度大小 $\dot{\theta}$（标量），并定义角速度（angular velocity）向量

$$
\omega=\hat{\omega}\dot{\theta}.
$$

这一定义把两个信息合成一个向量：方向 $\hat{\omega}$ 是瞬时旋转轴，长度 $\|\omega\|=|\dot{\theta}|$ 是角速度大小，方向正负仍由右手定则决定。

此时 $\hat{\omega}$ 还是几何空间中的轴，尚未被写成某个特定参考系的坐标。

由于每根单位轴的长度恒为 $1$，它们只能改变方向，不能沿自身方向伸缩。教材给出三个导数关系：

$$
\dot{\hat{x}}=\omega\times\hat{x},
\qquad
\dot{\hat{y}}=\omega\times\hat{y},
\qquad
\dot{\hat{z}}=\omega\times\hat{z}.
$$
- $\hat{x}$：单位 x 轴方向向量
- $\dot{\hat{x}}$：这个单位向量随时间的变化率
- $\times$：叉积符号，不是字母 \(x\)

角速度向量 $\omega$ 叉乘单位方向向量 $\hat{x}$

其中 $\times$ 是叉积（cross product）。每个导数都垂直于相应坐标轴：它表示单位轴端点在绕 $\omega$ 扫过圆周时的切向瞬时速度；若某根轴恰好与 $\omega$ 平行，则它的叉积为零，方向不变。公式中的 $\omega$ 和被求导的坐标轴必须用同一参考系表示后才能进行叉积。

叉乘计算的两个特别重要的性质：

第一，方向：

$a\times b$ 一定同时垂直于 a 和 b。

第二，大小：

$\|a\times b\| = \|a\|\|b\|\sin\theta$

其中 $\theta$ 是 \(a,b\) 的夹角。


==所以上面叉乘公式，可以得到向量的方向和大小==


圆周运动公式：

$$\boxed{ v=\omega r\sin\theta }$$

- $\omega$：角速度向量，方向就是旋转轴方向(不是切线的那种)，当加了一个hat，它的大小就变成了1
- r：旋转轴上的某个参考点，指向运动点的位置向量
- v：这个运动点的瞬时线速度
- $\theta$ 是 $\omega$ 和 r 的夹角

#### 用空间坐标表示角速度

令 $R(t)=R_{sb}(t)$，其三列 $r_1,r_2,r_3$ 分别是物体轴 $\hat{x},\hat{y},\hat{z}$ 用固定空间参考系 $\{s\}$ 表示的坐标。

$R= \begin{bmatrix} r_1&r_2&r_3 \end{bmatrix}$

也就是说：

> (r1,r2,r3) 分别是物体坐标系 {b} 的三根轴，在空间坐标系 {s} 中的坐标。

现在物体正在转，所以这三根轴的方向都会随时间变化。因此：

$r_1=r_1(t),\qquad r_2=r_2(t),\qquad r_3=r_3(t)$

对时间求导：

$\dot r_1,\qquad \dot r_2,\qquad \dot r_3$

就表示：

> 三根轴的方向分别以多快的速度在变化。


若 $\omega_s$ 是同一个几何角速度 $\omega$ 在 $\{s\}$ 中的坐标，则逐列有

$$
\dot r_i=\omega_s\times r_i,
\qquad i=1,2,3.
$$
（这是叉乘）

把三列合并，就得到旋转矩阵的变化率：

$$
\dot R
=
[\omega_s\times r_1\ \ \omega_s\times r_2\ \ \omega_s\times r_3].
$$

#### 反对称矩阵与叉积

为把叉积写成普通矩阵乘法，对 $x=(x_1,x_2,x_3)^T$ 定义一个反对称矩阵（skew-symmetric matrix）

$$
[x]=
\begin{bmatrix}
0 & -x_3 & x_2\\
x_3 & 0 & -x_1\\
-x_2 & x_1 & 0
\end{bmatrix}.
$$

方括号不是“向量的坐标”，而是把向量 $x$ 变成一个线性算子，使得

$$
[x]y=x\times y.
$$

引入了反对称矩阵，乘这个反对称矩阵的操作 就等价于上面对 $x=(x_1,x_2,x_3)^T$做叉乘，它把“叉乘”改写成普通矩阵乘法，快速求得坐标轴在旋转过程中的变化率

$[x]$ 代表“拿 \(x\) 去叉乘别的向量” 的这个


因此 $[x]^T=-[x]$；所有 $3\times3$ 实反对称矩阵组成的集合记为 $\mathfrak{so}(3)$。用 $\omega_s$ 的反对称矩阵可将上式简写为

$$
\dot R=[\omega_s]R,
\qquad
[\omega_s]=\dot R R^{-1}=\dot R R^T.
$$

这说明：在固定空间参考系中，$\dot R$ 等于“空间角速度的叉积算子”左乘 当前朝向。

空间角速度的叉积算子”是：$\boxed{[\omega_s]}$

它是由角速度向量 $\omega_s$变成的反对称矩阵。

"叉积算子”其实就是把“和 $\omega$  做叉乘”这件事，写成一个矩阵操作


#### 用物体坐标表示同一个角速度

同一个几何角速度也可用物体参考系表示为 $\omega_b$。它们只是坐标表示不同，因而由下标消去规则有

$$
\omega_s=R_{sb}\omega_b,
\qquad
\omega_b=R_{sb}^T\omega_s.
$$
相应地可以推出：

从空间角速度公式开始：（这里的R和上一小节的R 其实就是 Rsb，在空间坐标刚开始就介绍了）

$$\dot R=[\omega_s]R$$
对于旋转矩阵 R，有一个很重要的恒等式：

$$[R\omega_b]=R[\omega_b]R^T$$
而因为

$$\omega_s=R\omega_b$$

所以：

$$[\omega_s] = R[\omega_b]R^T$$
代入原来的：

$$\dot R=[\omega_s]R$$

可得到：
$$
[\omega_b]=R^{-1}\dot R=R^T\dot R,
\qquad
\dot R=R[\omega_b].
$$

把两种写法并列最清楚：

$$
\boxed{\dot R=[\omega_s]R=R[\omega_b].}
$$

左式中的角速度以固定空间参考系表示，所以在左侧作用；右式中的角速度以物体参考系表示，所以在右侧作用。它们不是两个角速度，而是同一个 $\omega$ 的两种坐标数组。




### 3.2.3 旋转的指数坐标表示（Exponential Coordinate Representation of Rotation）

目的是：
$$\boxed{\text{把“绕某根轴转 }\theta\text{”写成一个矩阵指数，也就是旋转矩阵R}}$$

之前部分也出现过旋转矩阵，二维平面旋转矩阵，三维旋转矩阵... 但他们直接从两个坐标轴方向构造 R ，已经知道 {b} 的三根轴在 {s} 中分别朝哪里，那就直接把三列拼起来。

然后上一节，研究 **旋转矩阵R 随时间怎么变化**：$\dot R=[\omega_s]R$

此时是 我知道“绕轴 $\hat\omega$ 转了角度 $\theta$”，但不知道最终三根轴坐标，要怎么直接得到最终 R？


一个三维旋转可由单位旋转轴 $\hat{\omega}$ 与绕它转过的角度 $\theta$ 描述；把两者相乘得到三维向量

$$
\hat{\omega}\theta\in\mathbb{R}^3,
\qquad
\|\hat{\omega}\|=1.
$$

这个向量称为旋转的指数坐标（exponential coordinates）。把 $\hat{\omega}$ 与 $\theta$ 分开写，称为轴角表示（axis-angle representation）。它用三个数而不是旋转矩阵的九个元素描述旋转，同时保留“绕哪条轴、转了多少”的几何意思。

假设旋转轴是单位向量：

$$\hat\omega= \begin{bmatrix} \omega_1\\ \omega_2\\ \omega_3 \end{bmatrix}, \qquad \|\hat\omega\|=1$$

绕这根轴旋转角度 $\theta$。

那么把“轴方向”和“旋转角度”合起来：
$$\hat\omega\theta$$

这个三维向量就叫**旋转的指数坐标**。

已经给我指数坐标 $\hat\omega\theta$ 了，怎么把它转换成旋转矩阵 R ？

对同一 $\hat{\omega}\theta$，教材给出三种等价理解：

1. 初始与 $\{s\}$ 对齐的参考系，绕以 $\{s\}$ 表示的 $\hat{\omega}$ 旋转 $\theta$ 后，其最终朝向由 $R$ 表示；
2. 同一初始参考系以恒定角速度 $\hat{\omega}\theta$ 运动一个单位时间，最终朝向仍为 $R$；
3. 同一初始参考系以恒定单位角速度 $\hat{\omega}$ 运动 $\theta$ 时间，最终朝向也为 $R$。

后两种理解把“有限旋转”连接到 常系数线性微分方程，引入时间的概念。


#### 3.2.3.1 矩阵指数（Matrix Exponential）

常规标量方程

$$
\dot{x}(t)=a x(t),
\qquad
x(0)=x_0,
$$

解为 $x(t)=e^{at}x_0$

其向量版本为

$$
\dot{x}(t)=Ax(t),
\qquad
x(0)=x_0,
\qquad
x(t)=e^{At}x_0,
$$
这时：

- x(t)是一个向量；
- A 是一个矩阵。


其中矩阵指数（matrix exponential）定义为

$$
e^{At}
=
I+At+\frac{(At)^2}{2!}+\frac{(At)^3}{3!}+\cdots.
$$

这里 $e^{At}$ 不是把矩阵每个元素分别取指数，而是上述矩阵幂级数。

它之所以即将用于旋转，是因为角速度导出的 $\dot R$ 正好是常系数线性微分方程的形式；下一小段会令这个常矩阵与 $[\hat{\omega}]$ 联系起来。

对有限常矩阵 $A$，上述级数一定收敛。逐项求导可验证

$$
\frac{d}{dt}e^{At}
=
Ae^{At}
=
e^{At}A,
$$

所以

$$
\frac{d}{dt}\left(e^{At}x_0\right)
=
Ae^{At}x_0
=
Ax(t).
$$

这里 $Ae^{At}=e^{At}A$ 成立，是因为 $e^{At}$ 只含 $A$ 的各次幂；它不表示任意两个矩阵都可以交换。


引入微分方程，就是为了后序 已知“绕哪根轴转、转多少角度”，求最终旋转矩阵 。

从旋转的指数坐标:  $\hat\omega\theta$

求旋转矩阵：

$\hat\omega\theta \quad\longrightarrow\quad R$


##### Proposition 3.10：矩阵指数的完整性质

对常矩阵 $A\in\mathbb{R}^{n\times n}$，初值问题

$$
\dot{x}(t)=Ax(t),
\qquad
x(0)=x_0
$$

的唯一解为

$$
x(t)=e^{At}x_0.
$$

教材同时给出以下四项性质：

1. **导数**

   $$
   \frac{d}{dt}e^{At}
   =
   Ae^{At}
   =
   e^{At}A.
   $$

2. **相似变换**

   若 $A=PDP^{-1}$，则

   $$
   e^{At}=Pe^{Dt}P^{-1}.
   $$

   若 $D=\operatorname{diag}(d_1,\ldots,d_n)$，则

   $$
   e^{Dt}
   =
   \operatorname{diag}\!\left(e^{d_1t},\ldots,e^{d_nt}\right).
   $$

   这项性质把难算的矩阵指数转换为较简单矩阵 $D$ 的指数。

3. **可交换矩阵的指数和**

   若 $AB=BA$，则保证下式成立（这是充分条件）：

   $$
   e^Ae^B=e^{A+B}.
   $$

   三维旋转矩阵通常不交换，因此后续不能无条件套用标量指数律。

4. **逆矩阵**

   $$
   \left(e^A\right)^{-1}=e^{-A}.
   $$

   因为 $A$ 与 $-A$ 可交换，故 $e^Ae^{-A}=e^0=I$。后续这会对应“将旋转角度取反即可撤销旋转”。

这一组结果建立了后续推导所需的工具链：常角速度给出常系数矩阵微分方程，矩阵指数给出有限时间后的朝向，而其逆与可交换条件决定旋转公式能够怎样化简。


#### 3.2.3.2 从指数坐标到旋转矩阵

![教材 Figure 3.11：绕单位轴旋转向量](../../attachments/ch03/fig-3-11-exponential-rotation.png)

*来源：书页 83／本地 PDF 第 102 页；局部截取。本单元覆盖书页 82—84，式 (3.49)—(3.51) 与 Proposition 3.11；Example 3.12 留至下一单元。*

这里的p是一个被旋转的向量

图中所有量都用固定参考系表示。旋转轴 $\hat{\omega}$ 与原点固定，$p(0)$ 绕轴转过 $\theta$ 成为 $p(\theta)$。$\phi$ 是向量与轴的夹角，保持不变；端点轨迹的半径为 $\|p\|\sin\phi$。$\theta$ 是绕轴扫过的角度，一般不等于两个位置向量的夹角。

设恒定角速率为 $1\,\mathrm{rad/s}$。端点切向速度的大小为 $\|p\|\sin\phi$，方向由右手定则给出，因此教材在单位速率约定下写成

$$
\dot p=\hat{\omega}\times p=[\hat{\omega}]p.
$$

对omega做叉乘，可以转换成左乘一个反对称矩阵

更明确地以角度为自变量，可写为 $dp/d\theta=[\hat{\omega}]p$；利用 Proposition 3.10，

$$
p(\theta)=e^{[\hat{\omega}]\theta}p(0).
$$

下面就是求解指数部分，它也就是我们想求的旋转矩阵 R


假设最初坐标系三个单位轴是：

$$p_1(0)=\hat x,\qquad p_2(0)=\hat y,\qquad p_3(0)=\hat z$$

分别对它们应用：

$$p_i(\theta)=e^{[\hat\omega]\theta}p_i(0)$$

就得到旋转后的三根坐标轴。

把这三根轴拼起来：

$$R= \begin{bmatrix} p_1(\theta)&p_2(\theta)&p_3(\theta) \end{bmatrix}$$

所以教材先研究“一个向量 p 怎么转”，最后自然就得到“整个坐标系 怎么转”。 这里拼出来的也是R。这是R的两种算法


$p(\theta)=e^{[\hat{\omega}]\theta}p(0)$   说明矩阵指数描述的恰好是绕固定轴的有限旋转。接下来把无穷级数化为可计算的表达式。令 $W=[\hat{\omega}]$，教材使用 $W^3=-W$；以下补出其推导。

向量三重积恒等式（vector triple product identity）为

$$
a\times(b\times v)=b(a^Tv)-v(a^Tb).
$$

令 $a=b=\hat{\omega}$，并使用 $\|\hat{\omega}\|=1$，得到

$$
W^2v=\hat{\omega}(\hat{\omega}^Tv)-v,
\qquad
W^2=\hat{\omega}\hat{\omega}^T-I.
$$

因为 $W\hat{\omega}=\hat{\omega}\times\hat{\omega}=0$，所以

$$
W^3=W(\hat{\omega}\hat{\omega}^T-I)=-W.
$$

继而 $W^4=-W^2,\ W^5=W$。将矩阵指数的奇次幂与偶次幂分别合并：

$$
\begin{aligned}
e^{W\theta}
&=I+\left(\theta-\frac{\theta^3}{3!}+\frac{\theta^5}{5!}-\cdots\right)W\\
&\quad+\left(\frac{\theta^2}{2!}-\frac{\theta^4}{4!}+\frac{\theta^6}{6!}-\cdots\right)W^2\\
&=I+\sin\theta\,W+(1-\cos\theta)W^2.
\end{aligned}
$$

这就是 Proposition 3.11 的罗德里格斯公式（Rodrigues' formula），式 (3.51)：

$$
\boxed{\operatorname{Rot}(\hat{\omega},\theta)
=e^{[\hat{\omega}]\theta}
=I+\sin\theta[\hat{\omega}]+(1-\cos\theta)[\hat{\omega}]^2\in SO(3).}
$$

输入是单位轴和角度，输出是旋转矩阵。将同一个公式用于 $R_0$ 的三列，得到空间轴旋转 $R'=e^{W\theta}R_0$；若轴用物体参考系表示，则 $R''=R_0e^{W\theta}$。这延续了前面左乘/右乘的含义。

一定要是单位旋转轴，不然得先进行归一化

罗德里格斯公式 :
从“理论形式”

$$R=e^{[\hat\omega]\theta}$$

推到“可计算形式”

$$\boxed{ R = I+\sin\theta[\hat\omega] +(1-\cos\theta)[\hat\omega]^2 }$$

从而计算出旋转矩阵R

标准 Rodrigues 公式记成：

$$\boxed{ \text{单位轴 }\hat\omega + \text{旋转角 }\theta \Rightarrow R }$$

如果手里拿到的是一个一般向量 \(\omega\)，就先拆成：

$$\omega = \hat\omega\,\|\omega\|$$

再决定 $\|\omega\|$ 到底是“轴向量的长度”还是“角速度大小”，不能直接混进 $\theta$ 里。


例如 $\hat{\omega}=(0,0,1)^T$ 时，

$$
W=\begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix},
\quad W^2=\begin{bmatrix}-1&0&0\\0&-1&0\\0&0&0\end{bmatrix}.
$$

代入即得到熟悉的绕 $z$ 轴矩阵：

$$
e^{W\theta}=
\begin{bmatrix}
\cos\theta&-\sin\theta&0\\
\sin\theta&\cos\theta&0\\
0&0&1
\end{bmatrix}.
$$

当 $\theta=\pi/2$ 且 $p(0)=(1,0,1)^T$ 时，

$$
p(\pi/2)
=
\begin{bmatrix}
0&-1&0\\
1&0&0\\
0&0&1
\end{bmatrix}
\begin{bmatrix}1\\0\\1\end{bmatrix}
=
\begin{bmatrix}0\\1\\1\end{bmatrix}.
$$

$z$ 分量不变，因为它是沿旋转轴的分量；旋转只改变垂直于轴的 $xy$ 平面分量。



##### Example 3.12：从轴角表示计算旋转矩阵

![教材 Figure 3.12：绕指定单位轴旋转得到物体参考系](../../attachments/ch03/fig-3-12-axis-angle-example.png)

*来源：教材 Figure 3.12，书页 85／本地 PDF 第 104 页；局部截取。例题计算位于书页 84／PDF 第 103 页。*

物体参考系 $\{b\}$ 起初与固定参考系 $\{s\}$ 重合，然后绕单位轴

$$
\hat\omega_1=
\begin{bmatrix}0\\0.866\\0.5\end{bmatrix},
\qquad
\theta_1=30^\circ=0.524\ \mathrm{rad}
$$

旋转。首先检查 $\|\hat\omega_1\|\approx1$，因此可以直接使用 Rodrigues 公式。对应的反对称矩阵为

$$
[\hat\omega_1]
=
\begin{bmatrix}
0&-0.5&0.866\\
0.5&0&0\\
-0.866&0&0
\end{bmatrix}.
$$

由于 $\sin30^\circ=0.5$、$1-\cos30^\circ\approx0.134$，

$$
\begin{aligned}
R
&=e^{[\hat\omega_1]\theta_1}\\
&=I+0.5[\hat\omega_1]+0.134[\hat\omega_1]^2\\
&\approx
\begin{bmatrix}
0.866&-0.250&0.433\\
0.250&0.967&0.058\\
-0.433&0.058&0.899
\end{bmatrix}.
\end{aligned}
$$

这个结果有两种等价表示：旋转矩阵 $R$，或轴角对 $(\hat\omega_1,\theta_1)$。相应的指数坐标为

$$
\hat\omega_1\theta_1
\approx
\begin{bmatrix}0\\0.453\\0.262\end{bmatrix}.
$$

注意指数坐标不是把 $\hat\omega_1$ 填进旋转矩阵；它先通过叉积映射变成 $[\hat\omega_1]\theta_1\in so(3)$，再取矩阵指数，才得到 $R\in SO(3)$。

例题后的正文再次强调乘法顺序。若接着绕固定参考系中的轴 $\hat\omega_2$ 转 $\theta_2$，则

$$
R'=e^{[\hat\omega_2]\theta_2}R.
$$

若同样三个数表示的是物体参考系中的轴，则

$$
R''=Re^{[\hat\omega_2]\theta_2}.
$$

当 $\hat\omega_2\ne\hat\omega_1$ 时通常 $R'\ne R''$：左乘会在固定坐标中旋转 $R$ 的每一列，右乘则把当前物体轴重新组合成新的列。这不是两种记号，而是两种不同的后续旋转。


#### 3.2.3.3 旋转的矩阵对数（Matrix Logarithm of Rotations）

上一小节解决了从指数坐标 $\hat\omega\theta$ 得到旋转矩阵 $R$ 的正向问题。本小节反过来：给定 $R\in SO(3)$，恢复产生它的旋转轴与角度。

##### 本节路线图

1. 将矩阵对数理解为矩阵指数的逆向映射。
2. 在一般情形 $0<\theta<\pi$ 下，由迹求角度、由反对称部分求旋转轴。
3. 单独处理 $\theta=0$ 和 $\theta=\pi$ 的特殊情形，再形成完整算法。

矩阵指数与矩阵对数（matrix logarithm）的方向分别是

$$
\exp:[\hat\omega]\theta\in so(3)\longrightarrow R\in SO(3),
$$

$$
\log:R\in SO(3)\longrightarrow[\hat\omega]\theta\in so(3).
$$

可以把矩阵指数理解为：把恒定角速度的矩阵表示“积分”一个单位时间，得到最终朝向。矩阵对数则反过来，从最终朝向恢复一组恒定角速度的矩阵表示，使其积分一个单位时间后产生该朝向。

严格地说，一个旋转矩阵可以有多个矩阵对数，因为角度加上 $2k\pi$ 后可能表示同一朝向。教材后续把 $\theta$ 限制在 $[0,\pi]$，并用算法选出一个标准结果；这里先研究没有奇异性的情形 $0<\theta<\pi$。

##### 一般情形：由 $R$ 求旋转角

令 $W=[\hat\omega]$。由前面已经证明的

$$
W^2=\hat\omega\hat\omega^T-I
$$

可知 $\operatorname{tr}W=0$，并且

$$
\operatorname{tr}(W^2)
=\operatorname{tr}(\hat\omega\hat\omega^T)-\operatorname{tr}I
=\hat\omega^T\hat\omega-3
=-2.
$$

因此对 Rodrigues 公式取迹，就得到式 (3.54)：

$$
\operatorname{tr}R
=r_{11}+r_{22}+r_{33}
=1+2\cos\theta.
$$

这里 $\operatorname{tr}R$ 是矩阵主对角线元素之和。它的作用是先从整个旋转矩阵中提取旋转角：

$$
\boxed{
\theta
=
\cos^{-1}\!\left(\frac{\operatorname{tr}R-1}{2}\right)
},
\qquad 0<\theta<\pi.
$$

直觉上，旋转角越接近 $0$，$R$ 越接近 $I$，其迹越接近 $3$；旋转角达到 $\pi$ 时，迹降到 $-1$。

##### 一般情形：由 $R$ 求旋转轴

因为 $W^T=-W$ 且 $(W^2)^T=W^2$，Rodrigues 公式的转置为

$$
R^T=I-\sin\theta\,W+(1-\cos\theta)W^2.
$$

把 $R$ 与 $R^T$ 相减，$I$ 和 $W^2$ 这些对称项抵消，只剩下由旋转轴产生的反对称部分：

$$
R-R^T=2\sin\theta[\hat\omega].
$$

只要 $\sin\theta\ne0$，也就是当前限定的 $0<\theta<\pi$，式 (3.53) 给出

$$
\boxed{
[\hat\omega]
=
\frac{1}{2\sin\theta}(R-R^T)
}.
$$

用反对称矩阵的三个独立元素读回向量，可写成

$$
\boxed{
\hat\omega
=
\frac{1}{2\sin\theta}
\begin{bmatrix}
r_{32}-r_{23}\\
r_{13}-r_{31}\\
r_{21}-r_{12}
\end{bmatrix}
}.
$$

因此一般情形的顺序是：先用 $\operatorname{tr}R$ 求 $\theta$，再用 $R-R^T$ 求 $\hat\omega$。最后得到指数坐标 $\hat\omega\theta$，以及矩阵对数

$$
\boxed{
\log R
=[\hat\omega]\theta
=
\frac{\theta}{2\sin\theta}(R-R^T)
},
\qquad 0<\theta<\pi.
$$

这个公式不能直接用于 $\theta=0$ 或 $\theta=\pi$，因为此时 $\sin\theta=0$。这不是可以忽略的代数小问题，而是三参数旋转表示不可避免的奇异情形；教材接下来会分别处理它们。



##### 特殊情形一：$\theta=0$

若 $R=I$，则没有发生旋转，因此教材取

$$
\theta=0,
\qquad
\hat\omega\text{ 未定义},
\qquad
[\hat\omega]\theta=0.
$$

旋转轴未定义并不是缺少答案：零角度绕任何轴都得到 $I$，所以最终朝向无法反推出唯一的轴。矩阵对数算法返回零矩阵即可。


##### 特殊情形二：$\theta=\pi$

当旋转角为 $\pi$ 时，$\operatorname{tr}R=-1$。由于 $\sin\pi=0$、$\cos\pi=-1$，Rodrigues 公式化为式 (3.55)：

$$
R=e^{[\hat\omega]\pi}
=I+2[\hat\omega]^2
=2\hat\omega\hat\omega^T-I.
$$

因此

$$
R+I=2\hat\omega\hat\omega^T.
$$

它的对角元素和非对角元素分别给出式 (3.56)—(3.57)：

$$
\hat\omega_i
=\pm\sqrt{\frac{r_{ii}+1}{2}},
\qquad i=1,2,3,
$$

$$
2\hat\omega_1\hat\omega_2=r_{12},
\qquad
2\hat\omega_2\hat\omega_3=r_{23},
\qquad
2\hat\omega_1\hat\omega_3=r_{13}.
$$

由于平方根只能给出分量大小，还要用非对角元素协调各分量的符号。实际计算时，从下面三个等价候选式中选择分母非零、数值稳定的一项：

$$
\hat\omega
=
\frac{1}{\sqrt{2(1+r_{33})}}
\begin{bmatrix}
r_{13}\\r_{23}\\1+r_{33}
\end{bmatrix},
\tag{3.58}
$$

$$
\hat\omega
=
\frac{1}{\sqrt{2(1+r_{22})}}
\begin{bmatrix}
r_{12}\\1+r_{22}\\r_{32}
\end{bmatrix},
\tag{3.59}
$$

或


$$
\hat\omega
=
\frac{1}{\sqrt{2(1+r_{11})}}
\begin{bmatrix}
1+r_{11}\\r_{21}\\r_{31}
\end{bmatrix}.
\tag{3.60}
$$

并非三个公式都必须使用；哪个对应的 $1+r_{ii}$ 非零，就可以选哪个。对于 $\theta=\pi$，$\hat\omega$ 和 $-\hat\omega$ 表示同一个旋转，因为绕反向轴转 $\pi$ 与绕原轴转 $\pi$ 的最终朝向相同。

##### 完整的矩阵对数算法

教材将 $\theta$ 限制到 $[0,\pi]$，于是对任意 $R\in SO(3)$ 按以下三种情况计算：

1. **若 $R=I$**：取 $\theta=0$，旋转轴未定义，矩阵对数为零矩阵。
2. **若 $\operatorname{tr}R=-1$**：取 $\theta=\pi$，用式 (3.58)—(3.60) 中可行的一式求 $\hat\omega$。
3. **其他情况**：

   $$
   \theta
   =
   \cos^{-1}\!\left(\frac{\operatorname{tr}R-1}{2}\right),
   $$

   $$
   [\hat\omega]
   =
   \frac{R-R^T}{2\sin\theta}.
   \tag{3.61}
   $$

最后统一输出

$$
\hat\omega\theta\in\mathbb R^3,
\qquad
[\hat\omega]\theta=\log R\in so(3).
$$

这三个分支覆盖所有 $R\in SO(3)$。其中判断 $R=I$ 必须放在一般公式之前，因为它的迹为 $3$，虽然角度公式能得到 $0$，但求轴的公式会除以零。



##### Figure 3.13：$SO(3)$ 的指数坐标球

![教材 Figure 3.13：将 SO(3) 表示为半径 pi 的实心球](../../attachments/ch03/fig-3-13-so3-solid-ball.png)

*来源：教材 Figure 3.13，书页 88／本地 PDF 第 107 页；局部截取。相关说明延续至书页 89／PDF 第 108 页。*

把旋转的指数坐标记为

$$
r=\hat\omega\theta\in\mathbb R^3.
$$

因为 $\|\hat\omega\|=1$，所以

$$
\|r\|=\theta.
$$

因此这个三维向量同时编码两件事：从原点指向 $r$ 的方向给出旋转轴 $\hat\omega=r/\|r\|$，离原点的距离给出旋转角 $\theta=\|r\|$。

矩阵对数算法把 $\theta$ 选在 $[0,\pi]$，所以标准指数坐标满足

$$
\|r\|\le\pi.
$$


$$\boxed{\text{任意旋转都可以等价地改写成一个不超过 }180^\circ\text{ 的旋转}}$$

所以矩阵对数在选标准轴角表示时，只取：$0\le \theta\le \pi$

举个例子：

$\text{绕 }z\text{ 轴转 }270^\circ$

和

$\text{绕 }-z\text{ 轴转 }90^\circ$

最终姿态完全一样。



> **所有三维旋转，都可以用球内的一个点来表示。**

这个点就是指数坐标向量: $r=\hat\omega\theta$


所有标准指数坐标便落在一个半径为 $\pi$ 的实心球中：

- **球心 $r=0$**：对应 $R=I$。此时没有旋转，轴未定义，但指数坐标点就是唯一的原点。
- **球内 $0<\|r\|<\pi$**：每个旋转矩阵对应唯一的标准指数坐标；此时 $\operatorname{tr}R\ne-1$。
- **球面 $\|r\|=\pi$**：对应转角为 $\pi$、$\operatorname{tr}R=-1$ 的旋转。球面上的一对对径点 $r$ 与 $-r$ 表示同一个旋转，因为

  $$
  e^{[\hat\omega]\pi}
  =e^{[-\hat\omega]\pi}.
  $$

这里不是任意两个球面点都等价，只有互为相反数的对径点等价。因此，$SO(3)$ 更准确的图像是“半径为 $\pi$ 的实心球，并将球面上的每一对对径点视为同一点”。普通实心球的边界点彼此不同，而 $SO(3)$ 的这个边界具有额外的等价关系。


### 3.2 小结

- 旋转矩阵 $R\in SO(3)$ 描述三维朝向，满足 $R^TR=I$、$\det R=1$，并保持向量长度与夹角。
- 反对称矩阵 $[\omega]$ 把叉积写成矩阵乘法；同一个角速度满足 $\dot R=[\omega_s]R=R[\omega_b]$。
- 指数映射用 Rodrigues 公式把轴角或指数坐标变成旋转矩阵：$R=e^{[\hat\omega]\theta}$。
- 矩阵对数反向从 $R$ 恢复 $\theta$ 与 $\hat\omega$；$R=I$ 和 $\theta=\pi$ 必须使用特殊分支。
- 标准指数坐标 $r=\hat\omega\theta$ 位于半径为 $\pi$ 的实心球中，球面上的对径点表示同一个 $\pi$ 旋转。




## 3.3 刚体运动与扭旋（Rigid-Body Motions and Twists）

第 3.2 节只描述朝向和角速度；第 3.3 节把位置和平移速度加入进来，建立完整的三维刚体位姿与速度表示。

上一节学的核心对象是：$R\in SO(3)$

它只告诉你：

> 物体坐标系 \{b\} 相对空间坐标系 \{s\} **朝哪里**。


默认只关心“物体朝哪儿”，不关心“物体原点搬到哪里”

这一节加入位置：$p\in\mathbb R^3$

表示：

> b 坐标系原点在 s 坐标系中的位置。

### 本节路线图

1. 用齐次变换矩阵同时表示位置和朝向。
2. 学习刚体运动的速度表示：螺旋轴和扭旋。
3. 建立刚体运动的指数映射与矩阵对数。

第 3.2 节与第 3.3 节的对应关系是：

| 只描述旋转 | 描述完整刚体运动 |
|---|---|
| 旋转矩阵 $R$ | 齐次变换矩阵 $T$ |
| 旋转轴 $\hat\omega$ | 螺旋轴 $S$ |
| 角速度 $\omega=\hat\omega\dot\theta$ | 扭旋 $V=S\dot\theta$ |
| 指数坐标 $\hat\omega\theta\in\mathbb R^3$ | 指数坐标 $S\theta\in\mathbb R^6$ |

上一节：

$$\hat\omega\theta \longrightarrow R=e^{[\hat\omega]\theta}$$

$SO(3)$ 研究的是**纯旋转**：默认关注的是物体朝向怎么变

“绕某条轴转”，但不关心物体原点搬到哪里

这一节升级成：

$$S\theta \longrightarrow T=e^{[S]\theta}$$

$SE(3)$ 则研究完整刚体位姿

这里加入的 p 是真正的**位置平移**：物体坐标系原点相对空间坐标系原点在哪里

其实就和上面3.1节二维描述的类似

旋转一般情况会带来点的改变

但是如果旋转轴恰好经过原点，那就不需要额外平移项，只要考虑朝向就够了


### 3.3.1 齐次变换矩阵（Homogeneous Transformation Matrices）

三维刚体的构型需要同时说明两件事：物体参考系 $\{b\}$ 相对于固定参考系 $\{s\}$ 的朝向 $R\in SO(3)$，以及 $\{b\}$ 原点在 $\{s\}$ 中的位置 $p\in\mathbb R^3$。教材把它们打包进一个矩阵。

#### Definition 3.13：特殊欧氏群 $SE(3)$

特殊欧氏群（special Euclidean group）$SE(3)$，也称三维刚体运动群或三维齐次变换矩阵（homogeneous transformation matrices）的集合，定义为

$$
SE(3)
=
\left\{
T=
\begin{bmatrix}
R&p\\
0&1
\end{bmatrix}
\;\middle|\;
R\in SO(3),\ p\in\mathbb R^3
\right\}.
$$

展开后，

$$
T=
\begin{bmatrix}
r_{11}&r_{12}&r_{13}&p_1\\
r_{21}&r_{22}&r_{23}&p_2\\
r_{31}&r_{32}&r_{33}&p_3\\
0&0&0&1
\end{bmatrix}.
\tag{3.62}
$$

各块的含义是：

- **左上角 $R$**：物体参考系的朝向。
- **右上角 $p$**：物体参考系原点的位置。
- **最后一行 $[0\ 0\ 0\ 1]$**：把旋转和平移统一进矩阵运算所需的齐次结构，不代表增加了第四个空间维度。

一个元素也可写成 $T=(R,p)$。带参考系下标时，$T_{sb}=(R_{sb},p_{sb})$ 表示 $\{b\}$ 相对于 $\{s\}$ 的完整位姿。

#### Definition 3.14：平面特殊欧氏群 $SE(2)$

对于平面刚体，$R\in SO(2)$、$p\in\mathbb R^2$，对应的齐次变换矩阵为

$$
T=
\begin{bmatrix}
R&p\\
0&1
\end{bmatrix}
\in SE(2),
$$

也就是

$$
T=
\begin{bmatrix}
\cos\theta&-\sin\theta&p_1\\
\sin\theta&\cos\theta&p_2\\
0&0&1
\end{bmatrix},
\qquad
\theta\in[0,2\pi).
\tag{3.63}
$$

因此，三维刚体用 $4\times4$ 矩阵表示，平面刚体用 $3\times3$ 矩阵表示。矩阵尺寸多出的一行和一列是为了容纳平移，并不意味着物理空间升维。后续将看到，这种打包方式能让朝向变换和平移用一次矩阵乘法共同完成。

#### 3.3.1.1 齐次变换矩阵的性质（Properties of Transformation Matrices）

单位矩阵 $I$ 显然属于 $SE(3)$。下面前三条性质说明 $SE(3)$ 对逆、乘法和结合律封闭，因而构成一个群；第四条性质说明这些矩阵确实描述刚体运动。

##### Proposition 3.15：逆变换

若

$$
T=
\begin{bmatrix}
R&p\\
0&1
\end{bmatrix}
\in SE(3),
$$

则

$$
\boxed{
T^{-1}
=
\begin{bmatrix}
R^T&-R^Tp\\
0&1
\end{bmatrix}
}.
\tag{3.64}
$$

验证时直接相乘：

$$
\begin{bmatrix}R&p\\0&1\end{bmatrix}
\begin{bmatrix}R^T&-R^Tp\\0&1\end{bmatrix}
=
\begin{bmatrix}
RR^T&-RR^Tp+p\\
0&1
\end{bmatrix}
=
\begin{bmatrix}I&0\\0&1\end{bmatrix}.
$$

这里平移部分是 $-R^Tp$，而不是一般意义下的 $-p$。原因是撤销平移之前，还必须把位移改用反向参考系的坐标表示。

##### Proposition 3.16：乘法封闭

令 $T_1=(R_1,p_1)$、$T_2=(R_2,p_2)$，则

$$
\boxed{
T_1T_2
=
\begin{bmatrix}
R_1R_2&R_1p_2+p_1\\
0&1
\end{bmatrix}
\in SE(3)
}.
$$

新朝向为 $R_1R_2$；第二个位移 $p_2$ 必须先经 $R_1$ 改写到外层参考系，再加上 $p_1$。由于 $R_1R_2\in SO(3)$ 且 $R_1p_2+p_1\in\mathbb R^3$，结果仍是合法的齐次变换矩阵。

##### Proposition 3.17：结合但通常不交换

齐次变换矩阵继承普通矩阵乘法的结合律：

$$
(T_1T_2)T_3=T_1(T_2T_3).
$$

但通常

$$
T_1T_2\ne T_2T_1.
$$

结合律表示多个变换可以改变加括号的方式；非交换性表示不能随意改变动作顺序。旋转和平移的复合尤其依赖顺序。

##### 齐次坐标与点的变换

把三维点 $x\in\mathbb R^3$ 追加一个 $1$，得到它的齐次坐标（homogeneous coordinates）：

$$
\bar x=
\begin{bmatrix}x\\1\end{bmatrix}.
$$

于是“先旋转、再平移”可以写成一次矩阵乘法：

$$
\boxed{
T\bar x
=
\begin{bmatrix}R&p\\0&1\end{bmatrix}
\begin{bmatrix}x\\1\end{bmatrix}
=
\begin{bmatrix}Rx+p\\1\end{bmatrix}
}.
\tag{3.65}
$$

教材后续简写 $Tx$ 时，实际指三维结果 $Rx+p$，而不是说 $4\times4$ 矩阵可以直接乘三维列向量。

##### Proposition 3.18：保持距离和夹角

把 $T=(R,p)$ 看成对三维点的作用 $Tx=Rx+p$，则

$$
\boxed{
\|Tx-Ty\|=\|x-y\|
}.
$$

因为

$$
Tx-Ty=R(x-y),
$$

平移 $p$ 相消，而 $R^TR=I$ 保持向量长度。

对于任意公共顶点 $z$，还有

$$
\boxed{
\langle Tx-Tz,\ Ty-Tz\rangle
=
\langle x-z,\ y-z\rangle
}.
$$

证明是

$$
\begin{aligned}
\langle Tx-Tz,\ Ty-Tz\rangle
&=\langle R(x-z),\ R(y-z)\rangle\\
&=(x-z)^TR^TR(y-z)\\
&=(x-z)^T(y-z).
\end{aligned}
$$

所以刚体变换保持任意两点之间的距离，也保持由三点形成的夹角。一个三角形经过 $T$ 后，边长和内角都不变；这种关系称为等距（isometric）。这正是 $SE(3)$ 能表示刚体运动的几何原因。

#### 3.3.1.2 齐次变换矩阵的三种用途（Uses of Transformation Matrices）

与旋转矩阵相似，齐次变换矩阵有三种主要用途：

1. 表示刚体或参考系的完整位姿；
2. 改变点、向量或参考系的坐标表示；
3. 主动旋转并平移一个点、向量或参考系。

前两种用途首先借助 Figure 3.14 说明。第三种用途还需要 Figure 3.15，将在下一连续单元展开。

![教材 Figure 3.14：三个空间参考系与点 v](../../attachments/ch03/fig-3-14-three-spatial-frames.png)

*来源：教材 Figure 3.14，书页 92／本地 PDF 第 111 页；局部截取。*

图中 $\{a\}$ 与固定参考系 $\{s\}$ 重合；$\{b\}$、$\{c\}$ 位于不同位置并具有不同朝向。点 $v$ 在 $\{b\}$ 中的坐标为

$$
v_b=\begin{bmatrix}0\\0\\1.5\end{bmatrix}.
$$

##### 用途一：表示参考系的位姿

教材给出的 $\{a\}$、$\{b\}$、$\{c\}$ 相对于 $\{s\}$ 的数据包括

$$
R_{sa}=I,
\qquad
p_{sa}=\begin{bmatrix}0\\0\\0\end{bmatrix},
$$

$$
R_{sb}=
\begin{bmatrix}
0&0&1\\
0&-1&0\\
1&0&0
\end{bmatrix},
\qquad
p_{sb}=\begin{bmatrix}0\\-2\\0\end{bmatrix},
$$

$$
R_{sc}=
\begin{bmatrix}
-1&0&0\\
0&0&1\\
0&1&0
\end{bmatrix},
\qquad
p_{sc}=\begin{bmatrix}-1\\1\\0\end{bmatrix}.
$$

所以 $T_{sa}=I$，而

$$
T_{sb}=
\begin{bmatrix}
0&0&1&0\\
0&-1&0&-2\\
1&0&0&0\\
0&0&0&1
\end{bmatrix},
$$

$$
T_{sc}=
\begin{bmatrix}
-1&0&0&-1\\
0&0&1&1\\
0&1&0&0\\
0&0&0&1
\end{bmatrix}.
$$

$T_{sb}$ 的含义是：$\{b\}$ 相对于 $\{s\}$ 的位姿。一般地，$T_{de}$ 表示 $\{e\}$ 相对于 $\{d\}$ 的位姿；反向描述满足

$$
\boxed{T_{de}=T_{ed}^{-1}}.
$$

参考系不必总是相对于 $\{s\}$ 表示。例如

$$
T_{bc}=T_{sb}^{-1}T_{sc}
$$

表示 $\{c\}$ 相对于 $\{b\}$ 的位姿。由图中数据可得

$$
T_{bc}=
\begin{bmatrix}
0&1&0&0\\
0&0&-1&-3\\
-1&0&0&-1\\
0&0&0&1
\end{bmatrix}.
$$


##### 用途二：改变坐标表示

对于三个参考系 $\{a\}$、$\{b\}$、$\{c\}$，齐次变换矩阵满足下标消去规则：

$$
\boxed{T_{ab}T_{bc}=T_{ac}}.
$$

两个相邻的 $b$ 消去，剩下 $a$ 和 $c$。这表示：先把 $\{c\}$ 中的表示转换到 $\{b\}$，再从 $\{b\}$ 转到 $\{a\}$，等价于直接从 $\{c\}$ 转到 $\{a\}$。

同一个点 $v$ 的坐标满足

$$
\boxed{v_a=T_{ab}v_b=R_{ab}v_b+p_{ab}}.
$$

这里物理点没有移动，改变的只是坐标表示。使用 Figure 3.14 时，由于 $\{a\}$ 与 $\{s\}$ 重合，$T_{ab}=T_{sb}$，因此

$$
v_a
=R_{sb}v_b+p_{sb}
=
\begin{bmatrix}1.5\\-2\\0\end{bmatrix}.
$$

要区分这两句话：$T_{ab}$ 本身可以表示 $\{b\}$ 相对于 $\{a\}$ 的位姿；当它作为算子作用于 $v_b$ 时，又把同一物理点的坐标从 $\{b\}$ 改写为 $\{a\}$。矩阵相同，语义取决于它在句子中的角色。


之前学的是：

$$[v]_a = R_{ab}[v]_b$$

这里默认的是一个**向量**，或者两个坐标系原点相同/平移对向量无影响。因为向量只关心方向和大小，不关心起点。

但现在如果是一个**点**  q，两个坐标系不仅朝向不同，而且原点也不在同一个地方，那就不能只乘 R 了。


##### 用途三：主动旋转并平移参考系

前一种用途只改变同一点的坐标表示；这一种用途讨论主动位移（active displacement）：参考系 $\{b\}$ 本身真的运动到新位姿。

![教材 Figure 3.15：固定参考系左乘与物体参考系右乘](../../attachments/ch03/fig-3-15-fixed-vs-body-transform.png)

*来源：教材 Figure 3.15，书页 94／本地 PDF 第 113 页；局部截取。*

先把纯旋转与纯平移写成齐次变换：

$$
\operatorname{Rot}(\hat\omega,\theta)
=
\begin{bmatrix}
R&0\\
0&1
\end{bmatrix},
\qquad
\operatorname{Trans}(p)
=
\begin{bmatrix}
I&p\\
0&1
\end{bmatrix},
$$
第一个是旋转，它的p为0，代表参考系原点位置没有改变

第二个是平移操作

其中 $R=e^{[\hat\omega]\theta}$。令 $T=(R,p)$ 表示同一组旋转和平移参数。


###### 相对于固定参考系运动：左乘

如果 $\hat\omega$ 和 $p$ 都用固定参考系 $\{s\}$ 表示，则先绕固定轴 $\hat\omega_s$ 旋转，再沿 $p_s$ 平移：

$$
\boxed{
T_{sb'}=TT_{sb}
=\operatorname{Trans}(p)\operatorname{Rot}(\hat\omega,\theta)T_{sb}
=
\begin{bmatrix}
RR_{sb}&Rp_{sb}+p\\
0&1
\end{bmatrix}}
\tag{3.66}
$$

$T_{sb}$ 就是：

$$\boxed{\text{物体坐标系 }\{b\}\text{ 相对于空间坐标系 }\{s\}\text{ 的齐次变换矩阵}}$$

也就是它同时描述了：

$$\boxed{\text{朝向 }R_{sb}+\text{位置 }p_{sb}}$$


$$T_{sb} = \begin{bmatrix} R_{sb}&p_{sb}\\ 0&1 \end{bmatrix}$$

其中：
$$R_{sb}=\text{当前朝向}$$$$p_{sb}=\text{当前 }b\text{ 原点在 }s\text{ 中的位置}$$


$$T_{sb'}=T\,T_{sb}$$

其中
$$T= \begin{bmatrix} R&p\\ 0&1 \end{bmatrix}$$

这里的 \(R,p\) 都是按照**空间坐标系**定义的。

相乘得到：

$$\boxed{ T_{sb'} = \begin{bmatrix} RR_{sb}&Rp_{sb}+p\\ 0&1 \end{bmatrix} }$$

矩阵作用从右向左读：$\operatorname{Rot}$ 先作用，$\operatorname{Trans}$ 后作用。由于旋转轴穿过 $\{s\}$ 的原点，若 $\{b\}$ 的原点不与它重合，旋转本身也会使 $\{b\}$ 的原点移动。

它是绕空间固定轴运动

原来物体原点的位置是： $p_{sb}$

现在你是在**世界坐标系中**做一个旋转 R。

所以这个原点本身也会被绕空间原点旋转：$p_{sb}\rightarrow Rp_{sb}$

然后再加一个空间坐标中的平移：+p

于是： $\boxed{p_{sb}'=Rp_{sb}+p}$



###### 相对于物体参考系运动：右乘

旋转平移运动用物体自己的坐标系 {b} 表示

如果 $\hat\omega$ 和 $p$ 都用物体参考系 $\{b\}$ 表示，则先沿物体坐标中的 $p_b$ 平移，再绕对应的新物体轴旋转：

$$
\boxed{
T_{sb''}=T_{sb}T
=T_{sb}\operatorname{Trans}(p)\operatorname{Rot}(\hat\omega,\theta)
=
\begin{bmatrix}
R_{sb}R&R_{sb}p+p_{sb}\\
0&1
\end{bmatrix}}
\tag{3.67}
$$


这里的 p 是：

> **用物体自己的 b 坐标系表示的平移。**

所以首先必须转换，用s坐标系去表示这个平移

$p_s=R_{sb}p_b$

然后才加到当前空间位置：$p_{\text{new}} = p_{sb}+R_{sb}p$


这里最右侧的 $\operatorname{Rot}$ 在代数上先作用于被变换的坐标，但从参考系运动的几何构造来看，教材描述为先把参考系沿自身方向平移，再绕平移后参考系的轴旋转；后一步绕自身原点旋转，不再改变该原点的位置。关键不是死背文字顺序，而是检查平移块：

$$
\text{左乘： }Rp_{sb}+p,
\qquad
\text{右乘： }R_{sb}p+p_{sb}.
$$

左乘中的 $p$ 已在空间坐标中，直接加到最终位置；右乘中的 $p$ 在物体坐标中，必须先由 $R_{sb}$ 转到空间坐标。


###### 总结


$$\boxed{\text{左乘：新增运动用空间坐标系 } \{s\}\text{ 表示}}$$
$$\boxed{\text{右乘：新增运动用物体坐标系 } \{b\}\text{ 表示}}$$

而最终得到的 $T_{sb'}$ 或 $T_{sb''}$

仍然都是：

$$\boxed{\text{物体新坐标系相对于空间坐标系 } \{s\}\text{ 的位姿}}$$

左乘和右乘的区别：

>	你这一步新增的旋转和平移，是用哪个坐标系来定义的。


###### Figure 3.15 的数值验证

图中

$$
\hat\omega=(0,0,1),
\qquad \theta=90^\circ,
\qquad p=(0,2,0)^T,
$$

所以

$$
T=
\begin{bmatrix}
0&-1&0&0\\
1&0&0&2\\
0&0&1&0\\
0&0&0&1
\end{bmatrix},
\qquad
T_{sb}=
\begin{bmatrix}
0&0&1&0\\
0&-1&0&-2\\
1&0&0&0\\
0&0&0&1
\end{bmatrix}.
$$

按固定参考系解释并左乘：

$$
T_{sb'}=TT_{sb}
=
\begin{bmatrix}
0&1&0&2\\
0&0&1&2\\
1&0&0&0\\
0&0&0&1
\end{bmatrix}.
$$

按物体参考系解释并右乘：

$$
T_{sb''}=T_{sb}T
=
\begin{bmatrix}
0&0&1&0\\
-1&0&0&-4\\
0&-1&0&0\\
0&0&0&1
\end{bmatrix}.
$$

同一个 $T$ 放在 $T_{sb}$ 左边或右边，会把 $\hat\omega,p$ 分别解释为空间参考系量或物体参考系量，因此得到不同的新位姿。这不是矩阵乘法的偶然差异，而是“这些运动参数用哪个参考系表达”的差异。

##### Example 3.19：移动平台、机械臂、相机与物体的参考系链

![教材 Figure 3.16：移动平台、末端、相机、物体与固定参考系](../../attachments/ch03/fig-3-16-reference-frame-assignment.png)

*来源：教材 Figure 3.16，书页 95／本地 PDF 第 114 页；局部截取。*

Figure 3.16 中：

- $\{a\}$：房间中的固定参考系；
- $\{b\}$：轮式移动平台参考系；
- $\{c\}$：机械臂末端执行器参考系；
- $\{d\}$：天花板相机参考系；
- $\{e\}$：待抓取物体参考系。

相机测得 $T_{db}$ 与 $T_{de}$，机械臂关节角给出 $T_{bc}$，相机相对于房间的标定 $T_{ad}$ 已知。目标是求物体相对于机械手的位姿 $T_{ce}$。

教材给定

$$
T_{db}=
\begin{bmatrix}
0&0&-1&250\\
0&-1&0&-150\\
-1&0&0&200\\
0&0&0&1
\end{bmatrix},
\qquad
T_{de}=
\begin{bmatrix}
0&0&-1&300\\
0&-1&0&100\\
-1&0&0&120\\
0&0&0&1
\end{bmatrix},
$$

$$
T_{ad}=
\begin{bmatrix}
0&0&-1&400\\
0&-1&0&50\\
-1&0&0&300\\
0&0&0&1
\end{bmatrix},
$$

$$
T_{bc}=
\begin{bmatrix}
0&-1/\sqrt2&-1/\sqrt2&30\\
0&1/\sqrt2&-1/\sqrt2&-40\\
1&0&0&25\\
0&0&0&1
\end{bmatrix}.
$$

不要一开始就代数字。先画出两条都从目标参考系通向固定参考系 $\{a\}$ 的链：

$$
T_{ac}=T_{ad}T_{db}T_{bc},
\qquad
T_{ae}=T_{ad}T_{de}.
$$

因为

$$
T_{ac}T_{ce}=T_{ae},
$$

所以

$$
\boxed{
T_{ce}=T_{ac}^{-1}T_{ae}
=(T_{ad}T_{db}T_{bc})^{-1}T_{ad}T_{de}}
$$

下标检查非常直接：

$$
\underbrace{T_{ac}}_{c\to a}
\underbrace{T_{ce}}_{e\to c}
=
\underbrace{T_{ae}}_{e\to a}.
$$

代入数据，教材先得到

$$
T_{ad}T_{de}=
\begin{bmatrix}
1&0&0&280\\
0&1&0&-50\\
0&0&1&0\\
0&0&0&1
\end{bmatrix},
$$

$$
T_{ad}T_{db}T_{bc}=
\begin{bmatrix}
0&-1/\sqrt2&-1/\sqrt2&230\\
0&1/\sqrt2&-1/\sqrt2&160\\
1&0&0&75\\
0&0&0&1
\end{bmatrix},
$$

以及

$$
(T_{ad}T_{db}T_{bc})^{-1}=
\begin{bmatrix}
0&0&1&-75\\
-1/\sqrt2&1/\sqrt2&0&70/\sqrt2\\
-1/\sqrt2&-1/\sqrt2&0&390/\sqrt2\\
0&0&0&1
\end{bmatrix}.
$$

最终

$$
\boxed{
T_{ce}=
\begin{bmatrix}
0&0&1&-75\\
-1/\sqrt2&1/\sqrt2&0&-260/\sqrt2\\
-1/\sqrt2&-1/\sqrt2&0&130/\sqrt2\\
0&0&0&1
\end{bmatrix}}
$$

它同时给出物体 $\{e\}$ 相对于机械手 $\{c\}$ 的朝向和位置，正是规划抓取动作需要的相对位姿。

还可以从符号上进一步化简：

$$
T_{ce}
=T_{bc}^{-1}T_{db}^{-1}T_{ad}^{-1}T_{ad}T_{de}
=\boxed{T_{bc}^{-1}T_{db}^{-1}T_{de}}.
$$

这是由下标消去得到的等价式，说明求 $\{e\}$ 相对于 $\{c\}$ 的位姿时，共同的固定参考系 $\{a\}$ 最终会抵消。$T_{ad}$ 仍有助于构造两条空间位姿链，但相对位姿本身不依赖于选择哪个共同固定参考系。

### 3.3.2 旋量（Twists）

旋量（twist）把刚体的角速度和线速度合并为一个六维速度

因为三维刚体的瞬时运动，分解开来就是怎么转 和 怎么移

而线速度，不能在刚体上随便挑一点求它的线速度，不同点线速度不同。

==这也就是 space twist 和 body twist 的区别来源==


设运动参考系 $\{b\}$ 相对于固定参考系 $\{s\}$ 的位姿为

$$
T_{sb}(t)=T(t)=
\begin{bmatrix}
R(t)&p(t)\\
0&1
\end{bmatrix}.
\tag{3.68}
$$

其中 $p(t)$ 是 $\{b\}$ 原点在 $\{s\}$ 中的位置，因此

$$
\dot T=
\begin{bmatrix}
\dot R&\dot p\\
0&0
\end{bmatrix},
\qquad
T^{-1}=
\begin{bmatrix}
R^T&-R^Tp\\
0&1
\end{bmatrix}.
$$

$\dot R$ : 表示朝向变化率；

$\dot p$ : 原本p是物体坐标系原点的位置，这里对时间求导，表示物体原点的线速度，**但目前是用空间坐标系 s 表示的**。


前面学过：
$$[\omega_b]=R^T\dot R$$
或者：

$$[\omega_s]=\dot R R^T $$

需要把 $\dot R$ 和当前姿态  R 结合，才能提取角速度。

同样地，如果我们想让**角速度和线速度都用物体坐标系 b 表示**，就需要做一次坐标转换

于是考虑：

$$\boxed{T^{-1}\dot T}$$


#### 物体旋量：$T^{-1}\dot T$

把 $\dot T$ 左乘 $T^{-1}$：

$$
\begin{aligned}
T^{-1}\dot T
&=
\begin{bmatrix}
R^T&-R^Tp\\
0&1
\end{bmatrix}
\begin{bmatrix}
\dot R&\dot p\\
0&0
\end{bmatrix}\\
&=
\begin{bmatrix}
R^T\dot R&R^T\dot p\\
0&0
\end{bmatrix}
=
\begin{bmatrix}
[\omega_b]&v_b\\
0&0
\end{bmatrix}.
\end{aligned}
\tag{3.69}
$$

这里

$$
[\omega_b]=R^T\dot R,
\qquad
\boxed{v_b=R^T\dot p}.
$$

$\dot p$ 是 $\{b\}$ 原点的线速度，用空间参考系 $\{s\}$ 表示；乘以 $R^T$ 后得到同一个线速度在物体参考系 $\{b\}$ 中的坐标 $v_b$。


$[v]_s=R_{sb}[v]_b$  意思是：

> 同一个几何向量，从 \(b\) 坐标表示转换成 \(s\) 坐标表示。

所以反过来： $[v]_b=R_{sb}^{-1}[v]_s$

而旋转矩阵有： $R^{-1}=R^T$

因此： $\boxed{ [v]_b=R_{sb}^T[v]_s }$


物体旋量（body twist）定义为

$$
\boxed{
V_b=
\begin{bmatrix}
\omega_b\\
v_b
\end{bmatrix}
\in\mathbb R^6.}
\tag{3.70}
$$

它的矩阵表示为

$$
\boxed{
[V_b]=
\begin{bmatrix}
[\omega_b]&v_b\\
0&0
\end{bmatrix}
=T^{-1}\dot T
\in\mathfrak{se}(3).}
\tag{3.71}
$$

$\mathfrak{se}(3)$ 是所有这种 $4\times4$ 矩阵的集合，也是李群 $SE(3)$ 对应的李代数（Lie algebra）。与 $T\in SE(3)$ 的最后一行 $[0\;0\;0\;1]$ 不同，速度矩阵 $[V]\in\mathfrak{se}(3)$ 的最后一行全为零。

在机器人里，你可以先把它理解成：

$$\boxed{\mathfrak{se}(3)=\text{“三维刚体瞬时运动”对应的 }4\times4\text{ 矩阵集合}}$$

它不是位姿本身，而是位姿的**瞬时速度形式**。

一个典型的 $\mathfrak{se}(3)$ 元素写成：

$$\boxed{ [V] = \begin{bmatrix} [\omega] & v\\ 0 & 0 \end{bmatrix} }$$

其中：

$$[\omega] = \begin{bmatrix} 0&-\omega_3&\omega_2\\ \omega_3&0&-\omega_1\\ -\omega_2&\omega_1&0 \end{bmatrix}$$

是 $3\times3$ 反对称矩阵，

而：

$$v= \begin{bmatrix} v_x\\ v_y\\ v_z \end{bmatrix}$$

是线速度部分。

原本的六维向量很适合“存参数”，但**不方便直接和齐次变换矩阵 T 做矩阵运算**


#### 空间旋量：$\dot T T^{-1}$

换成把 $\dot T$ 右乘 $T^{-1}$：

$$
\begin{aligned}
\dot T T^{-1}
&=
\begin{bmatrix}
\dot R&\dot p\\
0&0
\end{bmatrix}
\begin{bmatrix}
R^T&-R^Tp\\
0&1
\end{bmatrix}\\
&=
\begin{bmatrix}
\dot R R^T&\dot p-\dot R R^Tp\\
0&0
\end{bmatrix}
=
\begin{bmatrix}
[\omega_s]&v_s\\
0&0
\end{bmatrix}.
\end{aligned}
\tag{3.72}
$$

因此

$$
[\omega_s]=\dot R R^T,
\qquad
\boxed{v_s=\dot p-[\omega_s]p
=\dot p-\omega_s\times p
=\dot p+\omega_s\times(-p).}
\tag{3.73}
$$

![教材 Figure 3.17：空间旋量线速度分量的物理含义](../../attachments/ch03/fig-3-17-spatial-twist-interpretation.png)

*来源：教材 Figure 3.17，书页 98／本地 PDF 第 117 页；局部截取。*

这里必须区分，在空间旋量中：

$$
\boxed{v_s\neq\dot p\quad\text{（一般情况下）}.}
$$

$\dot p$ 是物体参考系原点的实际线速度；$v_s$ 则是假想刚体无限延伸时，刚体上当前恰好位于空间原点的那个点的瞬时速度，并用 $\{s\}$ 表示。

**空间旋量（spatial twist）里的线速度部分 $v_s$，并不是物体坐标系原点的实际线速度 $\dot p$**


$\boxed{\dot p=\text{物体原点 }O_b\text{ 的实际线速度}}$

$v_s$ 是**速度场在空间原点处的值**


这和前面学过的刚体速度公式： $\boxed{ \dot q=v+\omega\times q }$

是完全同一个东西。

对于 spatial twist： $\boxed{ \dot q=v_s+\omega_s\times q }$

其中 q 是刚体上任意一点在空间坐标系中的位置。

---
空间旋量 $V_s$：

其中 $v_s$ 的几何意义是：

$$\boxed{\text{刚体速度场在空间原点 }O_s\text{ 处的线速度}}$$

任意点 q 的速度：

$$\boxed{ \dot q=v_s+\omega_s\times q }$$

特别地令：$q=0$

得到： $\dot q=v_s$

因此 $v_s$ 就是“空间原点处”的速度项。



物体旋量 $V_b$:

它们全部用物体坐标系 {b\} 表示。

这时选择的参考点非常自然：**物体坐标系自己的原点 $O_b$**。

$$\dot q_b=v_b+\omega_b\times q_b$$

令 $q_b=0$，自然得到： $\dot q_b=v_b$

也就是说 $v_b$ 对应自己的原点。

因此：

$$\boxed{ v_b=R^T\dot p }$$

> $v_b$ 就是物体原点 $O_b$ 的实际线速度，只不过用物体系 b 表示。


一个是用空间系描述瞬时运动，一个是用自己的物体坐标系描述瞬时运动

---



Figure 3.17 中，从物体原点指向空间原点的向量是 $-p$，所以该点的速度为

$$
v_s=\dot p+\omega_s\times(-p).
$$


空间旋量（spatial twist）及其矩阵表示为

$$
\boxed{
V_s=
\begin{bmatrix}
\omega_s\\
v_s
\end{bmatrix},
\qquad
[V_s]=
\begin{bmatrix}
[\omega_s]&v_s\\
0&0
\end{bmatrix}
=\dot T T^{-1}.}
\tag{3.74}
$$


#### $V_b$ 与 $V_s$ 是同一运动的两种表示

由 $[V_s]=\dot T T^{-1}$ 可得 $\dot T=[V_s]T$；代入 $[V_b]=T^{-1}\dot T$：

$$
\boxed{[V_b]=T^{-1}[V_s]T.}
\tag{3.75}
$$

反过来，

$$
\boxed{[V_s]=T[V_b]T^{-1}.}
\tag{3.76}
$$

这与角速度中的

$$
[\omega_b]=R^T[\omega_s]R,
\qquad
[\omega_s]=R[\omega_b]R^T
$$

完全平行，只是现在旋转和平移速度被一起放进 $\mathfrak{se}(3)$：

$$
\boxed{
[V_b]=T^{-1}\dot T\ \text{在右侧生成运动},
\qquad
[V_s]=\dot T T^{-1}\ \text{在左侧生成运动}.}
$$


它的意思是：这个瞬时运动增量，是乘在当前位姿的右边还是左边

比如物体旋量：

$$[V_b]=T^{-1}\dot T$$

可以整理成：

$$\boxed{\dot T=T[V_b]}$$

所以说它“在右侧生成运动”，因为：

$$T \underbrace{[V_b]}_{\text{右边}}$$

这表示当前位姿 T 的变化，是通过**右乘一个物体系中的瞬时运动**产生的。


#### 伴随表示（Adjoint Representation）


这一部分是在解决一个很实际的问题：

同一个 twist，怎么从物体系坐标 $V_b$ 转成空间系坐标 $V_s$

之前普通向量做坐标变换只需要旋转矩阵：

$[v]_s=R[v]_b$

但 twist 不是普通 3 维向量，它是 6 维的：

$$V= \begin{bmatrix} \omega\\ v \end{bmatrix}$$

里面既有角速度 ，又有线速度 v。而线速度部分还会受到“坐标系原点位置不同”的影响，所以不能只乘一个 R 。 所以引入了伴随矩阵

使得：

$$\boxed{ V_s=\operatorname{Ad}_T V_b }$$

这就是它最大的用途。

下面就是求这个伴随：

式 (3.76) 给出 $[V_s]=T[V_b]T^{-1}$。令 $T=(R,p)$，将右侧完整展开：

$$
\begin{aligned}
T[V_b]T^{-1}
&=
\begin{bmatrix}R&p\\0&1\end{bmatrix}
\begin{bmatrix}[\omega_b]&v_b\\0&0\end{bmatrix}
\begin{bmatrix}R^T&-R^Tp\\0&1\end{bmatrix}\\
&=
\begin{bmatrix}
R[\omega_b]R^T&-R[\omega_b]R^Tp+Rv_b\\
0&0
\end{bmatrix}.
\end{aligned}
$$

利用 Proposition 3.8 和叉乘恒等式

$$
R[\omega_b]R^T=[R\omega_b]=[\omega_s],
\qquad
[\omega]p=-[p]\omega,
$$

得到

$$
-R[\omega_b]R^Tp
=-[\omega_s]p
=[p]\omega_s
=[p]R\omega_b.
$$

因此

$$
\begin{bmatrix}
\omega_s\\
v_s
\end{bmatrix}
=
\begin{bmatrix}
R&0\\
[p]R&R
\end{bmatrix}
\begin{bmatrix}
\omega_b\\
v_b
\end{bmatrix}.
$$

**Definition 3.20**：给定 $T=(R,p)\in SE(3)$，其伴随表示（adjoint representation）定义为

$$
\boxed{
[\operatorname{Ad}_T]
=
\begin{bmatrix}
R&0\\
[p]R&R
\end{bmatrix}
\in\mathbb R^{6\times6}.}
$$

对任意 $V\in\mathbb R^6$，伴随映射为

$$
V'=[\operatorname{Ad}_T]V,
$$

在 $\mathfrak{se}(3)$ 的矩阵表示下等价于

$$
\boxed{[V']=T[V]T^{-1}.}
$$

分块展开就是

$$
\boxed{
\omega'=R\omega,
\qquad
v'=[p]R\omega+Rv
=p\times(R\omega)+Rv.}
$$

角速度只需旋转坐标；线速度除了旋转坐标，还必须补上参考系原点偏移 $p$ 产生的 $p\times\omega'$。因此不能简单地对 $\omega$ 和 $v$ 各乘一次 $R$。


**Proposition 3.21** 给出两个基本性质。复合变换对应伴随矩阵的同序复合：

$$
\boxed{
\operatorname{Ad}_{T_1}(\operatorname{Ad}_{T_2}(V))
=\operatorname{Ad}_{T_1T_2}(V),}
$$

即

$$
\boxed{
[\operatorname{Ad}_{T_1}][\operatorname{Ad}_{T_2}]
=[\operatorname{Ad}_{T_1T_2}].}
\tag{3.77}
$$

逆变换的伴随矩阵就是伴随矩阵的逆：

$$
\boxed{
[\operatorname{Ad}_T]^{-1}
=[\operatorname{Ad}_{T^{-1}}].}
\tag{3.78}
$$

因为

$$
\operatorname{Ad}_{T^{-1}}(\operatorname{Ad}_T(V))
=\operatorname{Ad}_{T^{-1}T}(V)
=\operatorname{Ad}_I(V)=V.
\tag{3.79}
$$

#### 3.3.2.1 旋量结果总结

**Proposition 3.22**：对

$$
T_{sb}(t)=
\begin{bmatrix}R(t)&p(t)\\0&1\end{bmatrix},
\tag{3.80}
$$

物体旋量和空间旋量分别为

$$
\boxed{
T_{sb}^{-1}\dot T_{sb}=[V_b]
=\begin{bmatrix}[\omega_b]&v_b\\0&0\end{bmatrix}
\in\mathfrak{se}(3),}
\tag{3.81}
$$

$$
\boxed{
\dot T_{sb}T_{sb}^{-1}=[V_s]
=\begin{bmatrix}[\omega_s]&v_s\\0&0\end{bmatrix}
\in\mathfrak{se}(3).}
\tag{3.82}
$$

两者满足

$$
\boxed{
V_s=
\begin{bmatrix}
R&0\\
[p]R&R
\end{bmatrix}V_b
=[\operatorname{Ad}_{T_{sb}}]V_b,}
\tag{3.83}
$$

以及

$$
\boxed{
V_b=
\begin{bmatrix}R^T&0\\-R^T[p]&R^T\end{bmatrix}V_s
=[\operatorname{Ad}_{T_{bs}}]V_s.}
\tag{3.84}
$$

一般地，同一旋量在任意两个参考系 $\{c\}$、$\{d\}$ 中的表示满足

$$
\boxed{
V_c=[\operatorname{Ad}_{T_{cd}}]V_d,
\qquad
V_d=[\operatorname{Ad}_{T_{dc}}]V_c.}
$$

下标规则仍然有效：$T_{cd}$ 把在 $\{d\}$ 中表示的旋量转换成在 $\{c\}$ 中的表示。

教材还强调两个不变性：对于同一个物理旋量，空间表示 $V_s$ 不依赖于选取哪个物体参考系 $\{b\}$；物体表示 $V_b$ 也不依赖于选取哪个固定参考系 $\{s\}$。改变相应参考系的选择会同时改变 $T$ 和速度数据，但按定义组合后的 $V_s$ 或 $V_b$ 保持不变。

##### Example 3.23：三轮车辆的瞬时旋量

![教材 Figure 3.18：三轮车辆的瞬时旋转中心与旋量](../../attachments/ch03/fig-3-18-vehicle-twist.png)

*来源：教材 Figure 3.18，书页 102／本地 PDF 第 121 页；局部截取。*

车辆此刻绕平面内点 $r$、以 $w=2\ \mathrm{rad/s}$ 的角速度纯旋转。图中 $\hat z_s$ 指向纸外，而 $\hat z_b$ 指向纸内，所以同一物理角速度在两个参考系中的坐标为

$$
\omega_s=\begin{bmatrix}0\\0\\2\end{bmatrix},
\qquad
\omega_b=\begin{bmatrix}0\\0\\-2\end{bmatrix}.
$$

瞬时旋转中心在两个参考系中的位置为

$$
r_s=\begin{bmatrix}2\\-1\\0\end{bmatrix},
\qquad
r_b=\begin{bmatrix}2\\-1.4\\0\end{bmatrix}.
$$

对绕点 $r$ 的纯旋转，参考系原点相对于旋转中心的位置是 $-r$，因此旋量的线速度分量满足

$$
v=\omega\times(-r)=r\times\omega.
$$

空间表示为

$$
v_s=r_s\times\omega_s
=
\begin{bmatrix}2\\-1\\0\end{bmatrix}
\times
\begin{bmatrix}0\\0\\2\end{bmatrix}
=
\begin{bmatrix}-2\\-4\\0\end{bmatrix},
$$

物体表示为

$$
v_b=r_b\times\omega_b
=
\begin{bmatrix}2\\-1.4\\0\end{bmatrix}
\times
\begin{bmatrix}0\\0\\-2\end{bmatrix}
=
\begin{bmatrix}2.8\\4\\0\end{bmatrix}.
$$

因此

$$
\boxed{
V_s=
\begin{bmatrix}
0\\0\\2\\-2\\-4\\0
\end{bmatrix},
\qquad
V_b=
\begin{bmatrix}
0\\0\\-2\\2.8\\4\\0
\end{bmatrix}.}
$$

教材给出的车辆位姿为

$$
T_{sb}=
\begin{bmatrix}
-1&0&0&4\\
0&1&0&0.4\\
0&0&-1&0\\
0&0&0&1
\end{bmatrix},
$$

即

$$
R_{sb}=\operatorname{diag}(-1,1,-1),
\qquad
p_{sb}=\begin{bmatrix}4\\0.4\\0\end{bmatrix}.
$$

现在直接验证伴随关系。角速度部分：

$$
R_{sb}\omega_b
=
\begin{bmatrix}0\\0\\2\end{bmatrix}
=\omega_s.
$$

线速度部分：

$$
p_{sb}\times(R_{sb}\omega_b)
=
\begin{bmatrix}4\\0.4\\0\end{bmatrix}
\times
\begin{bmatrix}0\\0\\2\end{bmatrix}
=
\begin{bmatrix}0.8\\-8\\0\end{bmatrix},
$$

$$
R_{sb}v_b
=
\begin{bmatrix}-2.8\\4\\0\end{bmatrix}.
$$

相加得到

$$
p_{sb}\times(R_{sb}\omega_b)+R_{sb}v_b
=
\begin{bmatrix}-2\\-4\\0\end{bmatrix}
=v_s.
$$

所以数值上确实满足

$$
\boxed{V_s=[\operatorname{Ad}_{T_{sb}}]V_b.}
$$

这个例子也说明：$v_s$ 与 $v_b$ 不是把同一个普通三维向量旋转一下那么简单。由于两个旋量的参考点分别是空间原点和物体原点，必须通过伴随表示中的平移耦合项修正。


#### 3.3.2.2 旋量的螺旋解释


一个六维旋量，在几何上代表什么运动

>刚体的任意瞬时运动，都可以看成“绕某一条轴旋转，同时沿这条轴平移”

刚体的一般瞬时运动可以看成绕一条轴旋转，同时沿这条轴平移，就像螺钉旋入材料。螺旋轴（screw axis）可以先用三元组

$$
\{q,\hat s,h\}
$$

描述，其中：

- $q\in\mathbb R^3$ 是螺旋轴上的任意一点；
- $\hat s\in\mathbb R^3$ 是沿轴方向的单位向量；
- $h$ 是节距（pitch），即沿轴线速度与绕轴角速度之比。

一个是角速度，它是旋转产生的，沿着旋转轴的
一个是沿轴线速度，它是螺丝本身沿自身轴向里推进的速度

![教材 Figure 3.19：由轴上一点、单位方向和节距描述螺旋轴](../../attachments/ch03/fig-3-19-screw-axis.png)

*来源：教材 Figure 3.19，书页 103／本地 PDF 第 122 页；局部截取。*

若绕螺旋轴的角速度为 $\dot\theta$，则

$$
\omega=\hat s\dot\theta.
$$

原点相对于轴上一点 $q$ 的位置是 $-q$，因此旋转产生的原点速度为
$$
\omega\times(-q)
=-\hat s\dot\theta\times q.
$$

沿轴平移产生的速度为

$$
h\hat s\dot\theta.
$$

将两部分相加：

$$
\boxed{
V=
\begin{bmatrix}\omega\\v\end{bmatrix}
=
\begin{bmatrix}
\hat s\dot\theta\\
-\hat s\dot\theta\times q+h\hat s\dot\theta
\end{bmatrix}.}
$$

提取 $\dot\theta$：

$$
\boxed{
V=S\dot\theta,
\qquad
S=
\begin{bmatrix}
\hat s\\
-\hat s\times q+h\hat s
\end{bmatrix}.}
$$

线速度部分包含两个互相正交的分量：

$$
v=\underbrace{-\hat s\times q\,\dot\theta}_{\text{绕轴旋转引起}}
+
\underbrace{h\hat s\dot\theta}_{\text{沿轴平移引起}}.
$$

第一项垂直于 $\hat s$，第二项平行于 $\hat s$。因此节距可以从平行分量中恢复：

$$
\boxed{
h=\frac{\hat s^Tv}{\dot\theta}.}
$$



##### 从一般旋量恢复有限节距螺旋

上面是：

> 已知螺旋轴的几何参数 $q,\hat s,h$ 和运动速率 $\dot\theta$，怎么得到旋量 $V=(\omega,v)$？

现在是：

如果我已经拿到一个一般旋量

$$V= \begin{bmatrix} \omega\\ v \end{bmatrix}$$

怎么从这 6 个数恢复出：螺旋轴朝哪里、在哪里、节距是多少？


若 $\omega\ne0$，则

$$
\boxed{
\dot\theta=\|\omega\|,
\qquad
\hat s=\frac{\omega}{\|\omega\|},
\qquad
h=\frac{\hat s^Tv}{\dot\theta}.}
$$

轴上一点 $q$ 选择为使 $-\hat s\dot\theta\times q$ 等于 $v$ 中垂直于轴的部分。$q$ 不唯一：沿轴方向给它加上任意 $\lambda\hat s$，描述的仍是同一条轴。若选择离原点最近、满足 $q\perp\hat s$ 的点，则可写成

$$
q=\frac{\omega\times v}{\|\omega\|^2}.
$$

这是由教材关系直接推出的方便公式。

##### 纯平移：无穷节距

若 $\omega=0$，运动没有任何旋转。此时把节距理解为

$$
h=\infty,
$$

并定义

$$
\boxed{
\hat s=\frac{v}{\|v\|},
\qquad
\dot\theta=\|v\|.}
$$

这里 $\dot\theta$ 不再表示角速度，而解释为沿 $\hat s$ 的线速度。


##### Definition 3.24：归一化螺旋轴（screw axis）

为了避免三元组 $\{q,\hat s,h\}$ 中 $q$ 不唯一以及 $h$ 可能无穷的问题，教材把螺旋轴直接定义为归一化六维向量

$$
\boxed{
S=
\begin{bmatrix}\omega\\v\end{bmatrix}
\in\mathbb R^6.}
$$

它必须满足以下二者之一：

1. $\|\omega\|=1$，此时

   $$
   \boxed{v=-\omega\times q+h\omega,}
   $$

   其中 $q$ 在螺旋轴上；$h=0$ 表示绕该轴纯旋转。

   这个式子是在说：

- $\omega$：螺旋轴方向
- q：轴上的一点
- $-\omega\times q$：因为轴不经过原点而产生的线速度部分
- $h\omega$：沿轴方向的平移部分

所以：

$$\boxed{ S= \begin{bmatrix} \omega\\ -\omega\times q+h\omega \end{bmatrix} }$$

就是一个标准化的 screw axis。



2. $\omega=0$ 且 $\|v\|=1$。此时表示完全不转，只平移，它是沿 $v$ 方向的纯平移，节距为无穷。



给定一般旋量 $V=(\omega,v)$，归一化规则是

$$
\boxed{
\omega\ne0:\quad
S=\frac{V}{\|\omega\|},
\qquad
\dot\theta=\|\omega\|,}
$$

$$
\boxed{
\omega=0:\quad
S=\frac{V}{\|v\|},
\qquad
\dot\theta=\|v\|.}
$$

两种情况下都有

$$
\boxed{V=S\dot\theta.}
$$
因为一般旋量：

$$V= \begin{bmatrix} \omega\\ v \end{bmatrix}$$

里面的 $\omega$ 可能长度不是 1。

比如： $\|\omega\|=3$

说明当前角速度大小是 3。

为了只留下“运动方向”，把整个 6 维旋量一起除以 3：

$S= \frac{V}{\|\omega\|}$

于是新的角速度部分：$\frac{\omega}{\|\omega\|}$

就变成单位向量。

同时定义：

$$\boxed{ \dot\theta=\|\omega\| }$$

这样：

$$V=S\dot\theta$$

仍然成立。

---


教材同时提醒：$S=(\omega,v)$ 和一般旋量 $V=(\omega,v)$ 使用相同的符号结构，但 $S$ 是归一化的，而 $V$ 没有单位长度约束，必须根据上下文区分。

螺旋轴的矩阵表示为

$$
\boxed{
[S]=
\begin{bmatrix}
[\omega]&v\\
0&0
\end{bmatrix}
\in\mathfrak{se}(3),}
\tag{3.85}
$$

其中

$$
[\omega]=
\begin{bmatrix}
0&-\omega_3&\omega_2\\
\omega_3&0&-\omega_1\\
-\omega_2&\omega_1&0
\end{bmatrix}.
$$

螺旋轴换参考系时仍使用伴随表示：

$$
\boxed{
S_a=[\operatorname{Ad}_{T_{ab}}]S_b,
\qquad
S_b=[\operatorname{Ad}_{T_{ba}}]S_a.}
$$



### 3.3.3 刚体运动的指数坐标表示

#### 3.3.3.1 刚体运动的指数坐标

把之前学的“旋转的指数坐标”推广到了 “刚体位姿的指数坐标”

上一节 SO(3) 里，你有：

$$\hat\omega\theta \in \mathbb R^3$$

它表示：

- 轴方向：$\hat\omega$
- 转角：$\theta$

然后通过：

$$R=e^{[\hat\omega]\theta}$$

把指数坐标变成旋转矩阵。


Chasles–Mozzi 定理指出：任意空间刚体位移都可以表示为沿空间中某条固定螺旋轴 $S$ 的一次位移。纯平移也包含在内，只需把旋转轴理解为位于无穷远、节距为无穷。

这使 $SO(3)$ 中的轴角表示自然推广到 $SE(3)$：

$$
\hat\omega\theta\in\mathbb R^3
\quad\longrightarrow\quad
S\theta\in\mathbb R^6.
$$

$S\theta$ 称为齐次变换（homogeneous transformation）的指数坐标（exponential coordinates）。其中：

- 若 $S=(\omega,v)$ 的节距有限，则 $\|\omega\|=1$，$\theta$ 是绕螺旋轴旋转的角度；
- 若螺旋轴表示纯平移，则 $\omega=0$、$\|v\|=1$，$\theta$ 是沿轴移动的线距离。


矩阵指数和矩阵对数建立

$$
\exp:\ [S]\theta\in\mathfrak{se}(3)\longrightarrow T\in SE(3),
$$

$$
\log:\ T\in SE(3)\longrightarrow [S]\theta\in\mathfrak{se}(3)
$$

之间的对应。



 $$\boxed{\text{已知一个固定螺旋轴 }S\text{，沿它运动了 }\theta\text{，最终位姿 }T\text{ 是多少？}}$$

答案就是：

$$\boxed{T(\theta)=e^{[S]\theta}}$$

而下面的所有 $G(\theta)$、矩阵级数、Rodrigues 公式，都是在把这个 $e^{[S]\theta}$ **真正算出来**。

前面纯旋转时你已经见过同样的东西

纯旋转：$\dot R=[\omega]R$

如果旋转轴固定，解是：

$R(\theta)=e^{[\omega]\theta}$

然后矩阵指数经过化简，得到 Rodrigues：

$$R = I+\sin\theta[\omega] +(1-\cos\theta)[\omega]^2$$
这一节只是问：

> 如果不仅旋转，还同时平移呢？

于是把：$[\omega]$

升级成完整旋量矩阵：

$$[S] = \begin{bmatrix} [\omega]&v\\ 0&0 \end{bmatrix}$$

于是自然有：

$$\boxed{ T=e^{[S]\theta} }$$
它怎么得到的：

先从瞬时运动开始。对于固定螺旋轴 S，旋量速度写成：

$$V=S\dot\theta$$

$$\boxed{\text{实际旋量速度}=\text{归一化螺旋轴}\times\text{运动速率}}$$

把它写成矩阵形式：

$$[V]=[S]\dot\theta$$



之前是：

$$\omega=\hat\omega\dot\theta$$

把 $\omega$ 和 $\hat\omega$ 都写成叉积矩阵以后：

$$[\omega]=[\hat\omega]\dot\theta$$

在这里，这种改写成矩阵形式：
**不是“把一个向量等式随便改写成矩阵等式”**，而是我们定义了一个固定的映射，把 6 维向量 V 编码成一个 4 * 4  矩阵 V。

这里定义：
$$[V] = \begin{bmatrix} [\omega] & v\\ 0&0 \end{bmatrix}$$

如果 S 是空间螺旋轴，你前面学过：

$$[V_s]=\dot T T^{-1}$$

所以：

$$\dot T T^{-1}=[S]\dot\theta$$

右乘 T：

$$\boxed{ \dot T=[S]\dot\theta\,T }$$

这就是关于 T(t) 的微分方程。

如果改用 $\theta$ 而不是时间 t 做自变量，因为：

$\dot T=\frac{dT}{d\theta}\dot\theta$

代进去：

$\frac{dT}{d\theta}\dot\theta = [S]\dot\theta T$

假设 $\dot\theta\neq0$，约掉：

$\boxed{ \frac{dT}{d\theta}=[S]T }$

它和普通的：$\frac{dx}{dt}=Ax$ 完全同型。


于是：

$$\boxed{ T(\theta)=e^{[S]\theta}T(0) }$$

如果一开始从单位位姿出发：

$T(0)=I$

那么：

$$\boxed{ T(\theta)=e^{[S]\theta} }$$

---

##### 从矩阵指数级数得到平移块

对

$$
[S]=
\begin{bmatrix}
[\omega]&v\\
0&0
\end{bmatrix},
$$

矩阵指数定义为

$$
e^{[S]\theta}
=I+[S]\theta+[S]^2\frac{\theta^2}{2!}
+[S]^3\frac{\theta^3}{3!}+\cdots.
$$
它的本质就是：

$$\boxed{\text{把一个恒定的“瞬时螺旋速度方向” }S\text{ 累积成有限刚体位姿变化}}$$


因为

$$
[S]^2=
\begin{bmatrix}
[\omega]^2&[\omega]v\\
0&0
\end{bmatrix},
\qquad
[S]^3=
\begin{bmatrix}
[\omega]^3&[\omega]^2v\\
0&0
\end{bmatrix},
$$

所以旋转块和位移块分别聚合为

$$
\boxed{
e^{[S]\theta}
=
\begin{bmatrix}
e^{[\omega]\theta}&G(\theta)v\\
0&1
\end{bmatrix},}
$$
左上角是旋转块，右上角是位移块

左上角是：

$$I+[\omega]\theta+ \frac{[\omega]^2\theta^2}{2!}+\cdots = e^{[\omega]\theta}$$

这就是旋转矩阵。

右上角则是：

$$v\theta + [\omega]v\frac{\theta^2}{2!} + [\omega]^2v\frac{\theta^3}{3!} +\cdots$$

把共同的 v 放到右边：

$$= \left( I\theta+ [\omega]\frac{\theta^2}{2!} + [\omega]^2\frac{\theta^3}{3!} +\cdots \right)v$$

其中

$$
\boxed{
G(\theta)
=I\theta
+[\omega]\frac{\theta^2}{2!}
+[\omega]^2\frac{\theta^3}{3!}
+\cdots.}
\tag{3.86}
$$


这说明：只要运动中含有旋转，最终平移通常不是简单的 $v\theta$。瞬时线速度方向会随螺旋运动累积，所有高阶项共同形成 $G(\theta)v$。

G就是把瞬时线速度分量 v，在整个旋转过程中正确累计成最终位移 p 的矩阵。


##### $G(\theta)$ 的闭式

当 $\|\omega\|=1$ 时，

$$
[\omega]^3=-[\omega].
$$

把 $G(\theta)$ 中奇偶次幂分别归并：

$$
\begin{aligned}
G(\theta)
&=I\theta
+\left(\frac{\theta^2}{2!}-\frac{\theta^4}{4!}
+\frac{\theta^6}{6!}-\cdots\right)[\omega]\\
&\quad+
\left(\frac{\theta^3}{3!}-\frac{\theta^5}{5!}
+\frac{\theta^7}{7!}-\cdots\right)[\omega]^2.
\end{aligned}
$$

识别出余弦和正弦级数：

$$
\boxed{
G(\theta)
=I\theta
+(1-\cos\theta)[\omega]
+(\theta-\sin\theta)[\omega]^2.}
\tag{3.87}
$$

注意 $G(\theta)$ 不是旋转矩阵。它负责把螺旋轴的线速度分量 $v$ 积分成有限位移 $p$：

$$
\boxed{p=G(\theta)v.}
$$


##### Proposition 3.25：$SE(3)$ 矩阵指数闭式

若 $S=(\omega,v)$ 且 $\|\omega\|=1$，则

$$
\boxed{
e^{[S]\theta}
=
\begin{bmatrix}
e^{[\omega]\theta}
&
\left(
I\theta
+(1-\cos\theta)[\omega]
+(\theta-\sin\theta)[\omega]^2
\right)v\\
0&1
\end{bmatrix}.}
\tag{3.88}
$$

其中旋转块仍由 Rodrigues 公式给出：

$$
e^{[\omega]\theta}
=I+\sin\theta[\omega]
+(1-\cos\theta)[\omega]^2.
$$

若 $\omega=0$ 且 $\|v\|=1$，则

$$
[S]=
\begin{bmatrix}
0&v\\
0&0
\end{bmatrix},
\qquad
[S]^2=0.
$$

因此指数级数在一阶后终止：

$$
\boxed{
e^{[S]\theta}
=I+[S]\theta
=
\begin{bmatrix}
I&v\theta\\
0&1
\end{bmatrix}.}
\tag{3.89}
$$

这正是沿单位方向 $v$ 平移距离 $\theta$。


##### 从瞬时速度到有限位移

上一节的

$$
V=S\dot\theta
$$

描述沿螺旋轴的瞬时速度；

本节的

$$
T(\theta)=e^{[S]\theta}
$$

描述沿同一条固定螺旋轴累计运动参数 $\theta$ 后的有限位姿。对常量 $S$，两者通过微分方程联系：

$$
\frac{dT}{dt}=[S]\dot\theta\,T
\quad\text{或}\quad
\frac{dT}{dt}=T[S]\dot\theta,
$$

具体使用左侧还是右侧，取决于 $S$ 是空间螺旋轴还是物体螺旋轴。


#### 3.3.3.2 刚体运动的矩阵对数

矩阵指数解决“已知螺旋轴和运动量，求位姿”；矩阵对数解决反问题：

$$
\boxed{
e^{[S]\theta}
=T
=
\begin{bmatrix}
R&p\\
0&1
\end{bmatrix}.}
\tag{3.90}
$$

对应的矩阵对数写成

$$
[S]\theta
=
\begin{bmatrix}
[\omega]\theta&v\theta\\
0&0
\end{bmatrix}
\in\mathfrak{se}(3).
$$

其中 $S\theta\in\mathbb R^6$ 是 $T$ 的指数坐标，$[S]\theta$ 是 $T$ 的矩阵对数。

##### 算法分支一：$R=I$

若 $R=I$ 且 $p\ne0$，这是纯平移。取

$$
\boxed{
\omega=0,
\qquad
v=\frac{p}{\|p\|},
\qquad
\theta=\|p\|.}
$$

于是 $v\theta=p$，且

$$
e^{[S]\theta}
=
\begin{bmatrix}I&p\\0&1\end{bmatrix}.
$$

若进一步有 $p=0$，则 $T=I$，其主值矩阵对数是零矩阵；可以取 $\theta=0$，此时归一化螺旋轴 $S$ 不唯一。

##### 算法分支二：$R\ne I$

先使用 $SO(3)$ 的矩阵对数算法，由 $R$ 求出单位旋转轴 $\omega$ 和 $\theta\in(0,\pi]$。旋转块确定后，平移块满足

$$
p=G(\theta)v.
$$

因此

$$
\boxed{v=G^{-1}(\theta)p.}
\tag{3.91}
$$

教材给出

$$
\boxed{
G^{-1}(\theta)
=\frac{1}{\theta}I
-\frac12[\omega]
+\left(
\frac{1}{\theta}
-\frac12\cot\frac{\theta}{2}
\right)[\omega]^2.}
\tag{3.92}
$$

计算流程为：

1. 从 $R$ 求 $\omega,\theta$；
2. 构造 $G^{-1}(\theta)$；
3. 计算 $v=G^{-1}(\theta)p$；
4. 组成 $S=(\omega,v)$ 和 $[S]\theta$；
5. 用 $e^{[S]\theta}=T$ 回代检查。

##### Example 3.26：两个平面参考系之间的螺旋位移

![教材 Figure 3.20：平面中的初始参考系与最终参考系](../../attachments/ch03/fig-3-20-planar-frames.png)

*来源：教材 Figure 3.20，书页 107／本地 PDF 第 126 页；局部截取。*

初始参考系和最终参考系分别为

$$
T_{sb}=
\begin{bmatrix}
\cos30^\circ&-\sin30^\circ&0&1\\
\sin30^\circ&\cos30^\circ&0&2\\
0&0&1&0\\
0&0&0&1
\end{bmatrix},
$$

$$
T_{sc}=
\begin{bmatrix}
\cos60^\circ&-\sin60^\circ&0&2\\
\sin60^\circ&\cos60^\circ&0&1\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}.
$$

要找用空间参考系表示、把 $\{b\}$ 移到 $\{c\}$ 的固定螺旋运动，应写成

$$
T_{sc}=e^{[S]\theta}T_{sb}.
$$

因此先求

$$
\boxed{T_{sc}T_{sb}^{-1}=e^{[S]\theta}.}
$$

左乘说明 $S$ 用固定参考系 $\{s\}$ 表示。矩阵对数算法给出

$$
\boxed{
[S]=
\begin{bmatrix}
0&-1&0&3.37\\
1&0&0&-3.37\\
0&0&0&0\\
0&0&0&0
\end{bmatrix},}
$$

即

$$
\boxed{
S=
\begin{bmatrix}
0\\0\\1\\3.37\\-3.37\\0
\end{bmatrix},
\qquad
\theta=\frac{\pi}{6}=30^\circ.}
$$

由于这是平面内的零节距运动，

$$
\omega=
\begin{bmatrix}0\\0\\1\end{bmatrix},
\qquad
v=
\begin{bmatrix}3.37\\-3.37\\0\end{bmatrix}.
$$

螺旋轴经过

$$
\boxed{
q=
\begin{bmatrix}
3.37\\3.37\\0
\end{bmatrix}.}
$$

用 $v=-\omega\times q$ 检查：

$$
-\begin{bmatrix}0\\0\\1\end{bmatrix}
\times
\begin{bmatrix}3.37\\3.37\\0\end{bmatrix}
=
\begin{bmatrix}3.37\\-3.37\\0\end{bmatrix},
$$

与矩阵对数得到的 $v$ 一致。几何上，这表示 $\{b\}$ 绕固定点 $q$ 逆时针旋转 $30^\circ$ 后到达 $\{c\}$。

对于平面运动，相应的 $\mathfrak{se}(2)$ 矩阵形式为

$$
\begin{bmatrix}
0&-\omega&v_1\\
\omega&0&v_2\\
0&0&0
\end{bmatrix}.
$$

### 3.3 小结

本节建立了从位姿、速度到有限螺旋运动的一条完整链：

$$
T\in SE(3)
\xleftrightarrow[\log]{\exp}
[S]\theta\in\mathfrak{se}(3),
\qquad
V=S\dot\theta.
$$

- 齐次变换矩阵 $T=(R,p)$ 同时描述朝向和位置；
- $T^{-1}\dot T=[V_b]$、$\dot TT^{-1}=[V_s]$ 给出物体旋量和空间旋量；
- $V_s=[\operatorname{Ad}_T]V_b$ 负责旋量换系；
- $S=(\omega,v)$ 是归一化螺旋轴，$V=S\dot\theta$ 是瞬时运动；
- $T=e^{[S]\theta}$ 把螺旋轴积分成有限位姿；
- $\log T=[S]\theta$ 从位姿恢复螺旋轴和运动量。



## 3.4 力旋量（Wrenches）

设一个力 $f$ 作用于刚体上的点 $r$。在参考系 $\{a\}$ 中，力和作用点分别表示为 $f_a,r_a\in\mathbb R^3$，它对 $\{a\}$ 原点产生的力矩（moment）为

$$
\boxed{m_a=r_a\times f_a.}
$$

沿力的作用线移动作用点不会改变力矩。若 $r_a'=r_a+\lambda f_a$，则

$$
r_a'\times f_a
=r_a\times f_a+\lambda(f_a\times f_a)
=r_a\times f_a.
$$

### 六维力旋量

![教材 Figure 3.21：同一力旋量在两个参考系中的表示](../../attachments/ch03/fig-3-21-wrench-representations.png)

*来源：教材 Figure 3.21，书页 109／本地 PDF 第 128 页；局部截取。*

把力矩和力合并为六维空间力，即力旋量（wrench）：

$$
\boxed{
F_a=
\begin{bmatrix}
m_a\\
f_a
\end{bmatrix}
\in\mathbb R^6.}
\tag{3.93}
$$

顺序与旋量 $V=(\omega,v)$ 对齐：前三维是力矩，后三维是力。多个力旋量相加前必须先表示到同一个参考系：

$$
F_{\mathrm{total}}=\sum_iF_i.
$$

若 $f=0$ 而 $m\ne0$，称为纯力矩（pure moment）。纯力矩不依赖参考系原点的位置，只会随坐标轴朝向旋转。

### 用功率不变性推导换系公式

旋量与力旋量的内积是瞬时功率：

$$
\boxed{
P=V^TF
=\omega^Tm+v^Tf.}
$$

其中 $\omega^Tm$ 是转动功率，$v^Tf$ 是平动功率。功率是物理标量，不应因参考系改变而变化，因此

$$
\boxed{V_b^TF_b=V_a^TF_a.}
\tag{3.94}
$$

已知同一旋量的坐标关系

$$
V_a=[\operatorname{Ad}_{T_{ab}}]V_b.
$$

代入功率不变式：

$$
\begin{aligned}
V_b^TF_b
&=([\operatorname{Ad}_{T_{ab}}]V_b)^TF_a\\
&=V_b^T[\operatorname{Ad}_{T_{ab}}]^TF_a.
\end{aligned}
$$

该等式必须对任意 $V_b$ 成立，所以

$$
\boxed{
F_b=[\operatorname{Ad}_{T_{ab}}]^TF_a.}
\tag{3.95}
$$

反方向为

$$
\boxed{
F_a=[\operatorname{Ad}_{T_{ba}}]^TF_b.}
\tag{3.96}
$$

力旋量之所以使用伴随矩阵的转置，是为了与旋量的变换共同保持功率不变。注意变换方向与旋量公式看起来相反：

$$
V_a=[\operatorname{Ad}_{T_{ab}}]V_b,
\qquad
F_b=[\operatorname{Ad}_{T_{ab}}]^TF_a.
$$

### Proposition 3.27

同一个力旋量在 $\{a\}$、$\{b\}$ 中的表示满足

$$
\boxed{
F_b
=\operatorname{Ad}_{T_{ab}}^T(F_a)
=[\operatorname{Ad}_{T_{ab}}]^TF_a,}
\tag{3.97}
$$

$$
\boxed{
F_a
=\operatorname{Ad}_{T_{ba}}^T(F_b)
=[\operatorname{Ad}_{T_{ba}}]^TF_b.}
\tag{3.98}
$$

若 $T_{ab}=(R_{ab},p_{ab})$，则

$$
[\operatorname{Ad}_{T_{ab}}]^T
=
\begin{bmatrix}
R_{ab}^T&-R_{ab}^T[p_{ab}]\\
0&R_{ab}^T
\end{bmatrix}.
$$

所以

$$
\boxed{
f_b=R_{ab}^Tf_a,
\qquad
m_b=R_{ab}^T\bigl(m_a-p_{ab}\times f_a\bigr).}
$$

力只需要旋转坐标；力矩还要修正参考系原点改变所造成的力臂变化。

固定空间参考系中的表示称为空间力旋量（spatial wrench）$F_s$，物体参考系中的表示称为物体力旋量（body wrench）$F_b$。

### Example 3.28：力传感器、机械手与苹果

![教材 Figure 3.22：机械手托住受重力作用的苹果](../../attachments/ch03/fig-3-22-hand-apple-gravity.png)

*来源：教材 Figure 3.22，书页 110／本地 PDF 第 129 页；局部截取。*

苹果质量为 $0.1\ \mathrm{kg}$，机械手质量为 $0.5\ \mathrm{kg}$，取 $g=10\ \mathrm{m/s^2}$。定义：

- $\{f\}$：六轴力—力矩传感器参考系；
- $\{h\}$：机械手质心参考系；
- $\{a\}$：苹果质心参考系。

根据图中的坐标轴，机械手和苹果的重力力旋量分别为

$$
F_h=
\begin{bmatrix}
0\\0\\0\\0\\-5\ \mathrm N\\0
\end{bmatrix},
\qquad
F_a=
\begin{bmatrix}
0\\0\\0\\0\\0\\1\ \mathrm N
\end{bmatrix}.
$$

给定 $L_1=0.10\ \mathrm m$、$L_2=0.15\ \mathrm m$，教材给出的变换为

$$
T_{hf}=
\begin{bmatrix}
1&0&0&-0.10\\
0&1&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix},
$$

$$
T_{af}=
\begin{bmatrix}
1&0&0&-0.25\\
0&0&1&0\\
0&-1&0&0\\
0&0&0&1
\end{bmatrix}.
$$

必须先把两个力旋量都变换到传感器参考系 $\{f\}$，再相加：

$$
\boxed{
F_f
=[\operatorname{Ad}_{T_{hf}}]^TF_h
+[\operatorname{Ad}_{T_{af}}]^TF_a.}
$$

机械手重力的贡献为

$$
[\operatorname{Ad}_{T_{hf}}]^TF_h
=
\begin{bmatrix}
0\\0\\-0.5\ \mathrm{N\,m}\\0\\-5\ \mathrm N\\0
\end{bmatrix},
$$

苹果重力的贡献为

$$
[\operatorname{Ad}_{T_{af}}]^TF_a
=
\begin{bmatrix}
0\\0\\-0.25\ \mathrm{N\,m}\\0\\-1\ \mathrm N\\0
\end{bmatrix}.
$$

所以传感器测得

$$
\boxed{
F_f=
\begin{bmatrix}
0\\
0\\
-0.75\ \mathrm{N\,m}\\
0\\
-6\ \mathrm N\\
0
\end{bmatrix}.}
$$

$-6\ \mathrm N$ 是机械手和苹果的总重力；$-0.75\ \mathrm{N\,m}$ 是两者重力分别通过 $0.10\ \mathrm m$ 和 $0.25\ \mathrm m$ 力臂产生的总力矩：

$$
-(5)(0.10)-(1)(0.25)=-0.75\ \mathrm{N\,m}.
$$

### 3.4 小结

$$
\boxed{
F=
\begin{bmatrix}m\\f\end{bmatrix},
\qquad
P=V^TF,
\qquad
F_b=[\operatorname{Ad}_{T_{ab}}]^TF_a.}
$$

旋量是运动学量，力旋量是与它功率配对的动力学量；伴随矩阵作用于旋量，而其转置作用于力旋量，从而保证功率与参考系选择无关。

## 3.5 总结：旋转与刚体运动的平行结构

教材用一张两页对照表总结了 $SO(3)$ 与 $SE(3)$。核心思想是：三维旋转中的几乎每个对象，在完整刚体运动中都有一个自然推广。

### 群元素：$R$ 与 $T$

| 旋转 | 刚体运动 |
|---|---|
| $R\in SO(3)$，$3\times3$ | $T\in SE(3)$，$4\times4$ |
| $R^TR=I,\ \det R=1$ | $T=\begin{bmatrix}R&p\\0&1\end{bmatrix}$，其中 $R\in SO(3),p\in\mathbb R^3$ |
| $R^{-1}=R^T$ | $T^{-1}=\begin{bmatrix}R^T&-R^Tp\\0&1\end{bmatrix}$ |

坐标变换遵循同样的下标消去规则：

$$
R_{ab}R_{bc}=R_{ac},
\qquad
R_{ab}p_b=p_a,
$$

$$
T_{ab}T_{bc}=T_{ac},
\qquad
T_{ab}p_b=p_a
$$

其中第二行的点 $p_b,p_a$ 使用齐次坐标。

### 主动运动：左乘与右乘

旋转参考系时：

$$
R_{sb'}=RR_{sb}
\quad\Longleftrightarrow\quad
\text{绕空间轴 }\hat\omega_s=\hat\omega\text{ 旋转},
$$

$$
R_{sb''}=R_{sb}R
\quad\Longleftrightarrow\quad
\text{绕物体轴 }\hat\omega_b=\hat\omega\text{ 旋转}.
$$

刚体位移时，令

$$
T=
\begin{bmatrix}
\operatorname{Rot}(\hat\omega,\theta)&p\\
0&1
\end{bmatrix}.
$$

则

$$
T_{sb'}=TT_{sb}
$$

表示先绕空间轴 $\hat\omega_s=\hat\omega$ 旋转，再沿空间坐标中的 $p$ 平移；而

$$
T_{sb''}=T_{sb}T
$$

表示沿物体坐标中的 $p$ 平移，再绕新的物体轴 $\hat\omega_b=\hat\omega$ 旋转。

### 单位轴与瞬时速度

旋转的单位轴是

$$
\hat\omega\in\mathbb R^3,
\qquad
\|\hat\omega\|=1,
$$

角速度为

$$
\omega=\hat\omega\dot\theta.
$$

刚体运动的归一化螺旋轴是

$$
S=
\begin{bmatrix}\omega\\v\end{bmatrix}
\in\mathbb R^6,
$$

其中或者 $\|\omega\|=1$，或者 $\omega=0$ 且 $\|v\|=1$。有限节距螺旋轴 $\{q,\hat s,h\}$ 满足

$$
S=
\begin{bmatrix}
\hat s\\
-\hat s\times q+h\hat s
\end{bmatrix}.
$$

相应旋量为

$$
V=S\dot\theta.
$$

### 李代数矩阵

三维向量 $\omega$ 的反对称矩阵为

$$
[\omega]=
\begin{bmatrix}
0&-\omega_3&\omega_2\\
\omega_3&0&-\omega_1\\
-\omega_2&\omega_1&0
\end{bmatrix}
\in\mathfrak{so}(3).
$$

六维旋量的矩阵表示为

$$
[V]=
\begin{bmatrix}
[\omega]&v\\
0&0
\end{bmatrix}
\in\mathfrak{se}(3).
$$

教材汇总的反对称矩阵恒等式包括

$$
\boxed{
[\omega]=-[\omega]^T,
\qquad
[\omega]x=-[x]\omega,}
$$

$$
\boxed{
[\omega][x]=([x][\omega])^T,
\qquad
R[\omega]R^T=[R\omega].}
$$

### 速度与伴随表示

角速度满足

$$
\dot RR^{-1}=[\omega_s],
\qquad
R^{-1}\dot R=[\omega_b].
$$

旋量满足完全平行的关系：

$$
\dot TT^{-1}=[V_s],
\qquad
T^{-1}\dot T=[V_b].
$$

齐次变换的伴随表示为

$$
[\operatorname{Ad}_T]
=
\begin{bmatrix}
R&0\\
[p]R&R
\end{bmatrix}.
$$

其恒等式为

$$
\boxed{
[\operatorname{Ad}_T]^{-1}
=[\operatorname{Ad}_{T^{-1}}],
\qquad
[\operatorname{Ad}_{T_1}]
[\operatorname{Ad}_{T_2}]
=[\operatorname{Ad}_{T_1T_2}].}
$$

### 坐标表示转换

单位旋转轴和角速度像普通三维向量一样变换：

$$
\hat\omega_a=R_{ab}\hat\omega_b,
\qquad
\omega_a=R_{ab}\omega_b.
$$

螺旋轴和旋量使用伴随表示：

$$
S_a=[\operatorname{Ad}_{T_{ab}}]S_b,
\qquad
V_a=[\operatorname{Ad}_{T_{ab}}]V_b.
$$

### 指数坐标、指数映射与对数映射

旋转矩阵的指数坐标为

$$
\hat\omega\theta\in\mathbb R^3,
$$

其指数映射为

$$
\exp:[\hat\omega]\theta\in\mathfrak{so}(3)
\longrightarrow R\in SO(3),
$$

$$
R=e^{[\hat\omega]\theta}
=I+\sin\theta[\hat\omega]
+(1-\cos\theta)[\hat\omega]^2.
$$

刚体位姿的指数坐标为

$$
S\theta\in\mathbb R^6,
$$

其指数映射为

$$
\exp:[S]\theta\in\mathfrak{se}(3)
\longrightarrow T\in SE(3),
$$

$$
T=e^{[S]\theta}
=
\begin{bmatrix}
e^{[\omega]\theta}
&
\left(
I\theta+(1-\cos\theta)[\omega]
+(\theta-\sin\theta)[\omega]^2
\right)v\\
0&1
\end{bmatrix}.
$$

反方向分别是

$$
\log:R\in SO(3)\longrightarrow[\hat\omega]\theta\in\mathfrak{so}(3),
$$

$$
\log:T\in SE(3)\longrightarrow[S]\theta\in\mathfrak{se}(3).
$$

### 力矩与力旋量换系

纯力矩像普通向量一样变换：

$$
m_a=R_{ab}m_b.
$$

完整力旋量则使用转置伴随：

$$
\boxed{
F_a=
\begin{bmatrix}m_a\\f_a\end{bmatrix}
=[\operatorname{Ad}_{T_{ba}}]^TF_b.}
$$

这里出现 $T_{ba}$ 而不是 $T_{ab}$，是因为旋量和力旋量必须保持功率配对

$$
V_a^TF_a=V_b^TF_b.
$$

### 一张总路线

$$
\boxed{
\begin{array}{ccccc}
\hat\omega
&\xrightarrow{\ \dot\theta\ }&
\omega
&\xrightarrow{\ [\,\cdot\,]\ }&
[\omega]\in\mathfrak{so}(3)\\[2mm]
\downarrow\text{推广}
&&\downarrow\text{推广}
&&\downarrow\text{推广}\\[2mm]
S
&\xrightarrow{\ \dot\theta\ }&
V
&\xrightarrow{\ [\,\cdot\,]\ }&
[V]\in\mathfrak{se}(3)
\end{array}}
$$

有限运动则是

$$
\boxed{
[\hat\omega]\theta
\xrightarrow{\exp}
R\in SO(3),
\qquad
[S]\theta
\xrightarrow{\exp}
T\in SE(3).}
$$

## 3.6 软件（Software）

这一节把前面建立的数学对象映射到教材配套软件。函数名采用教材列出的 MATLAB 形式；其他语言版本保持相同的数学职责，但调用语法可能不同。

### 本节路线图

1. 在向量表示与李代数矩阵表示之间转换。
2. 用指数映射和对数映射在局部运动与有限运动之间转换。
3. 构造、拆分和求逆齐次变换，并计算伴随表示。
4. 从几何螺旋参数构造螺旋轴，再用完整数据流检查函数选择。

### 旋转与 $\mathfrak{so}(3)$

| 函数 | 输入 | 输出与数学含义 |
|---|---|---|
| `RotInv(R)` | $R\in SO(3)$ | $R^{-1}=R^T$ |
| `VecToso3(omg)` | $\omega\in\mathbb R^3$ | $[\omega]\in\mathfrak{so}(3)$ |
| `so3ToVec(so3mat)` | $[\omega]\in\mathfrak{so}(3)$ | $\omega\in\mathbb R^3$ |
| `AxisAng3(expc3)` | 指数坐标 $\hat\omega\theta\in\mathbb R^3$ | 单位旋转轴 $\hat\omega$ 与转角 $\theta$ |
| `MatrixExp3(so3mat)` | $[\hat\omega]\theta\in\mathfrak{so}(3)$ | $R=e^{[\hat\omega]\theta}\in SO(3)$ |
| `MatrixLog3(R)` | $R\in SO(3)$ | $[\hat\omega]\theta=\log R\in\mathfrak{so}(3)$ |

这里最容易混淆的是 `AxisAng3` 与 `MatrixLog3`：前者接收三维指数坐标，负责把“方向乘长度”拆开；后者接收旋转矩阵，负责先求出李代数矩阵。若从 $R$ 出发恢复轴角，完整链条是

$$
R
\xrightarrow{\texttt{MatrixLog3}}
[\hat\omega]\theta
\xrightarrow{\texttt{so3ToVec}}
\hat\omega\theta
\xrightarrow{\texttt{AxisAng3}}
(\hat\omega,\theta).
$$

反方向则为

$$
\hat\omega\theta
\xrightarrow{\texttt{VecToso3}}
[\hat\omega]\theta
\xrightarrow{\texttt{MatrixExp3}}
R.
$$

### 齐次变换的组装、拆分与求逆

| 函数 | 输入 | 输出与数学含义 |
|---|---|---|
| `RpToTrans(R,p)` | $R\in SO(3)$、$p\in\mathbb R^3$ | $T=\begin{bmatrix}R&p\\0&1\end{bmatrix}\in SE(3)$ |
| `TransToRp(T)` | $T\in SE(3)$ | 从 $T$ 中取出 $(R,p)$ |
| `TransInv(T)` | $T\in SE(3)$ | $T^{-1}=\begin{bmatrix}R^T&-R^Tp\\0&1\end{bmatrix}$ |

`TransInv` 不需要通用的 $4\times4$ 矩阵求逆；它直接利用 $SE(3)$ 的结构，因此数学意义更清楚，也避免了不必要的计算。

### 旋量、螺旋轴与 $\mathfrak{se}(3)$

| 函数 | 输入 | 输出与数学含义 |
|---|---|---|
| `VecTose3(V)` | $V=(\omega,v)\in\mathbb R^6$ | $[V]=\begin{bmatrix}[\omega]&v\\0&0\end{bmatrix}\in\mathfrak{se}(3)$ |
| `se3ToVec(se3mat)` | $[V]\in\mathfrak{se}(3)$ | $V=(\omega,v)\in\mathbb R^6$ |
| `Adjoint(T)` | $T=(R,p)\in SE(3)$ | $[\operatorname{Ad}_T]=\begin{bmatrix}R&0\\{}[p]R&R\end{bmatrix}$ |
| `ScrewToAxis(q,s,h)` | 轴上一点 $q$、单位方向 $s$、节距 $h$ | $S=(s,-s\times q+hs)\in\mathbb R^6$ |
| `AxisAng(expc6)` | 指数坐标 $S\theta\in\mathbb R^6$ | 归一化螺旋轴 $S$ 与运动量 $\theta$ |
| `MatrixExp6(se3mat)` | $[S]\theta\in\mathfrak{se}(3)$ | $T=e^{[S]\theta}\in SE(3)$ |
| `MatrixLog6(T)` | $T\in SE(3)$ | $[S]\theta=\log T\in\mathfrak{se}(3)$ |

教材这一版把六维轴角拆分函数写为 `AxisAng(expc6)`；它与三维的 `AxisAng3(expc3)` 作用平行，只是输入对象分别属于 $\mathbb R^6$ 与 $\mathbb R^3$。

从两个位姿求相对螺旋运动时，完整数据流是

$$
T_{bc}=T_{sb}^{-1}T_{sc}
\xrightarrow{\texttt{MatrixLog6}}
[S_b]\theta
\xrightarrow{\texttt{se3ToVec}}
S_b\theta
\xrightarrow{\texttt{AxisAng}}
(S_b,\theta).
$$

这里的乘法顺序决定表达参考系：$T_{sb}^{-1}T_{sc}$ 得到物体坐标表达；若要求空间坐标表达，则使用 $T_{sc}T_{sb}^{-1}$。软件不会替用户判断语义，必须先根据问题选择正确的相对变换。

### 最小工作流示例

若已知

$$
\hat\omega\theta=
\begin{bmatrix}0\\0\\\pi/2\end{bmatrix},
\qquad
p=
\begin{bmatrix}1\\2\\0\end{bmatrix},
$$

先用 `VecToso3` 与 `MatrixExp3` 得到 $R=R_z(\pi/2)$，再用 `RpToTrans` 组装

$$
T=
\begin{bmatrix}
0&-1&0&1\\
1&0&0&2\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}.
$$

反向检查时，`TransToRp` 应恢复相同的 $(R,p)$，`TransInv` 给出逆位姿，而 `MatrixLog6` 给出的不是普通六维速度 $V$，而是包含运动量的李代数矩阵 $[S]\theta$。

### 3.6 小结

函数命名体现了三类不同任务：

- `VecTo...` 与 `...ToVec`：只改变同一对象的表示形式。
- `MatrixExp...` 与 `MatrixLog...`：在李代数的局部运动和李群的有限运动之间转换。
- `RpToTrans`、`TransToRp`、`TransInv`、`Adjoint`：操作位姿结构或由位姿构造换系算子。

因此，做题时应先问“我当前有什么数学对象、需要什么对象”，再选函数，而不是根据变量维数猜函数。

## 3.7 注释与参考文献（Notes and References）

教材补充了本章概念的历史和术语背景：

- 旋转指数坐标在运动学文献中也称欧拉–罗德里格斯参数（Euler–Rodrigues parameters）；欧拉角、Cayley–Rodrigues 参数和单位四元数等其他旋转表示见附录 B。
- 经典螺旋理论（classical screw theory）源自 Mozzi 与 Chasles 关于“绕某轴旋转并沿同轴平移”的发现；本章的 Chasles–Mozzi 定理正是这一几何事实。
- Brockett 将经典螺旋理论与刚体运动的李群（Lie group）$SE(3)$ 结构联系起来，并把开链机构的正运动学写为矩阵指数乘积；这将成为第 4 章的主题。

这一节不引入新的计算公式，但说明了本章路线：$SO(3)$ 与 $SE(3)$ 的指数坐标不仅是描述工具，还会直接成为后续机器人运动学的计算语言。

## 第 3 章收束

本章建立了从几何位姿到速度、有限运动和受力换系的一条完整链条：

$$
\text{参考系与位姿}
\longrightarrow
SO(3),SE(3)
\longrightarrow
\mathfrak{so}(3),\mathfrak{se}(3)
\longrightarrow
\exp/\log
\longrightarrow
\text{旋量与力旋量换系}.
$$

应特别保持以下三组区分：

1. **物理对象与坐标表示**：换参考系不等于移动物体。
2. **瞬时量与有限量**：$V$ 是瞬时速度，$S\theta$ 是有限运动的指数坐标。
3. **运动量与对偶力学量**：旋量用 $[\operatorname{Ad}_T]$ 换系，力旋量用逆方向伴随的转置换系，以保持功率不变。
