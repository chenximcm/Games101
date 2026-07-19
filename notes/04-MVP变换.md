---
title: GAMES101 — Lecture 04 MVP 变换（模型-视图-投影）
date: 2026-07-19
tags: [图形学, GAMES101, 线性代数, 变换, MVP]
description: 深入讲解图形学渲染管线的核心——MVP 变换：模型变换将物体放入世界、视图变换将相机固定到原点、正交与透视投影将三维场景映射到二维屏幕。包含完整的矩阵推导与几何直觉。
---

# GAMES101 现代计算机图形学入门 — Lecture 04 学习笔记

## 课程信息

- **课程名称**：GAMES101 现代计算机图形学入门
- **授课教师**：闫令琪（UCSB 助理教授）
- **本节课主题**：Transformation Cont. — MVP Transformation（模型-视图-投影变换）
- **阅读材料**：虎书《Fundamentals of Computer Graphics》第 7 章（Viewing）
- **B站视频**：[Lecture 04 - Transformation Cont.](https://www.bilibili.com/video/BV1X7411F744?p=4)

---

## 零、前情回顾：从基本变换到 MVP

前三讲完成了从"什么是图形学"到"三维变换基础"的铺垫。上一讲我们掌握了缩放、旋转、平移这些基本变换，以及用齐次坐标和 4×4 矩阵统一表达任意仿射变换。

本讲将把这些"积木"组装起来，回答图形学中最核心的问题之一：

> **如何将一个三维空间中的物体，最终显示到二维屏幕上？**

答案就是 **MVP 变换**，它由三个变换串联组成：

$$
\mathbf{v}_{screen} = \mathbf{M}_{proj} \cdot \mathbf{M}_{view} \cdot \mathbf{M}_{model} \cdot \mathbf{v}_{local}
$$

| 变换 | 作用 | 比喻 |
|------|------|------|
| **Model（模型变换）** | 将物体从自身局部坐标系放到世界坐标系 | 摆放家具 |
| **View（视图变换）** | 将世界坐标变换到以相机为原点的坐标系 | 架好相机 |
| **Projection（投影变换）** | 将三维场景投影到二维平面 | 按下快门 |

Model 变换的本质就是上一讲的内容——平移、旋转、缩放的组合。所以本讲重点放在 **View** 和 **Projection** 两个变换上。

---

## 一、视图变换（View Transformation）

### 1.1 什么是视图变换？

视图变换的核心是**定义相机**。在图形学中，我们做一个关键的约定：

> 📌 **相机永远固定在原点 $(0,0,0)$，朝向 $-Z$ 方向，上方向为 $+Y$。**

为什么要这样约定？因为这样一来，所有物体的坐标都是**相对于相机**来描述的。移动物体和移动相机本质上是等价的（相对运动），但把相机固定能统一后续所有计算。

### 1.2 定义相机需要三个要素

定义一个"相机"只需要三个向量：

- **位置（Position）** $\vec{e}$：相机在世界空间中的位置
- **观察方向（Gaze Direction）** $\hat{g}$：相机看向的方向（单位向量）
- **上方向（Up Direction）** $\hat{t}$：相机的"头顶"朝向（单位向量）

有了 $\hat{g}$ 和 $\hat{t}$，可以通过叉乘得到相机的"右侧"方向：

$$
\hat{r} = \hat{g} \times \hat{t}
$$

这三个向量 $(\hat{r}, \hat{t}, -\hat{g})$ 构成了相机坐标系的三根轴。

### 1.3 视图变换矩阵的推导

**目标**：将任意位置和朝向的相机，变换到原点 $(0,0,0)$，朝向 $-Z$，上方向为 $+Y$。

这个过程分两步：

**Step 1：平移（将相机移到原点）**

$$
\mathbf{T}_{view} =
\begin{pmatrix}
1 & 0 & 0 & -x_e \\
0 & 1 & 0 & -y_e \\
0 & 0 & 1 & -z_e \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

**Step 2：旋转（将相机坐标轴对齐到世界坐标轴）**

将任意朝向的相机坐标轴 $(\hat{r}, \hat{t}, -\hat{g})$ 旋转到标准轴 $(X, Y, Z)$ 比较困难。但反过来——**从标准轴旋转到任意朝向**——非常简单：标准基向量就是目标轴。

所以先写出"标准轴 → 任意朝向"的旋转矩阵：

$$
\mathbf{R}_{view}^{-1} =
\begin{pmatrix}
x_{\hat{r}} & x_{\hat{t}} & x_{-\hat{g}} & 0 \\
y_{\hat{r}} & y_{\hat{t}} & y_{-\hat{g}} & 0 \\
z_{\hat{r}} & z_{\hat{t}} & z_{-\hat{g}} & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

> 🔑 这个矩阵的列就是相机坐标系的三个轴向量——每一列就是一个基向量在标准坐标系下的坐标。

利用旋转矩阵的**正交性**（$\mathbf{R}^{-1} = \mathbf{R}^T$），转置即得正向旋转矩阵：

$$
\mathbf{R}_{view} =
\begin{pmatrix}
x_{\hat{r}} & y_{\hat{r}} & z_{\hat{r}} & 0 \\
x_{\hat{t}} & y_{\hat{t}} & z_{\hat{t}} & 0 \\
x_{-\hat{g}} & y_{-\hat{g}} & z_{-\hat{g}} & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

**最终视图矩阵**：

$$
\mathbf{M}_{view} = \mathbf{R}_{view} \cdot \mathbf{T}_{view}
$$

> 💡 在实践（OpenGL / glm）中，你不需要手写这个矩阵——`glm::lookAt(eye, center, up)` 帮你搞定一切。但理解它的推导过程，才能理解后续的坐标空间概念。

---

## 二、投影变换（Projection Transformation）

投影变换的目标是将三维物体映射到二维平面。图形学中有两种投影方式：

### 2.1 正交投影（Orthographic Projection）

**特点**：没有近大远小，平行线投影后依然平行。

**核心思想**：定义一个三维空间中的长方体（立方体）$[l, r] \times [b, t] \times [f, n]$，将其映射到标准立方体 $[-1, 1]^3$（称为 **Canonical Cube**）。

> ⚠️ 注意：在右手坐标系中，沿 $-Z$ 方向看，**近平面 $n$ 大于远平面 $f$**（因为近平面离相机更近，Z 坐标值更大）。这和日常直觉相反，是图形学中常见的坑。

两步操作：

**① 平移：将立方体中心移到原点**

$$
\mathbf{T}_{ortho} =
\begin{pmatrix}
1 & 0 & 0 & -\frac{r+l}{2} \\
0 & 1 & 0 & -\frac{t+b}{2} \\
0 & 0 & 1 & -\frac{n+f}{2} \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

**② 缩放：将边长变为 2（$[-1,1]$ 范围）**

$$
\mathbf{S}_{ortho} =
\begin{pmatrix}
\frac{2}{r-l} & 0 & 0 & 0 \\
0 & \frac{2}{t-b} & 0 & 0 \\
0 & 0 & \frac{2}{n-f} & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

**最终正交投影矩阵**：

$$
\mathbf{M}_{ortho} = \mathbf{S}_{ortho} \cdot \mathbf{T}_{ortho} =
\begin{pmatrix}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{2} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{2} \\
0 & 0 & \frac{2}{n-f} & -\frac{n+f}{2} \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

### 2.2 透视投影（Perspective Projection）

**特点**：近大远小，平行线在远处汇聚（消失点），更接近人眼和相机的真实效果。

透视投影的推导是本节课最精彩的部分。核心思路——**"分两步走"**：

> 🔑 **Step 1**：将透视投影的视锥体（Frustum）"挤压"成一个正交投影的长方体
> **Step 2**：在这个长方体上做正交投影

### 2.3 推导"挤压矩阵"（Persp → Ortho）

从侧面看（YZ 平面），视锥体是一个梯形。对于视锥体内的任意一点 $(x, y, z)$，利用相似三角形：

$$
x' = \frac{n}{z} \cdot x, \quad y' = \frac{n}{z} \cdot y
$$

这意味着挤压后的坐标与 $z$ 成反比——这正是"近大远小"的数学本质。

> 💡 回顾齐次坐标的性质：$(x, y, z, 1)$ 等价于 $(kx, ky, kz, k)$。如果我们让变换后的 $w$ 分量等于 $z$，归一化（除以 $w$）时自然就实现了 $\frac{1}{z}$ 的缩放！

由此构造出挤压矩阵：

$$
\mathbf{M}_{persp \to ortho} =
\begin{pmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0
\end{pmatrix}
$$

**验证——近平面上的点不变**：

$$
\begin{pmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0
\end{pmatrix}
\begin{pmatrix} x \\ y \\ n \\ 1 \end{pmatrix}
= \begin{pmatrix} nx \\ ny \\ n^2 \\ n \end{pmatrix}
\rightarrow \begin{pmatrix} x \\ y \\ n \\ 1 \end{pmatrix}
$$

✅ 近平面上的点 $(x, y, n)$ 变换后不变。

**验证——远平面中心点不变**：

$$
\begin{pmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0
\end{pmatrix}
\begin{pmatrix} 0 \\ 0 \\ f \\ 1 \end{pmatrix}
= \begin{pmatrix} 0 \\ 0 \\ f^2 \\ f \end{pmatrix}
\rightarrow \begin{pmatrix} 0 \\ 0 \\ f \\ 1 \end{pmatrix}
$$

✅ 远平面中心点 $(0, 0, f)$ 也不变。挤压过程保持了远近两个平面的位置和大小。

### 2.4 最终透视投影矩阵

$$
\mathbf{M}_{persp} = \mathbf{M}_{ortho} \cdot \mathbf{M}_{persp \to ortho}
$$

在实践（OpenGL / glm）中，等价于 `glm::perspective(fov, aspect, near, far)`。

---

## 三、透视投影中 Z 坐标的处理

经过透视投影后，Z 坐标不再线性分布。挤压矩阵中第 $(3,3)$ 和 $(3,4)$ 项（$n+f$ 和 $-nf$）使得 Z 值分布变为非线性：

> 离相机越近，Z 精度越高；离相机越远，Z 精度越低。

这是 **Z-fighting**（深度冲突）的根源——远处的两个面因为 Z 精度不足，无法正确判断遮挡关系，产生闪烁。

> 💡 工程实践：尽量把近平面设远一点，远平面设近一点，让深度范围更紧凑，能有效缓解 Z-fighting。

Z 值在整个渲染管线中始终保留，用于后续的**深度测试（Z-buffer）**——判断像素之间的前后遮挡关系。

---

## 四、变换流程总览

一个顶点的完整变换路径：

```mermaid
graph LR
    A["局部坐标<br/>(x,y,z,1)"] -->|"Model<br/>摆家具"| B["世界坐标"]
    B -->|"View<br/>架相机"| C["相机坐标"]
    C -->|"Projection<br/>按快门"| D["裁剪坐标"]
    D -->|"透视除法<br/>÷w"| E["NDC<br/>[-1,1]³"]
    E -->|"视口变换"| F["屏幕坐标<br/>(像素)"]

    style A fill:#dbeafe,stroke:#2563eb
    style D fill:#fef3c7,stroke:#f59e0b
    style E fill:#fef3c7,stroke:#f59e0b
    style F fill:#d1fae5,stroke:#10b981
```

> 🔑 从裁剪坐标到 NDC 的这一步叫**透视除法（Perspective Division）**——将 $(x, y, z, w)$ 除以 $w$ 得到 $(x/w, y/w, z/w, 1)$。这一步是自动完成的（GPU 硬件实现），但理解它才能理解透视投影为什么能产生近大远小的效果。

---

## 五、个人总结与思考

第四课是 GAMES101 的**第一个里程碑**——MVP 变换构成了整个图形学渲染管线的入口。这节课给我的几个核心收获：

1. **变换的层次化思维**：Model、View、Projection 三个矩阵各司其职，层层递进。这种"分而治之"的设计让复杂的渲染管线变得模块化——你想改变物体位置就改 Model 矩阵，想改变相机角度就改 View 矩阵，互不干扰。

2. **旋转矩阵正交性的妙用**：视图旋转矩阵的推导中，"先求逆再转置"的技巧非常优雅。正交矩阵 $\mathbf{R}^{-1} = \mathbf{R}^T$ 这个性质在这门课里反复出现，是数学性质在工程中发挥威力的绝佳例子。

3. **透视投影的"挤压"思路**：将一个复杂的透视问题分解为"挤压 + 正交"两步，是一种典型的"化归"思维。而且利用齐次坐标的 $w$ 分量来"延迟做除法"，在数学上非常巧妙——在 MVP 阶段保留 $w$，让 GPU 在透视除法阶段自动完成归一化。

4. **Z 值非线性的工程影响**：透视投影对 Z 的"压缩"不是免费的——它直接导致远处物体的深度精度下降。Z-fighting 这个在游戏里常见的 Bug（远处的山在闪烁），根源就在这里。理解了原理，调参时就知道该动哪些参数。

5. **图形成像的本质**：MVP 变换本质上是在做一件事——把三维世界中的一个点"拍扁"到二维屏幕上的某个像素。整个管线就是一个长链的坐标变换，而这节课讲的是其中最核心的三步。

下一课将进入**光栅化（Rasterization）** ——真正开始把 3D 场景"画"到屏幕上，包括三角形采样、像素着色和抗锯齿等。这才是图形学"魔法发生的地方"。🚀

---

## 🎬 课程视频

- **B站链接**：[Lecture 04 - Transformation Cont.](https://www.bilibili.com/video/BV1X7411F744?p=4)

<iframe src="//player.bilibili.com/player.html?bvid=BV1X7411F744&p=4" 
        scrolling="no" 
        border="0" 
        frameborder="no" 
        framespacing="0" 
        allowfullscreen="true" 
        width="100%" 
        height="500">
</iframe>

---

**参考资料**：
- GAMES101 Lecture 04 课程视频
- 闫令琪老师课程课件
- 虎书《Fundamentals of Computer Graphics》第 7 章
