
许多机器学习算法都在优化一个目标函数，即相对于一组模型参数进行优化，这些参数控制着模型解释数据的好坏。如何寻找好的参数可被表述为一个优化问题（见 8.2 节和 8.3 节）。优化的例子包括：
1. 线性回归（见第9章），我们研究曲线拟合问题，并优化线性权重参数以最大化可能性；
2. 神经网络自编码器用于降维和数据压缩，其中参数是每层的权重和偏差，我们通过反复应用链式法则来最小化重建误差；
3.  Gauss 混合模型（见第11章）用于建模数据分布，我们优化每个混合组件的位置和形状参数，以最大化模型的可能性。
图5.1展示了我们通常使用利用梯度信息（第7.1节）的优化算法来解决这些问题。图5.2概述了本章概念之间以及它们与书中其他章节的联系。

本证的核心概念是函数。一个函数 $f$ 是一个数学对象，它将两个数学对象进行联系。本书中涉及的数学对象即为模型输入 $\boldsymbol{x} \in \mathbb{R}^{D}$ 以及拟合目标（函数值）$f(\boldsymbol{x})$，若无额外说明，默认拟合目标都是实数。这里 $\mathbb{R}^{D}$ 称为 $f$ 的**定义域（domain）**，而相对应的函数值 $f(\boldsymbol{x})$ 所在的集合被称为 $f$ 的**像集（image）或陪域（codomain）**。

![](attachments/Pasted%20image%2020240825122538.png)
<center>图 5.1 向量微积分在 (a) 回归问题（曲线拟合）和 (b) 分布密度估计（建模数据分布） <br>中有重要应用。</center>

![](attachments/Pasted%20image%2020240825122711.png)
<center>图 5.2 本章的概念地图及与其他章节的联系</center>

2.7.3 节中有对线性函数更为细致的讨论，但一般而言，我们将函数写为下面的形式
$$
\begin{align}
f : \mathbb{R}^D &\to \mathbb{R}\tag{5.1a}\\
\boldsymbol{x} &\mapsto f(\boldsymbol{x}) \tag{5.1b}
\end{align}
$$
其中 $(5.1a)$ 说的是 $f$ 是一个由 $\mathbb{R}^{D}$ 至 $\mathbb{R}$ 的映射，而 $(5.2b)$ 指的是 $f$ 将每一个输入 $\boldsymbol{x}$ 对应于唯一的函数值 $f(\boldsymbol{x})$。

> **示例 5.1**
> 请回忆在 3.2 节中我们谈到点积是一种特殊地内积。沿用之前的记号，函数 $f(\boldsymbol{x}) = \boldsymbol{x}^{\top}\boldsymbol{x}, \boldsymbol{x} \in \mathbb{R}^{2}$ 相当于
> $$\begin{align}f: \mathbb{R}^{2} & \rightarrow \mathbb{R}\tag{5.2a}\\\boldsymbol{x} &\mapsto x_{1}^{2} + x_{2}^{2}. \tag{5.2b}\end{align}$$

本章将介绍如何计算函数的梯度——这在机器学习中如何充分利用*学习*非常重要，因为梯度指向函数值提升的最陡峭方向。所以向量微积分是机器学习中所需的基础数学工具。我们在全书中都默认函数是可微的，但若具备一些尚未提及的额外定义，很多机器学习方法可被扩展至**次梯度（sub-differentials，当函数连续但在某些点不可微时）**。我们将在第七章探讨带有条件限制的此类函数。

## 5.1 一元函数的微分

接下来我们简要复习一下学过高中数学读者较为熟悉的一元函数的微分。我们从用于定义微分的重要概念——一元函数 $y = f(x), x,y \in \mathbb{R}$ 的差商开始。

![](attachments/Pasted%20image%2020240827204933.png)
<center>图 5.3 </center>

> **定义 5.1 (差商）**
> 一元函数的差商$$\frac{\delta y}{\delta x} := \frac{f(x + \delta x) - f(x)}{\delta x} \tag{5.3}$$
> 计算连接函数 $f$ 之图像上的两点的割线之斜率。如图 5.3 所示，两点的横坐标分别为 $x_{0}$ 和 $x_{0} + \delta x$。

若 $f$ 是线性函数，差商也可以看做函数 $f$ 上从点 $x$ 至 $x + \delta x$ 之间的平均斜率。若对 $\delta x$ 去极限 $\delta x \rightarrow 0$，我们得到了 $f$ 在 $x$ 处的切线斜率；如果 $f$ 可微，这个切线斜率就是 $f$ 在 $x$ 处的导数。

> **定义 5.2 （导数）**
> 对正实数 $h > 0$，函数 $f$ 在 $x$ 处的导数由下面的极限定义：$$\frac{\mathrm{d}f}{\mathrm{d}x} := \lim_{h \to 0} \frac{f(x + h) - f(x)}{h} \tag{5.4}$$
> 对应到图 5.3 中，割线将变为切线。

$f$ 的导数时刻指向 $f$ 提升最快的方向。

> **示例 5.2 （多项式函数的导数）**
> 我们想计算多项式函数 $f(x) = x^{n}, n \in \mathbb{N}$ 的导数。即使我们可能已经知道答案是 $n x^{n-1}$，但我们依然希望使用导数和差商的定义导出它。
> 使用导数的定义 $(5.4)$，我们有
> $$\begin{align}\frac{\mathrm{d}f}{\mathrm{d}x} &= \lim_{h \to 0} \frac{{\color{blue} f(x+h)} - {\color{orange} f(x)}}{h} \tag{5.5a}\\&= \lim_{h \to 0} \frac{{\color{blue} (x + h)^n} - {\color{orange} x^n}}{h} \tag{5.5b}  \\&= \lim_{h \to 0} \frac{{\color{blue}\sum\limits_{i=0}^{n} \binom{n}{i} x^{n-i} h^i} - {\color{orange}x^n}}{h}. \tag{5.5c}\end{align}$$
> 注意到 $\displaystyle {\color{orange} x^{n}} = \binom{n}{i} x^{n-0}h^{0}$，所以上式分子相当于求和项从 $1$ 开始，于是上式变为$$\begin{align} \frac{\mathrm{d}f}{\mathrm{d}x} &= \lim_{ h \to 0 } \frac{\sum\limits_{i=1}^{n} \binom{n}{i} x^{n-i}h^{i}}{h} \tag{5.6a}\\ &= \lim_{ h \to 0 } \sum\limits_{i=1}^{n} \binom{n}{i} x^{n-i}h^{i-1} \tag{5.6b}\\ &= \lim_{ h \to 0 } \left( \binom{n}{1} x^{n-1} + \underbrace{\sum\limits_{i=2}^{n} \binom{n}{i} x^{n-i}h^{i-1}}_{\to 0 \text{ as } h \rightarrow 0} \right) \tag{5.6c}\\ &= \frac{n!}{1!(n-1)!}x^{n-1} = nx^{n-1}. \tag{5.6d} \end{align}$$

### 5.1.1 Taylor 级数

所谓 Taylor 级数是将函数 $f$ 表示成的那个无限项求和式，其中的所有的项都和 $f$ 在点 $x_{0}$ 处的导数相关。

> **定义 5.3（Taylor 多项式）**
> 函数 $f: \mathbb{R} \rightarrow \mathbb{R}$ 在点 $x_{0}$ 的 $n$ 阶 Taylor 多项式是 $$T_n(x) := \sum_{k=0}^{n} \frac{f^{(k)}(x_0)}{k!} (x - x_0)^k, \tag{5.7}$$其中 $f^{(k)}(x_{0})$ 是 $f$ 在 $x_{0}$ 处的 $k$ 阶导数（假设其存在），而 $\displaystyle \frac{f^{(k)}(x_{0})}{k!}$ 是多项式各项的系数。

> 对于所有的 $t \in \mathbb{R}$ 我们约定 $t^{0} := 1$

> **定义 5.4（Taylor 级数）**
> 对于光滑函数 $f \in \mathcal{C}^{\infty}, f: \mathbb{R}\rightarrow \mathbb{R}$，它在点 $x_{0}$ 处的 Taylor 级数定义为$$T_\infty(x) = \sum_{k=0}^{\infty} \frac{f^{(k)}(x_0)}{k!} (x - x_0)^k. \tag{5.8}$$若 $x_{0} = 0$，我们得到了一个 Taylor 级数的特殊情况 —— Maclaurin 级数。如果 $f(x) = T_{\infty}(x)$，则我们称 $f$ 是**解析函数**。

> 注：一般而言，某个不一定为多项式函数的 $n$ 阶 Taylor 多项式是这个函数的近似，它在 $x_{0}$ 的邻域中与 $f$ 接近。事实上，对于阶数为 $k \leqslant n$ 的多项式函数 $f$，$n$ 阶 Taylor 多项式就是这个多项式函数本身，因为对所有的 $i > k$，多项式函数 $f$ 的 $i$ 阶导数 $f^{(i)}$ 均为零。

> **示例 5.3（Taylor 多项式）**
> 考虑多项式 $$f(x) = x^{4} \tag{5.9}$$并求它在 $x_{0} = 1$ 处的 Taylor 多项式 $T_{6}$。我们先求函数在该点的各阶导数 $f^{(k)}(1), k=0, \dots, 6$：$$\begin{align}f(1) &= 1 \tag{5.10} \\f'(1) &= 4 \tag{5.11} \\f''(1) &= 12 \tag{5.12} \\f^{(3)}(1) &= 24 \tag{5.13} \\f^{(4)}(1) &= 24 \tag{5.14} \\f^{(5)}(1) &= 0 \tag{5.15} \\f^{(6)}(1) &= 0 \tag{5.16}\end{align}$$于是所求的 Taylor 多项式为$$\begin{align}T_6(x) &= \sum_{k=0}^{6} \frac{f^{(k)}(x_0)}{k!} (x - x_0)^k \tag{5.17a}\\&= 1 + 4(x-1) + 6(x-1)^{2} + 4(x-1)^{3} + (x-1)^{4} + 0.\tag{5.17b}\end{align}$$整理得到$$\begin{align}T_{6}(x) &= (1-4+6-4+1) + x(4-12+12-4) \\&~ ~ ~+ x^{2}(6-12+6) + x^{3}(4-4) + x^{4}\tag{5.18a}\\&= x^{4} = f(x).\tag{5.18b}\end{align}$$
> 我们得到了和原函数一模一样的 Taylor 多项式。

![](attachments/Pasted%20image%2020240829103047.png)
<center>图 5.4</center>

> **示例 5.4（Taylor 级数）**
> 如图 5.4，考虑函数$$f(x) = \sin (x) + \cos(x) \in \mathcal{C}^{\infty}.$$我们计算其在 $x_{0} = 0$ 处的 Taylor 级数，也就是 Maclaurin 级数。我们可以求得 $f$ 在 $0$ 处的各阶导数：$$\begin{align} f(0) &= \sin(0) + \cos(0) = 1 \tag{5.20}\\ f'(0) &= \cos(0) - \sin(0) = 1 \tag{5.21}\\f''(0) &= -\sin(0) - \cos(0) = -1 \tag{5.22}\\f^{(3)}(0) &= -\cos(0) + \sin(0) = -1 \tag{5.23}\\f^{(4)}(0) &= \sin(0) + \cos(0) = f(0) = 1\tag{5.24} \\ &\vdots \end{align}$$从上面的结果我们可以找到一些规律。首先由于 $\sin(0)= 0$，有级数的各项系数只能为 $\pm {1}$，其中每个不同的值在切换为其相反数时都连续出现两次，进而有 $f^{(k+4)}(0)= f^{(k)}(0)$。
> 因此我们可以得到函数 $f$ 在 $x_{0} = 0$ 处的 Taylor 级数：$$\begin{align} T_{\infty}(x) &= \sum\limits_{k=0}^{\infty} \frac{f^{(k)}(x_{0})}{k!}(x-x_{0})^{k}\tag{5.25a}\\ &= 1 + x - \frac{1}{2!}x^{2} - \frac{1}{3!}x^{3} + \frac{1}{4!}x^{4} + \frac{1}{5!}x^{5} - \cdots\tag{5.25b}\\ &= {\color{orange} 1 - \frac{1}{2!}x^{2} + \frac{1}{4!}x^{4} \mp \cdots } + {\color{blue} x - \frac{1}{3!}x^{3} + \frac{1}{5!}x^{5} \mp \cdots }\tag{5.25c}\\ &= {\color{orange} \sum\limits_{k=0}^{\infty} \frac{(-1)^{k}}{(2k)!}x^{2k} } + {\color{blue} \sum\limits_{k=0}^{\infty} \frac{(-1)^{k}}{(2k+1)!}x^{2k+1} } \tag{5.25d} \\ &= {\color{orange} \cos(x) } + {\color{blue} \sin(x) } ,\tag{5.25e} \end{align} $$其中我们使用了三角函数的幂级数表示：$$\begin{align} \cos(x) &= \sum_{k=0}^{\infty} \frac{(-1)^k}{(2k)!}x^{2k}, \tag{5.26}\\ \sin(x) &= \sum_{k=0}^{\infty} \frac{(-1)^k}{(2k + 1)!}x^{2k+1}. \tag{5.27} \end{align}$$图 5.4 展示了上述条件下的前几个 Taylor 多项式 $T_{n}$，其中 $n=0,1,5,10$。

> 注：Taylor 级数是一种特殊形式的幂级数：$$f(x) = \sum\limits_{k=0}^{\infty} a_{k}(x-c)^{k},\tag{5.28}$$其中 $a_{k}$ 是系数，$c$ 是常数。不难看出这与定义 5.4 形式的一致性。

### 5.1.2 微分法则

下面我们简要介绍基本的微分法则，其中我们使用 $f'$ 表示 $f$ 的导数。
$$
\begin{align}
\text{乘法法则:}\quad & [f(x)g(x)]' = f'(x)g(x) + f(x)g'(x) \tag{5.29}\\[0.2em]
\text{除法法则:}\quad & \left[ \frac{f(x)}{g(x)} \right]' = \frac{f'(x)g(x)-f(x)g'(x)}{\big[ g(x) \big]^{2}}\tag{5.30}\\[0.2em]
\text{加法法则:}\quad & [f(x) + g(x)]' = f'(x) + g'(x) \tag{5.31}\\[0.2em]
\text{链式法则:}\quad & \Big( g\big[ f(x) \big] \Big)' = (g \circ f)'(x) = g'\big[f(x)\big]f'(x)\tag{5.32}
\end{align}
$$
其中 $g \circ f$ 表示函数的复合：$x \mapsto f(x) \mapsto g\big[f(x)\big]$。

> **示例 5.5（链式法则）**
> 使用链式法则计算函数 $h(x) = (2x+1)^{4}$ 的导数。
> 不难看出$$\begin{align}h(x) &= (2x+1)^{4} = g \big[ f(x) \big], \tag{5.33}\\ 
f(x) &= 2x+1, \tag{5.34}\\g(f) &= f^{4}, \tag{5.35}\end{align}$$然后计算 $f$ 和 $g$ 的导数：$$\begin{align}f'(x) &= 2,\tag{5.36}\\g'(f) &= 4f^{3},\tag{5.37}\\\end{align}$$这样我们就得到 $h$ 的导数：$$h'(x) {~}\mathop{=\!=\!=}\limits^{（5.32}{~} g'(f)f'(x) = (4f^{3})\cdot2 {~}\mathop{=\!=\!=}\limits^{(5.34)}{~}4(2x+1)^{3} \cdot 2 = 8(2x+1)^{3}, \tag{5.38}$$其中我们用到了链式法则，并在 $g'(f)$ 中的 $f$ 代换为 $(5.34)$ 中的表达式。


## 5.2 偏导数和梯度

在 5.1 节中讨论了标量元 $x \in \mathbb{R}$ 的函数 $f$ 的微分之后，本节将考虑函数 $f$ 的自变量含有多个元的一般情形，即 $\boldsymbol{x} \in \mathbb{R}^{n}$；例如 $f(x_{1}, x_{2})$。相应地，函数的导数就推广到多元情形就变成了**梯度**。

我们可以通过保持其他变量不动，然后改变变元 $x$ 来获取函数的梯度：将对各变元的偏导数组合起来。

> **定义 5.5（偏导数）**
> 给定 $n$ 元函数 $f: \mathbb{R}^{n} \rightarrow \mathbb{R}$，$\boldsymbol{x} \mapsto f(\boldsymbol{x}), \boldsymbol{x} \in \mathbb{R}^{n}$，它的各偏导数为$$\begin{align}\frac{ \partial f }{ \partial x_{1} } &= \lim_{ h \to 0 } \frac{f(x_{1}+h, x_{2}, \dots, x_{n}) - f(\boldsymbol{x})}{h}\\&\,\,\, \vdots\\\frac{ \partial f }{ \partial x_{n} } &= \lim_{ h \to 0 } \frac{f(x_{1}, \dots, x_{n-1}, x_{n}+h) - f(\boldsymbol{x})}{h}\end{align}\tag{5.39}$$ 然后将各偏导数组合为向量，就得到了梯度向量$$\nabla_{x}f = \text{grad} f = \frac{\mathrm{d}f}{\mathrm{d}\boldsymbol{x}} = \left[ \frac{ \partial f(\boldsymbol{x}) }{ \partial x_{1} }, \frac{ \partial f(\boldsymbol{x}) }{ \partial x_{2} }, \dots, \frac{ \partial f(\boldsymbol{x}) }{ \partial x_{n} } \right] \in \mathbb{R}^{1 \times n}, \tag{5.40}$$其中 $n$ 是变元数，$1$ 是 $f$ 像集（陪域）的维数。我们在此定义列向量 $\boldsymbol{x} = [x_{1}, \dots, x_{n}]^{\top} \in \mathbb{R}^{n}$。行向量 $(5.40)$ 称为 $f$ 的**梯度**或者**Jacobian 矩阵**，是 5.1 节中的导数的推广。

> 注：此处的 Jacobian 矩阵是其特殊情况。在 5.3 节中我们将讨论向量值函数的 Jacobian 矩阵。

> 译者注：可以看到，梯度向量是一个线性变换：$D: \mathbb{R}^{n} \rightarrow \mathbb{R}$。这样的行向量又被称为 **余向量（covector）**，其中的 余（co-）表示行和列的对偶关系。

> **示例 5.6（使用链式法则计算偏导数）**
> 给定函数 $f(x,y) = (x + 2y^{3})^{2}$，我们可以这样计算它的偏导数：$$\begin{align}\frac{ \partial f(x,y) }{ \partial x } &= 2(x+2y^{3}) \cdot \frac{ \partial  }{ \partial x } (x + 2y^{3}) = 2(x+2y^{3}), \tag{5.41} \\\frac{ \partial f(x,y) }{ \partial y } &= 2(x+2y^{3}) \cdot \frac{ \partial  }{ \partial y } (x + 2y^{3}) = 12(x+2y^{3})y^{2}. \tag{5.42}\end{align}$$上述过程中我们使用了链式法则 $(5.32)$。

> 注（作为行向量的梯度）：文献中并不常像一般的向量表示那样将梯度写为列向量。这样做的原因有两个：首先，这样的定义方便拓展为向量值函数 $f: \mathbb{R}^{n} \rightarrow \mathbb{R}^{m}$ 的情形，这样梯度就变为矩阵；其次，我们可以方便地对其使用多变元的链式法则而不用注意梯度的维数。我们将在 5.3 节中进一步讨论以上两点。

> **示例 5.7（梯度）**
> 给定函数 $f(x,y) = x_{1}^{2}x_{2} + x_{1}x_{2}^{3} \in \mathbb{R}$，它的各偏导数（相对于 $x_{1}$ 和 $x_{2}$ 求偏导）为$$\begin{align}
\frac{ \partial f(x_{1}, x_{2}) }{ \partial x_{1} } &= 2x_{1}x_{2} + x_{2}^{3} \tag{5.43}\\
\frac{ \partial f(x_{1}, x_{2}) }{ \partial x_{2} } &= x_{1}^{2} + 3 x_{1}x_{2}^{2} \tag{5.44} 
\end{align}$$于是我们可以得到梯度$$\frac{\mathrm{d}f}{\mathrm{d}\boldsymbol{x}} = \left[ \frac{ \partial f(x_{1},x_{2}) }{ \partial x_{1} } , \frac{ \partial f(x_{1}, x_{2}) }{ \partial x_{2} }  \right] = [2x_{1}x_{2} + x_{2}^{3}, x_{1}^{2} + 3x_{1}x_{2}^{2}] \in \mathbb{R}^{1 \times 2}. \tag{5.45}$$

### 5.2.1 偏导数的基本法则

当 $\boldsymbol{x} \in \mathbb{R}^{n}$ 时，即在多元函数的情况下的微分法则（如加法、乘法、链式法则）和我们在学校中学到的无异。但在对向量 $\boldsymbol{x} \in \mathbb{R}^{n}$ 求导时，我们需要额外注意，因为我们现在得到的梯度包括向量和矩阵，而矩阵乘法是非交换的。
下面是一般的加法、乘法、和链式法则：

$$
\begin{align}
\text{Product rule:}~&~\frac{ \partial  }{ \partial \boldsymbol{x} } \big[ f(\boldsymbol{x})g(\boldsymbol{x}) \big] = \frac{ \partial f }{ \partial \boldsymbol{x} }g(\boldsymbol{x}) + f(\boldsymbol{x})\frac{ \partial g }{ \partial \boldsymbol{x} } \tag{5.46}\\
\text{Sum rule:}~&~\frac{ \partial  }{ \partial \boldsymbol{ x }  } \big[ f(\boldsymbol{x}) + g(\boldsymbol{x})\big] = \frac{ \partial f }{ \partial \boldsymbol{ x }  } \frac{ \partial g }{ \partial \boldsymbol{ x }  } \tag{5.47}\\
\text{Chain rule:}~&~\frac{ \partial  }{ \partial \boldsymbol{ x }  } (g \circ f)(x) = \frac{ \partial  }{ \partial \boldsymbol{ x }  } g\big[ f(\boldsymbol{ x } ) \big] = \frac{ \partial g }{ \partial f } \frac{ \partial f }{ \partial \boldsymbol{ x }  } \tag{5.48} 
\end{align}
$$

我们仔细观察链式法则 $(5.48)$，可以通过它看到相应矩阵乘法的规律，即相邻相乘矩阵的相邻维度需要相等（见 2.2.1 节）。从左往右看，可以发现 $\partial f$ 先出现在第一项的“分母”，然后出现在第二项“分子”，按照通常乘法的定义可以理解，$\partial f$ 对应的维数对应则可以消去，剩下的就是 $\displaystyle \frac{ \partial g }{ \partial \boldsymbol{ x } }$。

> 注意，$\displaystyle \frac{ \partial f }{ \partial \boldsymbol{ x } }$ 并不是严格意义上的分数，上述说法只是为了增进理解

### 5.2.2 链式法则（chain rule）

考虑变元为 $x_{1}, x_{2}$ 函数 $f: \mathbb{R}^{2} \rightarrow \mathbb{R}$，而 $x_{1}(t)$ 和 $x_{2}(t)$ 又是变元 $t$ 的函数。为了计算 $f$ 对 $t$ 的梯度，需要用到链式法则 $(5.48)$：

$$
\frac{\mathrm{d}f}{\mathrm{d}t} = \begin{bmatrix}
\displaystyle \frac{ \partial f }{ \partial x_{1} } & \displaystyle \frac{ \partial f }{ \partial x_{2} }  
\end{bmatrix} \begin{bmatrix}
\displaystyle \frac{ \partial x_{1}(t) }{ \partial t }\\
\displaystyle \frac{ \partial x_{2}(t) }{ \partial t }\\
\end{bmatrix} = \frac{ \partial f }{ \partial x_{1} } \frac{ \partial x_{1} }{ \partial t }  + \frac{ \partial f }{ \partial x_{2} } \frac{ \partial x_{2} }{ \partial t },\tag{5.49} 
$$
其中 $\mathrm{d}$ 表示梯度，而 $\partial$ 表示偏导数。

> **示例 5.8**
> 考虑函数 $f(x_{1}, x_{2}) = x_{1}^{2} + 2x_{2}$，其中 $x_{1} = \sin t$，$x_{2} = \cos t$，则 $$\begin{align}\frac{\mathrm{d}f}{\mathrm{d}t} &= \frac{ \partial f }{ \partial x_{1} } \frac{ \partial x_{1} }{ \partial t } + \frac{ \partial f }{ \partial x_{2} } \frac{ \partial x_{2} }{ \partial t } \tag{5.50a}\\&= 2\sin t \frac{ \partial \sin t }{ \partial t } + 2 \frac{ \partial \cos t }{ \partial t } \tag{5.50b}\\&= 2\sin t \cos t - 2\sin t = 2\sin t(\cos t-1)\tag{5.50c}\end{align}$$就是 $f$ 关于 $t$ 的梯度。

如果 $f(x_{1}, x_{2})$ 是 $x_{1}$ 和 $x_{2}$ 的函数，而 $x_{1}(s, t)$ 和 $x_{2}(s,t)$ 又分别为 $s$ 和 $t$ 的函数，那么根据链式法则会得到下面的结果：

$$
\begin{align}
\frac{ \partial f }{ \partial {\color{orange} s }  } &= \frac{ \partial f }{ \partial {\color{blue} x_{1} }  } \frac{ \partial {\color{blue} x_{1} }  }{ \partial {\color{orange} s }  }  + \frac{ \partial f }{ \partial {\color{blue} x_{2} }  } \frac{ \partial {\color{blue} x_{2} }  }{ \partial {\color{orange} s }  } \tag{5.51}\\
\frac{ \partial f }{ \partial {\color{orange} t }  } &= \frac{ \partial f }{ \partial {\color{blue} x_{1} }  } \frac{ \partial {\color{blue} x_{1} }  }{ \partial {\color{orange} t }  }  + \frac{ \partial f }{ \partial {\color{blue} x_{2} }  } \frac{ \partial {\color{blue} x_{2} }  }{ \partial {\color{orange} t }  } \tag{5.52}
\end{align} 
$$
而函数的梯度为
$$
\frac{\mathrm{d}f}{\mathrm{d}(s,t)} = \frac{ \partial f }{ \partial \boldsymbol{ x }  } \frac{ \partial \boldsymbol{ x }  }{ \partial (s,t) } = \underbrace{ \begin{bmatrix}
\displaystyle \frac{ \partial f }{\color{blue} \partial x_{1} } &
\displaystyle \frac{ \partial f }{\color{orange} \partial x_{2} } 
\end{bmatrix} }_{ \displaystyle =\frac{ \partial f }{ \partial \boldsymbol{ x }  }  } \underbrace{ \begin{bmatrix}
\displaystyle {\color{blue} \frac{ \partial x_{1} }{ \partial s }  } & 
\displaystyle {\color{blue} \frac{ \partial x_{1} }{ \partial t }  } \\
\displaystyle {\color{orange} \frac{ \partial x_{2} }{ \partial s }  } & 
\displaystyle {\color{orange} \frac{ \partial x_{2} }{ \partial t }  } \\
\end{bmatrix} }_{ \displaystyle =\frac{ \partial \boldsymbol{x} }{ \partial (s,t) }  }.\tag{5.53}
$$
以上的写法 $(5.53)$ 当且仅当梯度被写为行向量时才是正确的，否则我们需要对结果进行转置，以保证矩阵的维度对应。在梯度为向量或矩阵时这样看来似乎比较显然，但当之后讨论中涉及的梯度变成 **张量（tensor）** 时对其进行转置就不那么容易了。

> **验证梯度是否正确**
> 将差商取极限而得到梯度的方法在计算机程序中的数值算法处被加以利用。当我们计算函数梯度时，我们可以通过数值的微小改变计算差商，然后校验梯度的正确性：取一个较小的值（例如 $h=10^{-4}$）然后计算有限差商和梯度的解析计算结果，如果误差足够小则说明梯度的解析结果大概率是正确的。误差足够小是指 $\displaystyle \sqrt{ \frac{\sum_{i} (dh_{i} - df_{i})^{2}}{\sum_{i} (dh_{i} + df_{i})^{2}} } < 10^{-6}$，其中 $dh_{i}$ 是指 $f$ 关于 $x_{i}$ 得到的有限差商的估计结果，$df_{i}$ 是指 $f$ 关于 $x_{i}$ 的解析梯度的计算结果。

## 5.3 向量值函数的梯度

一直以来我们讨论的都是实值函数 $f : \mathbb{R}^{n} \rightarrow \mathbb{R}$ 的偏导数和梯度，接下来我们将将此概念扩展至向量值函数（向量场）$\boldsymbol{f}: \mathbb{R}^{n} \rightarrow \mathbb{R}^{m}$ 的情形，其中 $n \geqslant 1, m \geqslant 1$。

给定向量值函数 $\boldsymbol{f}: \mathbb{R}^{n} \rightarrow \mathbb{R}^{m}$ 和向量 $\boldsymbol{x} = [x_{1}, \dots, x_{n}]^{\top}\in \mathbb{R}^{n}$，则该函数的函数值可以写为

$$
\boldsymbol{f}(\boldsymbol{x}) = \begin{bmatrix}
f_{1}(\boldsymbol{x})\\
\vdots\\
f_{m}(\boldsymbol{x})\\
\end{bmatrix} \in \mathbb{R}^{m}.\tag{5.54}
$$

这样写可以让我们将向量值函数 $\boldsymbol{f}: \mathbb{R}^{n} \rightarrow \mathbb{R}^{m}$ 看成一个全部由实值函数 $f_{i}: \mathbb{R}^{n} \rightarrow \mathbb{R}$  构成的向量 $[f_{1}, \dots, f_{m}]^{\top}$，而对于每一个 $f_{i}$ 我们可以不加修改的直接应用 5.2 节中的所有微分法则。这样一来，向量值函数对变元 $x_{i} \in \mathbb{R}, i=1, \dots, n$ 的偏导数由下式给出

$$\frac{ \partial \boldsymbol{f} }{ \partial x_{i} } = \begin{bmatrix}
\displaystyle \frac{ \partial f_{1} }{ \partial x_{i} } \\
\vdots\\
\displaystyle \frac{ \partial f_{m} }{ \partial x_{i} } 
\end{bmatrix} = \begin{bmatrix}
\displaystyle \lim_{ h \to 0 } \frac{f_{1}(x_{1}, \dots, x_{i-1}, x_{i}+h, x_{i+1}, x_{n}) - f_{1}(\boldsymbol{x})}{h}\\
\vdots\\
\displaystyle \lim_{ h \to 0 } \frac{f_{m}(x_{1}, \dots, x_{i-1}, x_{i}+h, x_{i+1}, x_{n}) - f_{m}(\boldsymbol{x})}{h}\\
\end{bmatrix} \in \mathbb{R}^{m}\tag{5.55}$$

从 $(5.40)$ 中我们了解到函数 $\boldsymbol{f}$ 对向量求导得到的是由一系列偏导数组合得到的行向量。在 $(5.55)$ 中，每个偏导数 $\displaystyle \frac{ \partial \boldsymbol{f}(\boldsymbol{x}) }{ \partial x_{i} }$ 自己就是一个列向量，于是我们可以将它们组合起来得到函数 $\boldsymbol{f}: \mathbb{R}^{n} \rightarrow \mathbb{R}^{m}$ 对向量 $\boldsymbol{x} \in \mathbb{R}^{n}$ 的梯度：

$$
\begin{align}
\frac{\mathrm{d}\boldsymbol{f}}{\mathrm{d}\boldsymbol{x}} &= 
\begin{bmatrix}
{\color{blue} \displaystyle \frac{ \partial \boldsymbol{f}(x) }{ \partial x_{1} }}  & \cdots & \color{orange} \displaystyle \frac{ \partial \boldsymbol{f}(x) }{ \partial x_{n} } 
\end{bmatrix} \tag{5.56a}\\[0.2em]
&= \begin{bmatrix}
\color{blue} \displaystyle \frac{ \partial f_{1}(\boldsymbol{x}) }{ \partial x_{1} } &\cdots &  \color{orange} \displaystyle \frac{ \partial f_{1}(\boldsymbol{x}) }{ \partial x_{n} } \\
\color{blue} \vdots & \ddots & \color{orange} \vdots\\
\color{blue} \displaystyle \frac{ \partial f_{m}(\boldsymbol{x}) }{ \partial x_{1} } & \cdots & \color{orange}  \displaystyle \frac{ \partial f_{m}(\boldsymbol{x}) }{ \partial x_{n} } \\
\end{bmatrix} \in \mathbb{R}^{m \times n} \tag{5.56b}
\end{align}
$$

> **定义 5.6 (Jacobian 矩阵)**
> 向量值函数 $\boldsymbol{f}: \mathbb{R}^{n} \rightarrow \mathbb{R}^{m}$ 的各一阶偏微分的合集称为 Jacobian 矩阵，它的形状是 $m \times n$ ，定义如下：$$\begin{align}\boldsymbol{J} &= \nabla_{x} \boldsymbol{f} = \frac{\mathrm{d}\boldsymbol{f}(\boldsymbol{x})}{\mathrm{d}\boldsymbol{x}} = \begin{bmatrix}\displaystyle \frac{ \partial \boldsymbol{f} }{ \partial x_{1} } & \cdots & \displaystyle \frac{ \partial \boldsymbol{f} }{ \partial x_{n} } \tag{5.57}\\\end{bmatrix}\\&= \begin{bmatrix}
 \displaystyle \frac{ \partial f_{1}(\boldsymbol{x}) }{ \partial x_{1} } &\cdots &  \displaystyle \frac{ \partial f_{1}(\boldsymbol{x}) }{ \partial x_{n} } \\\vdots & \ddots &  \vdots\\\displaystyle \frac{ \partial f_{m}(\boldsymbol{x}) }{ \partial x_{1} } & \cdots & \displaystyle \frac{ \partial f_{m}(\boldsymbol{x}) }{ \partial x_{n} } \\\end{bmatrix}, \tag{5.58}\\[0.2em]&\quad \boldsymbol{x} = \begin{bmatrix}x_{1}\\\vdots\\x_{n}\end{bmatrix}, \quad J(i,j) = \frac{ \partial f_{i} }{ \partial x_{j} }. \tag{5.59} \end{align}$$

作为 $(5.58)$ 的一个特例，标量值的向量变元函数 $f: \mathbb{R}^{n} \rightarrow \mathbb{R}^{1}$ （如 $\displaystyle f(\boldsymbol{x}) = \sum\limits_{i=1}^{n}x_{i}$）的 Jacobian 矩阵是一个行向量（形状为 $1 \times n$）；见 $(5.40)$。

> 注：本书中的微分使用 *分子布局（numerator layout）*。这是说函数 $\boldsymbol{f} \in \mathbb{R}^{m}$ 对 $\boldsymbol{x} \in \mathbb{R}^{n}$ 的微分 $\displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{x} }$ 得到矩阵的形状为 $m \times n$ ——如 $(4.58)$ —— 其中 $\boldsymbol{f}$ 决定这矩阵的行，$\boldsymbol{x}$ 决定矩阵的列。当然也有所谓的 *分母布局（denominator layout）*，得到的结果是分子布局的转置。

Jacobian 矩阵将在 6.7 节中概率分布的变量变换方法中起作用，而其中的缩放大小取决于其**行列式（determinant）**。

![300](attachments/Pasted%20image%2020240915233233.png)
<center>图 5.5</center>

在 4.1 节中，我们已使用行列式计算平行四边形的面积：如果给定正方形的两边所对应的两个向量$\boldsymbol{b}_{1} = [1, 0]^{\top}$ 和 $\boldsymbol{b}_{2} = [0, 1]^{\top}$，那么它们构成的正方形的面积是
$$
\begin{vmatrix}
\text{det}\left(\begin{bmatrix}
1 & 0\\0 & 1
\end{bmatrix}\right)
\end{vmatrix} = 1. \tag{5.60}
$$
如果我们取平行四边形的两边 $\boldsymbol{c}_{1} = [-2, 1]^{\top}$ 和 $\boldsymbol{c}_{2} = [1, 1]^{\top}$（如图 5.5 所示），其面积等于下面行列式的绝对值：
$$
\begin{vmatrix}
\text{det}\left( \begin{bmatrix}
-2 & 1\\1 & 1
\end{bmatrix} \right) 
\end{vmatrix} = |-3| = 3,
\tag{5.61}
$$
其刚好是单位正方形面积的三倍。我们可以通过测量单位正方形映射后得到的图形面积得到对应的缩放比例。如果使用线性代数的语言，我们做了一个从 $(\boldsymbol{b}_{1}, \boldsymbol{b}_{2})$ 到 $(\boldsymbol{c}_{1}, \boldsymbol{c}_{2})$ 的变量变换。在本例中，这个变换是线性的，变换本身的行列式就给出了缩放比例。

现在我们介绍两种确认这样的映射的方法。首先我们假设这个变换是线性的，这样就可以使用第二张中的内容确定它。随后我们将使用本章介绍的偏导数计算这个映射。

> **方法 1**
> 为了使用线性代数的工具，我们假定 $\{ \boldsymbol{b}_{1}, \boldsymbol{b}_{2} \}$ 和 $\{ \boldsymbol{c}_{1}, \boldsymbol{c}_{2} \}$ 是 $\mathbb{R}^{2}$ 的一个基（见 2.6.1 节），可见事实上我们做了一个从 $(\boldsymbol{b}_{1}, \boldsymbol{b}_{2})$ 到 $(\boldsymbol{c}_{1}, \boldsymbol{c}_{2})$ 的基变换，我们要找的变换矩阵就是执行这一基变换的矩阵。使用 2.7.2 节的结论，可以得到变换矩阵$$\boldsymbol{J} = \begin{bmatrix}-2 & 1\\ 1 & 1\end{bmatrix}, \tag{5.62}$$满足 $\boldsymbol{Jb}_{1} = \boldsymbol{c}_{1}$，$\boldsymbol{Jb}_{2} = \boldsymbol{c}_{2}$。该矩阵行列式的绝对值为 $|\det(\boldsymbol{J})| = 3$，这就是所求的缩放参数，也即基 $(\boldsymbol{c}_{1}, \boldsymbol{c}_{2})$ 张成的平行四边形的面积是基 $(\boldsymbol{b}_{1}, \boldsymbol{b}_{2})$ 张成的平行四边形的面积的三倍。

> **方法 2**
> 线性代数的方法可用于解线性函数的 Jacobi 矩阵，而对于非线性函数（在 6.7 节中涉及），我们使用一种更具一般性的方法 —— 使用偏导数。

考虑一个变量转换函数 $\boldsymbol{f} : \mathbb{R}^{2} \rightarrow \mathbb{R}^{2}$ ，它将基 $(\boldsymbol{b}_{1}, \boldsymbol{b}_{2})$ 下表示的向量 $\boldsymbol{x} \in \mathbb{R}^{2}$ 转换为基 $(\boldsymbol{c}_{1}, \boldsymbol{c}_{2})$ 表示下的向量 $\boldsymbol{y} \in \mathbb{R}^{2}$，我们通过计算映射 $\boldsymbol{f}$ 作用前后单位面积/体积的变化来确定这个映射。从这个角度出发，我们可以研究当我们稍稍改变 $\boldsymbol{x}$ 一点后 $\boldsymbol{f}(\boldsymbol{x})$ 的变化，而这恰好就是 $\displaystyle \frac{\mathrm{d}\boldsymbol{f}}{\mathrm{d}\boldsymbol{x}} \in \mathbb{R}^{2 \times 2}$。我们可以写出 $\boldsymbol{x}$ 和 $\boldsymbol{y}$ 的联系如下：$$\begin{align}y_{1} &= -2x_{1} + x_{2} \tag{5.63}\\y_{2} &= x_{1} + x_{2} \tag{5.64}\end{align}$$就可以容易的写出各项偏导数：$$\frac{ \partial y_{1} }{ \partial x_{1} }  = -2, \quad \frac{ \partial y_{1} }{ \partial x_{2} } = 1, \quad \frac{ \partial y_{2} }{ \partial x_{1} } =1, \quad \frac{ \partial y_{2} }{ \partial x_{2} } = 1\tag{5.65}$$
并得到表示坐标变换的 Jacobian 矩阵$$\boldsymbol{J} = \begin{bmatrix}\displaystyle \frac{ \partial y_{1} }{ \partial x_{1} } & \displaystyle \frac{ \partial y_{1} }{ \partial x_{2} } \\\displaystyle \frac{ \partial y_{2} }{ \partial x_{1} } & \displaystyle \frac{ \partial y_{2} }{ \partial x_{2} }\end{bmatrix} = \begin{bmatrix}-2 & 1 \\ 1 & 1\end{bmatrix}.\tag{5.66}$$
如果我们处理的是线性函数，则刚刚的 Jacobian 矩阵即为所求（注意 $(5.66)$ 和 $(5.62)$ 完全一样）；如若不然， Jacobian 矩阵是非线性映射的局部线性近似。Jacobian 矩阵的行列式 $|\boldsymbol{J}|$ 称为 Jabobian 行列式，它的值就是面积或体积变换前后的缩放比例，在这个例子中有 $|\boldsymbol{J}| = 3$。

上面提到的 Jacobian 行列式和变量替换在 6.7 节中对随机变量和分布进行变换时会涉及，它们在机器学习和深度学习中的 **重参数技巧（Reparametrization Trick）** 中十分重要，也被称为 **无穷摄动分析（Infinite Perturbation Analysis）**。


![](attachments/Pasted%20image%2020250106153605.png)
<center>图5.6 （偏）导数的维度和形状</center>

在本章讨论了函数的导数，图5.6给出它们的形状。如果$f: \mathbb{R} \rightarrow \mathbb{R}$，其梯度只是一个标量（左上角）。如果$f: \mathbb{R}^{D} \rightarrow \mathbb{R}$，它的梯度是一个形状为 $1 \times D$ 的行向量（右上角）。如果$\boldsymbol{f}: \mathbb{R} \rightarrow \mathbb{R}^{E}$，它的梯度是一个形状为 $E×1$ 的列向量，而如果 $\boldsymbol{f}: \mathbb{R}^{D} \rightarrow \mathbb{R}^{E}$，梯度则是一个形状为 $E×D$ 的矩阵。

> **示例 5.9（向量值函数的梯度）**
> 给定$$\boldsymbol{f}(\boldsymbol{x}) = \boldsymbol{Ax}, \quad \boldsymbol{f}(\boldsymbol{x}) \in \mathbb{R}^{M}, \quad \boldsymbol{A} \in \mathbb{R}^{N}.$$
> 为了计算梯度$\displaystyle \displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{x} }$，我们首先确定$\displaystyle \displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{x} }$的维数：由于$\boldsymbol{f}: \mathbb{R}^{N} \rightarrow \mathbb{R}^{M}$，所以有 $\displaystyle \displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{x} } \in \mathbb{R}^{M \times N}$。为了计算梯度，我们接下来计算 $\boldsymbol{f}$ 相对于每个变元 $x_j$ 的偏导数$$f_{i}(\boldsymbol{x}) = \sum\limits_{j=1}^{N} A_{i,j}x_{j} \implies \frac{ \partial f_{i} }{ \partial x_{j} } = A_{i,j} \tag{5.67}$$
> 知道了这些偏导数，我们就得到了梯度$$\frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{x} }  = \begin{bmatrix}\displaystyle \frac{ \partial f_{1} }{ \partial x_{1} } & \cdots & \displaystyle \frac{ \partial f_{1} }{ \partial x_{N} } \\\vdots & \ddots & \vdots\\\displaystyle \frac{ \partial f_{M} }{ \partial x_{1} } & \cdots &\displaystyle \frac{ \partial f_{M} }{ \partial x_{N } } \end{bmatrix} = \begin{bmatrix}A_{1,1} & \cdots & A_{1,N}\\\vdots & \ddots & \vdots\\A_{M,1} & \cdots & A_{M,N}\end{bmatrix} = \boldsymbol{A} \in \mathbb{R}^{M \times N}.\tag{5.68}$$

> **示例 5.10（链式法则）**
> 考虑函数 $h: \mathbb{R} \rightarrow \mathbb{R}$，$h(t) = (f \circ g)(t)$，其中$$
\begin{align}
f &: \mathbb{R}^{2} \rightarrow \mathbb{R} \tag{5.69}\\
g &: \mathbb{R} \rightarrow \mathbb{R}^{2} \tag{5.70}\\
f(\boldsymbol{x}) &= \exp (x_{1}, x_{2}^{2}), \tag{5.71}\\
\boldsymbol{x} &= \begin{bmatrix}
x_{1} \\ x_{2}
\end{bmatrix} = g(t) = \begin{bmatrix}
t\cos t\\t\sin t
\end{bmatrix} \tag{5.72}
\end{align}
$$计算 $h$ 关于 $t$ 的梯度。
> 因为 $f:\mathbb{R}^{2}→\mathbb{R}$ 和 $g:\mathbb{R}\rightarrow \mathbb{R}^{2}$，于是我们有$$\displaystyle \frac{ \partial f }{ \partial \boldsymbol{x} } \in \mathbb{R}^{1 \times 2}, \quad \displaystyle \frac{ \partial g }{ \partial t } \in \mathbb{R}^{2 \times 1}. \tag{5.73}$$
> 复合函数的梯度可通过链式法则求得：$$\begin{align}\displaystyle \frac{ \mathrm{d}h }{ \mathrm{d}t } &= {\color{blue} \displaystyle \frac{ \partial f }{ \partial \boldsymbol{x} } } {\color{orange} \displaystyle \frac{ \partial \boldsymbol{x} }{ \partial t }  } = {\color{blue} \begin{bmatrix}\displaystyle \frac{ \partial f }{ \partial x_{1} } & \displaystyle \frac{ \partial f }{ \partial x_{2} } \end{bmatrix} } {\color{orange} \begin{bmatrix}\displaystyle \frac{ \partial x_{1} }{ \partial t } \\ \displaystyle \frac{ \partial x_{2} }{ \partial t }\end{bmatrix} }   \tag{5.74a}\\&= {\color{blue} \begin{bmatrix} \exp(x_{1}x_{2}^{2})x_{2}^{2} & 2\exp(x_{1}x_{2}^{2})x_{1}x_{2} \end{bmatrix}}{\color{orange} \begin{bmatrix}\cos t - t\sin t\\sin t + t\cos t\end{bmatrix}} \tag{5.74b}\\&= \exp(x_{1}x_{2}^{2}) \big[ x_{2}^{2}(\cos t - t\sin t) + 2x_{1}x_{2}(\sin t + t\cos t) \big] , \tag{5.74c}\end{align}$$
> 其中，$x_{1} = t\cos t$，$x_{2} = t\sin t$；见（5.72）。



> **示例 5.11（线性模型中最小二乘损失之梯度）**
> 考虑下面的线性模型$$\boldsymbol{y} = \boldsymbol{\Phi \theta },\tag{5.75}$$其中 $\boldsymbol{\theta} \in \mathbb{R}^{D}$ 是参数向量，$\boldsymbol{\Phi}\in \mathbb{R}^{N \times D}$ 是输入特征，$\boldsymbol{y} \in \mathbb{R}^{N}$ 是对应的观测值。为方便叙述，我们定义$$\begin{align}L(\boldsymbol{e}) &:= \|\boldsymbol{e}\|^{2}, \tag{5.76}\\\boldsymbol{e}(\boldsymbol{\theta}) &:= \boldsymbol{y} - \boldsymbol{\Phi \theta}. \tag{5.77}\end{align}$$我们使用链式法则计算偏导数 $\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta} }$，其中$L$称为*最小二乘损失函数 (Least-squares loss function)*。开始计算之前，我们首先确定梯度的维数： $$\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta} } \in \mathbb{R}^{L \times D}. \tag{5.78}$$然后使用链式法则：$$\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta} } = {\color{blue} \displaystyle \frac{ \partial L }{ \partial \boldsymbol{e} }  } {\color{orange} \displaystyle \frac{ \partial \boldsymbol{e} }{ \partial \boldsymbol{\theta} }  } , \tag{5.79}$$其中等式左边向量从左往右的第 $d$ 个元素是由$$\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta} } [1,d] = \sum\limits_{n=1}^{N} \displaystyle \frac{ \partial L }{ \partial \boldsymbol{e} } [n] \displaystyle \frac{ \partial \boldsymbol{e} }{ \partial \boldsymbol{\theta} } [n,d] \tag{5.80}$$给出的。我们知道 $\|\boldsymbol{e}\|^{2} = \boldsymbol{e}^{\top}\boldsymbol{e}$（见 3.2 节）因此有$${\color{blue} \displaystyle \frac{ \partial L }{ \partial \boldsymbol{e} }  = 2\boldsymbol{e}^{\top} } \in \mathbb{R}^{1 \times N}. \tag{5.81}$$进一步，我们有$${\color{orange} \displaystyle \frac{ \partial \boldsymbol{e} }{ \partial \boldsymbol{\theta} } = -\boldsymbol{\Phi} } \in \mathbb{R}^{N \times D} , \tag{5.82}$$结合起来即为所求：$$\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta} } = {\color{orange} - } {\color{blue} 2\boldsymbol{e}^{\top} }{\color{orange} \boldsymbol{\Phi} } {~}\mathop{=\!=\!=}\limits^{(5.77)}{~} {\color{orange} - } {\color{blue} \underbrace{ 2(\boldsymbol{y}^{\top} - \boldsymbol{\theta}^{\top}\boldsymbol{\Phi}^{\top}) }_{ 1 \times N } }~{\color{orange} \underbrace{ \boldsymbol{\Phi} }_{ N \times D } } \in \mathbb{R}^{1 \times D}. \tag{5.83}$$
> 注：如果不使用链式法则，我们就可以通过展开下面的函数进行求导以得到相同的结果$$L_{2}(\boldsymbol{\theta}) := \|\boldsymbol{y} - \boldsymbol{\Phi \theta}\|^{2} = (\boldsymbol{y} - \boldsymbol{\Phi \theta})^{\top}(\boldsymbol{y} - \boldsymbol{\Phi \theta}). \tag{5.84}$$然而这种方法虽然可用于这样的简单函数，但当我们面对深度复合的复杂函数时这样做就不现实了。

## 5.4 矩阵的梯度

接下来我们将会看见需要求矩阵对向量（或其他矩阵）的梯度的情形。它们的结果是一个多维度的*张量（tensor）*，我们可以将其看做装有偏导数的多维数组。例如，如果我们计算一个 $m \times n$ 形状的矩阵 $\boldsymbol{A}$ 对 $p \times q$ 形状的矩阵 $\boldsymbol{B}$ 的梯度，结果雅可比矩阵的形状将是 $(m \times n) \times (p \times q)$，即一个四维张量 $\boldsymbol{J}$，它的每个分量可以写为 $\boldsymbol{J}_{i,j,k,l} = \displaystyle \frac{ \partial \boldsymbol{A}_{i,j} }{ \partial \boldsymbol{B}_{k, l} }$。

由于矩阵代表着线性变换。我们可以用这样的事实构造形状为 $m \times n$ 矩阵空间 $\mathbb{R}^{m \times n}$ 到 $mn$ 长度的向量空间 $\mathbb{R}^{mn}$ 的向量空间同构（可逆的线性映射）。这样一来我们就可以调整矩阵 $\boldsymbol{A}$ 和 $\boldsymbol{B}$ 的形状，使其分别变成长度为 $mn$ 和长度为 $pq$ 的向量。因此对这样的向量求梯度就得到形状为 $mn \times pq$ 的Jacobi矩阵。图 5.7 画出了上面两种方法的示意图。实际操作中，将矩阵压扁成向量然后继续处理Jacobi矩阵的方法较受欢迎，因为这样一来链式法则（5.48）就变成简单的矩阵乘法；而如果处理的是Jacobi张量，我们就得对于二者相乘时求和的维度倍加小心。

![](attachments/Pasted%20image%2020250106171552.png)

> **示例 5.12（向量对矩阵的梯度）**
> 考虑下面的例子：$$\boldsymbol{f} = \boldsymbol{A}\boldsymbol{x}, \quad \boldsymbol{f} \in \mathbb{R}^{M}, \quad \boldsymbol{A} \in \mathbb{R}^{M \times N}, \quad \boldsymbol{x} \in \mathbb{R}^{N},\tag{5.85}$$求梯度 $\displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{A} }$。
> 首先确定梯度的维数：$$\displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{A} } \in \mathbb{R}^{M \times (M \times N)}. \tag{5.86}$$按照定义，梯度里面装着一族偏导的结果：$$\displaystyle \frac{ \mathrm{d}\boldsymbol{f} }{ \mathrm{d}\boldsymbol{A} } = \begin{bmatrix}\displaystyle \frac{ \partial f_{i} }{ \partial \boldsymbol{A} } \\ \vdots  \\ \displaystyle \frac{ \partial f_{M} }{ \partial \boldsymbol{A} } \end{bmatrix}, \quad \displaystyle \frac{ \partial f_{i} }{ \partial \boldsymbol{A} } \in \mathbb{R}^{1 \times (M \times N)}. \tag{5.87}$$接下来我们求每一项的值。我们首先根据矩阵乘法分别展开每个结果分量 $f_{i}$：$$f_{i} = \sum\limits_{j=1}^{N} A_{i,j}x_{j}, \quad  i= 1, \dots, M, \tag{5.88}$$然后得到 $f_{i}$ 对矩阵中每一份量的偏导数$$\displaystyle \frac{ \partial f_{i} }{ \partial A_{j,q} } = x_{q}. \tag{5.89}$$将它们一行一行的组合起来，并注意一下结果的形状，我们就得到了 $f_{i}$ 对矩阵 $\boldsymbol{A}$ 中各行的偏导：$$\begin{align}\displaystyle \frac{ \partial f_{i} }{ \partial A_{i,:}} &= \boldsymbol{x}^{\top} \in \mathbb{R}^{1 \times 1 \times N}, \tag{5.90}\\\displaystyle \frac{ \partial f_{i} }{ \partial A_{k\neq i, :} } &= \boldsymbol{0}^{\top} \in \mathbb{R}^{1 \times 1 \times N}, \tag{5.91}\end{align}$$由于 $f_{i}$ 是实值函数，矩阵 $\boldsymbol{A}$ 的每一行形状为 $1×N$，我们得到的 $f_{i}$ 关于矩阵每一行的偏导数张量的形状就是 $1×1×N$。最后我们将（5.91）堆叠起来，就得到所求的梯度（5.87）中的每一项：$$\displaystyle \frac{ \partial f_{i} }{ \partial \boldsymbol{A} } = \begin{bmatrix}\boldsymbol{0}^{\top} \\ \vdots \\ \boldsymbol{0}^{\top} \\ \boldsymbol{x}^{\top} \\ \boldsymbol{0}^{\top} \\ \vdots \\\boldsymbol{0}^{\top}\end{bmatrix} \in \mathbb{R}^{1 \times (M \times N)}. \tag{5.92}$$

> **示例 5.13（矩阵对矩阵的梯度）**
> 给定矩阵 $\boldsymbol{R} \in \mathbb{R}^{M \times N}$，和矩阵值函数 $\boldsymbol{f}: \mathbb{R}^{M \times N} \rightarrow \mathbb{R}^{N \times N}$:$$\boldsymbol{f}(\boldsymbol{R}) = \boldsymbol{R}^{\top}\boldsymbol{R} =: \boldsymbol{K} \in \mathbb{R}^{N \times N}, \tag{5.93}$$求梯度 $\displaystyle \frac{ \mathrm{d}\boldsymbol{K} }{ \mathrm{d}\boldsymbol{R} }$。
> 这个问题有些困难。我们先写下已知信息：梯度的维数$$\displaystyle \frac{ \mathrm{d}\boldsymbol{K} }{ \mathrm{d}\boldsymbol{R} } \in \mathbb{R}^{(N \times N) \times (M \times N)}, \tag{5.94} $$毫无疑问这是个张量。我们进一步写出 $\boldsymbol{K}$ 中每个元素对矩阵 $\boldsymbol{R}$ 的梯度维数：$$\displaystyle \frac{ \mathrm{d}K_{p,q} }{ \mathrm{d}\boldsymbol{R} } \in \mathbb{R}^{1 \times M \times N}, p,q = 1, \dots, N \tag{5.95}$$其中 $\boldsymbol{K}_{p,q}$ 是 $\boldsymbol{K} = \boldsymbol{f} (\boldsymbol{R})$ 中处于第 $p$ 行，第 $q$ 列的元素。用 $\boldsymbol{r}_{i}$ 表示 $\boldsymbol{R}$ 的第 $i$ 列，则 $\boldsymbol{K}$ 中的每个元素可以写成 $\boldsymbol{R}$ 中两列的点积，即$$K_{p,q} = \boldsymbol{r}_{p}^{\top}\boldsymbol{r}_{q} =\sum\limits_{m=1}^{M} R_{m,p}R_{m,q}. \tag{5.96}$$接着我们计算偏导数$$\displaystyle \frac{ \partial K_{p,q} }{ \partial R_{i,j} } = \sum\limits_{m=1}^{M} \displaystyle \frac{ \partial   }{ \partial R_{i,j} } R_{m,p}R_{m,q} = \partial_{p,q,i,j},\tag{5.97}$$其中$$\partial_{p,q,i,j} = \begin{cases}R_{i, q}, & j = p, p \neq q\\R_{i, p}, & j = q, p \neq q\\2R_{i,q}, & j=p, p=q\\0, & \text{其他情形}\end{cases}\quad .\tag{5.98}$$从（9.94）我们知道目标梯度的形状是 $(N \times N) \times (M \times N)$，它的每个分量的值由（5.98）给出，其中 $p,q,j = 1, \dots, N$，$i = 1, \dots, M$。


## 5.5 常用梯度恒等式

下面，我们列出了一些在机器学习环境中常用的梯度恒等式（Petersen and Pedersen，2012）。其中 $\text{tr}(\cdot)$ 代表矩阵的迹（见定义4.4），$\text{det}(\cdot)$ 表示矩阵的行列式（见 4.1 节），$\boldsymbol{f}(\boldsymbol{X})^{-1}$ 表示 $\boldsymbol{f}(\boldsymbol{X})$ 的逆（假设其存在）。
$$
\begin{align}
\frac{ \partial }{ \partial \boldsymbol{X} } \boldsymbol{f}(\boldsymbol{X})^{\top} &= \left( \frac{ \partial \boldsymbol{f}(\boldsymbol{X}) }{ \partial \boldsymbol{X} }  \right)^{\top}, \tag{5.99}\\[0.2em]
\frac{ \partial  }{ \partial \boldsymbol{X} } \text{tr}\big[ \boldsymbol{f}(\boldsymbol{X}) \big] &= \text{tr}\left[ \frac{ \partial \boldsymbol{f}(\boldsymbol{X}) }{ \partial \boldsymbol{X} }  \right], \tag{5.100}\\[0.2em]
\frac{ \partial  }{ \partial \boldsymbol{X} } \det \big[ \boldsymbol{f}(\boldsymbol{X}) \big] &= \det \big[ \boldsymbol{f}(\boldsymbol{X}) \big] \text{tr}\left[ \boldsymbol{f}(\boldsymbol{X})^{-1} \frac{ \partial \boldsymbol{f}(\boldsymbol{X}) }{ \partial \boldsymbol{X} }  \right], \tag{5.101}\\[0.2em]
\frac{ \partial  }{ \partial \boldsymbol{X} } \boldsymbol{f}(\boldsymbol{X})^{-1} &= -\boldsymbol{f}(\boldsymbol{X})^{-1} \left[ \frac{ \partial \boldsymbol{f}(\boldsymbol{X}) }{ \partial \boldsymbol{X} }  \right]\boldsymbol{f}(\boldsymbol{X})^{-1} \tag{5.102}\\[0.2em]
\frac{ \partial \boldsymbol{a}^{\top}\boldsymbol{X}^{-1}\boldsymbol{b} }{ \partial \boldsymbol{X} } &= -(\boldsymbol{X}^{-1})^{\top}\boldsymbol{a}\boldsymbol{b}^{\top}(\boldsymbol{X}^{-1})^{\top}\tag{5.103}\\[0.2em]
\frac{ \partial \boldsymbol{x}^{\top}\boldsymbol{a} }{ \partial \boldsymbol{x} } &= \boldsymbol{a}^{\top}\tag{5.104}\\[0.2em]
\frac{ \partial \boldsymbol{a}^{\top}\boldsymbol{x} }{ \partial \boldsymbol{x} } &= \boldsymbol{a}^{\top}\tag{5.105}\\[0.2em]
\frac{ \partial \boldsymbol{a}^{\top}\boldsymbol{X}\boldsymbol{b} }{ \partial \boldsymbol{X} } &= \boldsymbol{a}\boldsymbol{b}^{\top} \tag{5.106}\\[0.2em]
\frac{ \partial \boldsymbol{x}^{\top}\boldsymbol{B}\boldsymbol{x} }{ \partial \boldsymbol{x} } &= \boldsymbol{x}^{\top}(\boldsymbol{B} + \boldsymbol{B}^{\top})\tag{5.107}\\[0.2em]
\frac{ \partial  }{ \partial \boldsymbol{s} } (\boldsymbol{x} - \boldsymbol{A}\boldsymbol{s})^{\top}\boldsymbol{W}(\boldsymbol{x} - \boldsymbol{A}\boldsymbol{s}) &= -2(\boldsymbol{x} - \boldsymbol{A}\boldsymbol{s})^{\top}\boldsymbol{W}\boldsymbol{A}, \tag{5.108}\\[0.2em]
& \quad \quad \text{for symmetric }\boldsymbol{W}
\end{align}
$$
注：本书中我们仅讨论矩阵的迹和转置。然而，我们已看到求导的结果可能是高维张量，通常的迹和转置在其范畴中没有定义。在这些情况下，形状为 $D×D×E×F$ 的张量的迹将是一个 $E×F$ 形状的矩阵。这是 *张量缩并（tensor contraction）* 的一种特殊情况。类似地，当我们“转置”一个张量时，我们说的是交换前两个维度。具体而言，在（5.99）到（5.102）中，当我们计算多元函数 $\boldsymbol{f}(\cdot)$ 对矩阵的导数时，我们需要张量相关的计算，而像在 5.4 节中那样将其拉直成向量。

【以下为机器翻译结果，需要进行后期修正】

## 5.6 反向传播与自动微分

在许多机器学习的应用中，我们通过计算学习目标关于模型参数的梯度，然后执行梯度下降（见 7.1 节）找好的模型参数。对于给定的目标函数，我们可以利用微积分的链式法则得到其对模型参数的梯度（见 5.2.2 节）。我们在 5.3 节已经尝试对平方损失结果关于线性回归模型参数求梯度。

考虑下面的函数：
$$
f(x) = \sqrt{ x^{2} + \exp(x^{2}) } + \cos \Big[ x^{2} + \exp(x^{2}) \Big]. \tag{5.109}
$$
由链式法则，并注意到微分的线性性，我们可以得到：
$$
\begin{align}
\displaystyle \frac{ \mathrm{d}f }{ \mathrm{d}x } &= \frac{2x+2x\exp\{x^{2}\}}{2\sqrt{ x^{2} + \exp\{x^{2}\} }} - \sin \Big( x^{2} + \exp\{ x^{2} \} \Big) \Big( 2x + 2x \exp\{ x^{2} \} \Big) \\
&= 2x \left[ \frac{1}{2\sqrt{ x^{2} + \exp\{ x^{2} \} }} - \sin \Big( x^{2} + \exp\{ x^{2} \} \Big) \right] \Big(1 + \exp\{ x^{2} \}\Big). 
\end{align}
$$

$$
\tag{5.110}
$$
像这样显式求解得到这样冗长的导数表达往往不切实际。在实践中这意味着若不小心处理，梯度的实现可能比计算函数值要昂贵得多，这增加了不必要的开销。对于神经网络模型，反向传播算法(Kelley, 1960; Bryson, 1961; Dreyfus, 1962; Rumelhart et al., 1986)是一种计算误差对模型参数梯度的有效方法。

### 5.6.1 深度神经网络中的梯度

深度学习领域将链式法则的功用发挥到了极致，输入 $\boldsymbol{x}$ 通过多层复合的函数得到函数值 $\boldsymbol{y}$ ：
$$
\boldsymbol{y} = (f_{K} \circ f_{K-1} \circ \cdots \circ f_{1})(\boldsymbol{x}) = f_{K}\Big\{ f_{K-1}\big[\cdots (f_{1}(\boldsymbol{x})\cdots )\big] \Big\} , \tag{5.111}
$$
其中，$\boldsymbol{x}$ 是输入（如图像），$\boldsymbol{y}$ 是观测值（如类标签），每个函数 $f_{i}, i = 1, \dots, K$，有各自的参数。

![800](attachments/Pasted%20image%2020250131211557.png)
<center>图 5.8 多层神经网络的前向传播</center>

在一般的多层神经网络中，第 $i$ 层中有函数  $f_{i}(\boldsymbol{x}_{i-1}) = \sigma(\boldsymbol{A}_{i-1}\boldsymbol{x}_{i-1} + \boldsymbol{b}_{i-1})$ 。其中 $x_{i-1}$ 是 $i=1$ 层的输出和一个激活函数 $\sigma$，例如 sigmoid 函数 $\displaystyle \frac{1}{1-e^{-x}}$，$\tanh$ 或修正线性单元（rectified linear unit, ReLU）。为了训练这样的模型，我们需要一个损失函数 $L$，对其值求关于所有模型参数 $\boldsymbol{A}_j, \boldsymbol{b}_{j}, j=1, \dots, K$ 的梯度。这同时要求我们求其对模型中各层的输入的梯度。例如，如果我们有输入x和观测值y和一个网络结构，则它被定义为
$$
\begin{align}
\boldsymbol{f}_{0} &:= \boldsymbol{x} \tag{5.112}\\
\boldsymbol{f}_{i} &:= \sigma_{i} \Big( \boldsymbol{A}_{i-1}\boldsymbol{f}_{i-1} + \boldsymbol{b}_{i-1} \Big), \quad  i=1, \dots, K, \tag{5.113} 
\end{align}
$$
参见图5.8的可视化，我们可能有兴趣找到Aj，bj为j = 0，……，K−1，这样的平方损失
$$
L(\boldsymbol{\theta}) = \Big\| \boldsymbol{y} - \boldsymbol{f}_{K}\big( \boldsymbol{\theta, x} \big)  \Big\|^{2} \tag{5.114}
$$
被最小化，其中θ = {A0，b0，……，AK−1，bK−1}。为了得到参数相对于参数集θ的梯度，我们需要L对每一层j=的参数θj = {Aj，bj}的偏导数0，…，K−1。链式规则允许我们确定偏导数为
$$
\begin{align}
\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta}_{K-1} } &= \displaystyle \frac{ \partial L }{ \partial \boldsymbol{f}_{K} } {\color{blue} \displaystyle \frac{ \partial \boldsymbol{f}_{K} }{ \partial \boldsymbol{\theta}_{K-1} } } \tag{5.115}\\
\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta}_{K-2} }  &= \displaystyle \frac{ \partial L }{ \partial \boldsymbol{f}_{K} } \boxed{ {\color{orange} \displaystyle \frac{ \partial \boldsymbol{f}_{K} }{ \partial \boldsymbol{f}_{K-1} }  } {\color{blue} \displaystyle \frac{ \partial \boldsymbol{f}_{K-1} }{ \partial \boldsymbol{\theta}_{K-2} }  }}\tag{5.116}\\  
\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta}_{K-3} } &= \displaystyle \frac{ \partial L }{ \partial \boldsymbol{f}_{K} } {\color{orange} \displaystyle \frac{ \partial \boldsymbol{f}_{K} }{ \partial \boldsymbol{f}_{K-1} }  } \boxed{ {\color{orange} \displaystyle \frac{ \partial \boldsymbol{f}_{K-1} }{ \partial \boldsymbol{f}_{K-2} }  } {\color{blue} \displaystyle \frac{ \partial \boldsymbol{f}_{K-2} }{ \partial \boldsymbol{\theta}_{K-3} }  } } \tag{5.117}\\
\displaystyle \frac{ \partial L }{ \partial \boldsymbol{\theta}_{i} }  &= \displaystyle \frac{ \partial L }{ \partial \boldsymbol{f}_{K} } {\color{orange} \displaystyle \frac{ \partial \boldsymbol{f}_{K} }{ \partial \boldsymbol{f}_{K-1} } \cdots } \boxed{ {\color{orange} \displaystyle \frac{ \partial \boldsymbol{f}_{i+2} }{ \partial \boldsymbol{f}_{i+1} }  } {\color{blue} \displaystyle \frac{ \partial \boldsymbol{f}_{i+1} }{ \partial \boldsymbol{\theta}_{i} }  }  } \tag{5.118}
\end{align}
$$
橙色的项是一个层的输出相对于其输入的偏导数，而蓝色的项是一个层的输出相对于其参数的偏导数。假设，我们已经计算出了偏导数∂L/∂θi+1，那么大部分的计算都可以被重用来计算∂L/∂θi。我们所提供的附加条款需要计算的是用方框表示。图5.9显示了梯度通过网络向后传递。

### 5.6.2 自动微分
结果表明，反向传播是数值分析中一般采用的自动微分技术的一种特殊情况。我们可以把自动差分看作是一组技术，通过处理中间变量和应用链规则，用数值（与符号化相反）来评估一个函数的精确（直到机器精度）梯度。自动微分应用一系列初等算术运算，如加法、乘法和初等函数，如sin、cos、exp、log。通过将链式规则应用于这些操作，可以自动计算出相当复杂的函数的梯度。自动微分适用于一般的计算机程序，并具有正向和反向模式。Baydin等人（2018）对机器学习中的自动分化进行了很好的概述。图5.10显示了一个简单的图，通过一些中间变量a，b表示从输入x到输出y的数据流。如果我们要计算导数dy/dx，我们将应用链规则并得到
$$
\displaystyle \frac{ \mathrm{d}y }{ \mathrm{d}x }  = \displaystyle \frac{ \mathrm{d}y }{ \mathrm{d}b } \displaystyle \frac{ \mathrm{d}b }{ \mathrm{d}a } \displaystyle \frac{ \mathrm{d}a }{ \mathrm{d}x } . \tag{5.119}
$$
直观地说，正模和反模在多重的顺序上不同-在一般情况下，我们使用雅可比矩阵，它可以是向量、矩阵或张量。阳离子。由于矩阵乘法的结合性，我们可以从中进行选择
$$
\begin{align}
\displaystyle \frac{ \mathrm{d}y }{ \mathrm{d}x } &= \left( \displaystyle \frac{ \mathrm{d}y }{ \mathrm{d}b } \displaystyle \frac{ \mathrm{d}b }{ \mathrm{d}a }  \right) \displaystyle \frac{ \mathrm{d}a }{ \mathrm{d}x } , \tag{5.120}\\
\displaystyle \frac{ \mathrm{d}y }{ \mathrm{d}x } &= \displaystyle \frac{ \mathrm{d}y }{ \mathrm{d}b } \left( \displaystyle \frac{ \mathrm{d}b }{ \mathrm{d}a } \displaystyle \frac{ \mathrm{d}a }{ \mathrm{d}x }  \right). \tag{5.121}
\end{align}
$$
方程（5.120）将是反向模式，因为梯度通过图向后传播，即反向传播到数据流。公式（5.121）是正向模式，其中梯度与数据从左到右流动。

下面，我们将重点关注反向模式的自动微分，即反向传播。在神经网络中，输入的维数通常比标签的维数高得多，反向模式在计算上比正向模式低得多。让我们从一个有益的例子开始
$$
f(x) = \sqrt{ x^{2} + \exp\{ x^{2} \} } + \cos \Big( x^{2} + \exp\{ x^{2} \} \Big) \tag{5.122}
$$
从（5.109）开始。如果我们要在计算机上实现一个函数f，我们将能够通过使用中间变量来节省一些计算：
$$
\begin{align}
a &= x^{2}, \tag{5.123}\\
b &= \exp\{ a \}, \tag{5.124}\\
c &= a + b, \tag{5.125}\\
d &= \sqrt{ c }, \tag{5.126}\\
e &= \cos(c), \tag{5.127}\\
f &= d + e. \tag{5.128}\\
\end{align}
$$
这是在应用链规则时发生的相同的思维过程。请注意，前面一组方程所需要的操作比（5.109）中定义的函数f (x)的直接实现要少。图5.11中对应的计算图显示了获得函数值f所需的数据流和计算量。包含中间变量的方程组可以被认为是一个计算图，这是一种广泛应用于神经网络软件库实现中的表示方法。通过回顾初等函数的导数的定义，我们可以直接计算中间变量与其相应输入的导数。我们获得以下信息：
$$
\begin{align}
\displaystyle \frac{ \partial a }{ \partial x } &= 2x \tag{5.129}\\
\displaystyle \frac{ \partial b }{ \partial a } &= \exp\{ a \}\tag{5.130}\\
\displaystyle \frac{ \partial c }{ \partial a } &= 1 = \displaystyle \frac{ \partial c }{ \partial b } \tag{5.131}\\
\displaystyle \frac{ \partial d }{ \partial c } &= \frac{1}{2\sqrt{ c }}\tag{5.132}\\
\displaystyle \frac{ \partial e }{ \partial c } &= -\sin(c)\tag{5.133}\\
\displaystyle \frac{ \partial f }{ \partial d } &= 1 = \displaystyle \frac{ \partial f }{ \partial e }. \tag{5.134}
\end{align}
$$
通过查看图5.11中的计算图，我们可以通过从输出中向后工作来计算∂f/∂x并得到
$$
\begin{align}
\displaystyle \frac{ \partial f }{ \partial c } &= \displaystyle \frac{ \partial f }{ \partial d } \displaystyle \frac{ \partial d }{ \partial c }  + \displaystyle \frac{ \partial f }{ \partial e } \displaystyle \frac{ \partial e }{ \partial c } \tag{5.135}\\
\displaystyle \frac{ \partial f }{ \partial b } &= \displaystyle \frac{ \partial f }{ \partial c } \displaystyle \frac{ \partial c }{ \partial b } \tag{5.136}\\
\displaystyle \frac{ \partial f }{ \partial a } &= \displaystyle \frac{ \partial f }{ \partial b } \displaystyle \frac{ \partial b }{ \partial a } + \displaystyle \frac{ \partial f }{ \partial c } \displaystyle \frac{ \partial c }{ \partial a } \tag{5.137}\\
\displaystyle \frac{ \partial f }{ \partial x } &= \displaystyle \frac{ \partial f }{ \partial a } \displaystyle \frac{ \partial a }{ \partial x }. \tag{5.138}\\
\end{align}
$$
注意，我们隐式地应用了链规则来获得∂f/∂x。用初等函数导数的结果，得到
$$
\begin{align}
\displaystyle \frac{ \partial f }{ \partial c } &= 1 \cdot \frac{1}{2\sqrt{ c }} + 1 \cdot \big[ -\sin(c) \big] \tag{5.139}\\
\displaystyle \frac{ \partial f }{ \partial b } &= \displaystyle \frac{ \partial f }{ \partial c } \cdot 1\tag{5.140}\\
\displaystyle \frac{ \partial f }{ \partial a } &= \displaystyle \frac{ \partial f }{ \partial b } \exp\{ a \} + \displaystyle \frac{ \partial f }{ \partial c } \cdot 1 \tag{5.141}\\
\displaystyle \frac{ \partial f }{ \partial x } &= \displaystyle \frac{ \partial f }{ \partial a }  \cdot 2x. \tag{5.142}
\end{align}
$$
通过将上面的每个导数作为一个变量，我们观察到计算导数所需的计算与函数本身的计算的复杂性相似。这是非常违反直觉的，因为导数∂f∂x（5.110）的数学表达式比（5.109）中的函数f (x)的数学表达式要复杂得多。

自动区分是实施例5.14的形式化。设x1，……，xd是函数的输入变量，xd+1，……，xd−1是中间变量，xD是输出变量。则计算图可以表示为：
$$
\text{For }i = d+1, \dots, D:\quad x_{i} = g_{i}\Big[x_{\text{Pa}(x_{i})}\Big], \tag{5.143}
$$
其中，gi（·）是基本函数，xPa（xi）是图中变量xi的父节点。给定一个以这种方式定义的函数，我们可以使用链式规则来逐步计算该函数的导数。回想一下，根据定义，f = xD，因此
$$
\displaystyle \frac{ \partial f }{ \partial x_{D} } =1. \tag{5.144}
$$
对于其他变量xi，我们应用链式规则
$$
\displaystyle \frac{ \partial f }{ \partial x_{i} } = \sum\limits_{x_{j}: x_{i} \in \text{Pa}(x_{j})} \displaystyle \frac{ \partial f }{ \partial x_{j} } \displaystyle \frac{ \partial x_{j} }{ \partial x_{i} } = \sum\limits_{x_{j}: x_{i} \in \text{Pa}(x_{j})} \displaystyle \frac{ \partial f }{ \partial x_{j} } \displaystyle \frac{ \partial g_{j} }{ \partial x_{i} } ,\tag{5.145}  
$$
其中，Pa（xj）是计算图中xj的父节点的集合。方程（5.143）是一个函数的正向传播，而（5.145）是该梯度通过计算图的反向传播。对于神经网络训练，我们反向传播关于标签的预测误差。当我们有一个可以表示为计算图的函数时，其中初等函数是可微的。事实上，这个函数甚至可能不是一个数学函数，而是一个计算机程序。然而，并不是所有的计算机程序都可以自动微分，例如，如果我们不能找到微分的初等函数。编程结构，如循环和if语句，也需要更多的小心。

## 5.7 高阶导数

到目前为止，我们讨论了梯度，即一阶导数。有时，我们对更高阶的导数感兴趣，例如，当我们想要使用 Newton 法进行优化时，这需要二阶导数（Nocedal and Wright, 2006）。在5.1.1节中，我们讨论了使用多项式近似函数的 Taylor 级数。在多变量情况下，我们可以做完全相同的事情。在接下来的内容中，我们将详细讨论这一点。但让我们先从一些符号开始。

考虑一个函数 $f:\mathbb{R}^{2}\to \mathbb{R}$ 有两个变量 $x,y$。我们使用以下符号表示高阶偏导数（和梯度）：

$\frac{\partial^{2}f}{\partial x^{2}}$ 是 $f$ 关于 $x$ 的二阶偏导数。 $\frac{\partial^{n}f}{\partial x^{n}}$ 是 $f$ 关于 $x$ 的 $n$ 阶偏导数。 $\frac{\partial^{2}f}{\partial y\partial x}=\frac{\partial}{\partial y}(\frac{\partial f}{\partial x})$ 是先对 $x$ 求偏导，然后对 $y$ 求偏导得到的偏导数。 $\frac{\partial^{2}f}{\partial x\partial y}$ 是先对 $y$ 求偏导，然后对 $x$ 求偏导得到的偏导数。

 Hessian 矩阵是所有二阶偏导数的集合。

如果 $f(x,y)$ 是二次（连续）可微函数，那么 $\frac{\partial^{2}f}{\partial x\partial y}=\frac{\partial^{2}f}{\partial y\partial x}$，即求导顺序无关，并且相应的 Hessian 矩阵 $H=\left[\begin{array}{cc}\frac{\partial^{2}f}{\partial x^{2}}&\frac{\partial^{2}f}{\partial x\partial y}\\frac{\partial^{2}f}{\partial x\partial y}&\frac{\partial^{2}f}{\partial y^{2}}\end{array}\right]$ 是对称的。 Hessian 矩阵表示为 $\nabla_{x,y}^{2}f(x,y)$。一般来说，对于 $x\in \mathbb{R}^{n}$ 和 $f:\mathbb{R}^{n}\to \mathbb{R}$， Hessian 矩阵是一个 $n\times n$ 矩阵。 Hessian 矩阵衡量了函数在 $(x,y)$ 附近的局部曲率。注意（向量场的 Hessian 矩阵）：如果 $f:\mathbb{R}^{n}\to \mathbb{R}^{m}$ 是一个向量场， Hessian 矩阵是一个 $(m\times n\times n)$ 张量。

## 5.8 线性近似和多元 Taylor 级数

函数 $f$ 的梯度 $\nabla f$ 通常用于 $f$ 在 $x_{0}$ 附近的局部线性近似：

$$f(x)\approx f(x_{0})+(\nabla_{x}f)(x_{0})(x - x_{0})$$

这里 $(\nabla_{x}f)(x_{0})$ 是 $f$ 关于 $x$ 的梯度，在 $x_{0}$ 处求值。图5.12说明了这种线性近似。原函数被一条直线近似，这种近似在局部是准确的，但随着我们远离 $x_{0}$ 而变差。方程(5.148)是 $f$ 在 $x_{0}$ 处多元 Taylor 级数展开的特例，其中我们只考虑前两项。

我们考虑一个在 $x_{0}$ 处光滑的函数 $f:\mathbb{R}^{D}\to \mathbb{R}$，$x\mapsto f(x)$，$x\in \mathbb{R}^{D}$。当我们定义差向量 $\delta :=x - x_{0}$ 时，$f$ 在 $(x_{0})$ 处的多元 Taylor 级数定义为

$$f(x)=\sum_{k = 0}^{\infty}\frac{D_{x}^{k}f(x_{0})}{k!}\delta^{k}$$

其中 $D_{x}^{k}f(x_{0})$ 是 $f$ 关于 $x$ 的第 $k$ 阶（全）导数，在 $x_{0}$ 处求值。

$f$ 在 $x_{0}$ 处的 $n$ 阶 Taylor 多项式包含(5.151)中级数的前 $n + 1$ 项，定义为

$$T_{n}(x)=\sum_{k = 0}^{n}\frac{D_{x}^{k}f(x_{0})}{k!}\delta^{k}$$

在(5.151)和(5.152)中，我们使用了略显草率的 $\delta^{k}$ 记号，这对于向量 $x\in \mathbb{R}^{D}$，$D>1$，和 $k>1$ 是未定义的。注意，$D_{x}^{k}f$ 和 $\delta^{k}$ 都是 $k$ 阶张量，即 $k$ 维数组。$k$ 阶张量 $\delta^{k}\in \mathbb{R}^{\overbrace{D\times D\times\cdots\times D}^{k times}}$ 是通过向量 $\delta\in \mathbb{R}^{D}$ 的 $k$ 重外积得到的，用 $\otimes$ 表示。例如，$\delta^{2}:=\delta\otimes\delta=\delta\delta^{\top}$，$\delta^{2}[i,j]=\delta[i]\delta[j]$ $\delta^{3}:=\delta\otimes\delta\otimes\delta$，$\delta^{3}[i,j,k]=\delta[i]\delta[j]\delta[k]$

我们写出 Taylor 级数展开的前几项 $D_{x}^{k}f(x_{0})\delta^{k}$，其中 $k = 0,\cdots,3$ 且 $\delta :=x - x_{0}$：

$k = 0: D_{x}^{0}f(x_{0})\delta^{0}=f(x_{0})\in \mathbb{R}$
$k = 1: D_{x}^{1}f(x_{0})\delta^{1}=\underbrace{\nabla_{x}f(x_{0})}{1\times D}\underbrace{\delta}{D\times 1}=\sum_{i = 1}^{D}\nabla_{x}f(x_{0})[i]\delta[i]\in \mathbb{R}$
$k = 2: D_{x}^{2}f(x_{0})\delta^{2}=tr(\underbrace{H(x_{0})}{D\times D}\underbrace{\delta}{D\times 1}\underbrace{\delta^{\top}}{1\times D})=\delta^{\top}H(x{0})\delta=\sum_{i = 1}^{D}\sum_{j = 1}^{D}H[i,j]\delta[i]\delta[j]\in \mathbb{R}$
$k = 3: D_{x}^{3}f(x_{0})\delta^{3}=\sum_{i = 1}^{D}\sum_{j = 1}^{D}\sum_{k = 1}^{D}D_{x}^{3}f(x_{0})[i,j,k]\delta[i]\delta[j]\delta[k]\in \mathbb{R}$ 这里，$H(x_{0})$ 是 $f$ 在 $x_{0}$ 处求值的 Hessian 矩阵
例5.15（两个变量函数的 Taylor 级数展开） 考虑函数 $f(x,y)=x^{2}+2xy + y^{3}$。我们要在 $(x_{0},y_{0})=(1,2)$ 处计算 $f$ 的 Taylor 级数展开。

在开始之前，让我们讨论一下我们的预期：(5.161)中的函数是一个3次多项式。我们正在寻找 Taylor 级数展开，它本身是多项式的线性组合。因此，我们不期望 Taylor 级数展开包含四阶或更高阶的项来表示一个三阶多项式。这意味着，确定(5.151)的前四项应该足以精确地表示(5.161)的另一种形式。

为了确定 Taylor 级数展开，我们从常数项和一阶导数开始：

$f(1,2)=13$
$\frac{\partial f}{\partial x}=2x + 2y\Rightarrow\frac{\partial f}{\partial x}(1,2)=6$
$\frac{\partial f}{\partial y}=2x + 3y^{2}\Rightarrow\frac{\partial f}{\partial y}(1,2)=14$ 因此，我们得到 $D_{x,y}^{1}f(1,2)=\nabla_{x,y}f(1,2)=\left[\begin{array}{ll}6&14\end{array}\right]\in \mathbb{R}^{1\times 2}$ 使得 $\frac{D_{x,y}^{1}f(1,2)}{1!}\delta=\left[\begin{array}{ll}6&14\end{array}\right]\left[\begin{array}{l}x - 1\y - 2\end{array}\right]=6(x - 1)+14(y - 2)$
二阶偏导数是：

当我们收集二阶偏导数时，我们得到 Hessian 矩阵 $H=\left[\begin{array}{ll}\frac{\partial^{2}f}{\partial x^{2}}&\frac{\partial^{2}f}{\partial x\partial y}\\frac{\partial^{2}f}{\partial y\partial x}&\frac{\partial^{2}f}{\partial y^{2}}\end{array}\right]=\left[\begin{array}{rr}2&2\2&6y\end{array}\right]$，因此 $H(1,2)=\left[\begin{array}{cc}2&2\2&12\end{array}\right]\in \mathbb{R}^{2\times2}$

因此， Taylor 级数展开的下一项由以下给出： $\frac{D_{x,y}^{2}f(1,2)}{2!}\delta^{2}=\frac{1}{2}\delta^{\top}H(1,2)\delta=\frac{1}{2}\left[\begin{array}{ll}x - 1&y - 2\end{array}\right]\left[\begin{array}{cc}2&2\2&12\end{array}\right]\left[\begin{array}{l}x - 1\y - 2\end{array}\right]=(x - 1)^{2}+2(x - 1)(y - 2)+6(y - 2)^{2}$

三阶导数获得为 $D_{x,y}^{3}f=\left[\begin{array}{ll}\frac{\partial H}{\partial x}&\frac{\partial H}{\partial y}\end{array}\right]\in \mathbb{R}^{2\times2\times2}$

$D_{x,y}^{3}f[:,:,1]=\frac{\partial H}{\partial x}=\left[\begin{array}{cc}\frac{\partial^{3}f}{\partial x^{3}}&\frac{\partial^{3}f}{\partial x^{2}y}\\frac{\partial^{3}f}{\partial x\partial y\partial x}&\frac{\partial^{3}f}{\partial x\partial y^{2}}\end{array}\right]$

$D_{x,y}^{3}f[:,:,2]=\frac{\partial H}{\partial y}=\left[\begin{array}{cc}\frac{\partial^{3}f}{\partial y\partial x^{2}}&\frac{\partial^{3}f}{\partial y\partial x\partial y}\\frac{\partial^{3}f}{\partial y^{2}\partial x}&\frac{\partial^{3}f}{\partial y^{3}}\end{array}\right]$

由于 Hessian 矩阵(5.171)中的大多数二阶偏导数是常数，唯一非零的三阶偏导数是 $\frac{\partial^{3}f}{\partial y^{3}}=6\Rightarrow\frac{\partial^{3}f}{\partial y^{3}}(1,2)=6$

更高阶导数和3阶混合导数（例如，$\frac{\partial f^{3}}{\partial x^{2}\partial y}$）消失，因此 $D_{x,y}^{3}f[:,:,1]=\left[\begin{array}{ll}0&0\0&0\end{array}\right]$, $D_{x,y}^{3}f[:,:,2]=\left[\begin{array}{ll}0&0\0&6\end{array}\right]$

且 $\frac{D_{x,y}^{3}f(1,2)}{3!}\delta^{3}=(y - 2)^{3}$

这收集了 Taylor 级数的所有三次项。

总的来说，$f$ 在 $(x_{0},y_{0})=(1,2)$ 处的（精确） Taylor 级数展开是

$$ \begin{align*} f(x)&=f(1,2)+D_{x,y}^{1}f(1,2)\delta+\frac{D_{x,y}^{2}f(1,2)}{2!}\delta^{2}+\frac{D_{x,y}^{3}f(1,2)}{3!}\delta^{3}\ &=f(1,2)+\frac{\partial f(1,2)}{\partial x}(x - 1)+\frac{\partial f(1,2)}{\partial y}(y - 2)\ &+\frac{1}{2!}\left(\frac{\partial^{2}f(1,2)}{\partial x^{2}}(x - 1)^{2}+\frac{\partial^{2}f(1,2)}{\partial y^{2}}(y - 2)^{2}+2\frac{\partial^{2}f(1,2)}{\partial x\partial y}(x - 1)(y - 2)\right)+\frac{1}{6}\frac{\partial^{3}f(1,2)}{\partial y^{3}}(y - 2)^{3}\ &=13+6(x - 1)+14(y - 2)\ &+(x - 1)^{2}+6(y - 2)^{2}+2(x - 1)(y - 2)+(y - 2)^{3} \end{align*} $$

在这种情况下，我们得到了(5.161)中多项式的精确 Taylor 级数展开，即(5.180c)中的多项式与(5.161)中的原始多项式完全相同。在这个特定的例子中，这个结果并不令人惊讶，因为原始函数是一个三阶多项式，我们通过常数项、一阶、二阶和三阶多项式的线性组合在(5.180c)中表达了它。

## 5.9 拓展阅读

关于矩阵微分的更多细节，以及对所需线性代数的简短回顾，可以在Magnus和Neudecker（2007）中找到。自动微分有着悠久的历史，我们参考Griewank和Walther（2003）、Griewank和Walther（2008）、Elliott（2009）以及其中的参考文献。

在机器学习（以及其他学科）中，我们经常需要计算期望，即我们需要求解形如$E_{x}[f(x)] = \int f(x)p(x)dx$的积分。即使$p(x)$是一个方便的形式（例如 Gauss 分布），这个积分通常也不能解析求解。$f$的 Taylor 级数展开是找到近似解的一种方法：假设$p(x)=N(\mu,\sum)$是 Gauss 分布，那么在$\mu$附近的一阶 Taylor 级数展开将非线性函数$f$局部线性化。对于线性函数，如果$p(x)$是 Gauss 分布，我们可以精确计算均值（和协方差）（见6.5节）。扩展 Kalman 滤波器（Maybeck，1979）在非线性动态系统（也称为“状态空间模型”）的在线状态估计中大量利用了这一性质。其他确定性的方法来近似（5.181）中的积分包括无迹变换（Julier和Uhlmann，1997），它不需要任何梯度，或者 Laplace 近似（MacKay，2003；Bishop，2006；Murphy，2002），它使用二阶 Taylor 级数展开（需要Hessian）在其模式周围对$p(x)$进行局部 Gauss 近似。

## 练习

5.1 计算$f(x)=\log(x^{4})\sin(x^{3})$的导数$f'(x)$。
5.2 计算 Logistic 函数$f(x)=\frac{1}{1 + \exp(-x)}$的导数$f'(x)$。
5.3计算函数$f(x)=\exp(-\frac{1}{2\sigma^{2}}(x - \mu)^{2})$的导数$f'(x)$，其中$\mu,\sigma\in \mathbb{R}$是常数。
5.4计算$f(x)=\sin(x)+\cos(x)$在$x_{0}=0$处的 Taylor 多项式$T_{n}$，$n = 0,\cdots,5$。
5.5考虑以下函数：
$f_{1}(x)=\sin(x_{1})\cos(x_{2})$，$x\in \mathbb{R}^{2}$
$f_{2}(x,y)=x^{\top}y$，$x,y\in \mathbb{R}^{n}$
$f_{3}(x)=x x^{\top}$，$x\in \mathbb{R}^{n}$
    - （a）$\frac{\partial f_{i}}{\partial x}$的维度是什么？
    - （b）计算 Jacobi 矩阵。
5.6对$f$关于$t$求导，对$g$关于$x$求导，其中$f(t)=\sin(\log(t^{\top}t))$，$t\in \mathbb{R}^{D}$，$g(X)=tr(AXB)$，$A\in \mathbb{R}^{D\times E}$，$X\in \mathbb{R}^{E\times F}$，$B\in \mathbb{R}^{F\times D}$，其中$tr(.)$表示迹。
5.7使用链式法则计算以下函数的导数$df/dx$。提供每个单个偏导数的维度。详细描述你的步骤。
    - （a）$f(z)=\log(1 + z)$，$z = x^{\top}x$，$x\in \mathbb{R}^{D}$
    - （b）$f(z)=\sin(z)$，$z = Ax + b$，$A\in \mathbb{R}^{E\times D}$，$x\in \mathbb{R}^{D}$，$b\in \mathbb{R}^{E}$，其中$\sin(.)$应用于$z$的每个元素。
5.8计算以下函数的导数$df/dx$。详细描述你的步骤。
    - （a）使用链式法则。提供每个单个偏导数的维度。不需要显式计算偏导数的乘积。$f(z)=\exp(-\frac{1}{2}z)$，$z = g(y)=y^{\top}S^{-1}y$，$y = h(x)=x - \mu$，其中$x,\mu\in \mathbb{R}^{D}$，$S\in \mathbb{R}^{D\times D}$。
    - （b）$f(x)=tr(xx^{\top}+\sigma^{2}I)$，$x\in \mathbb{R}^{D}$。这里$tr(A)$是$A$的迹，即对角元素$A_{ii}$的和。提示：显式写出外积。
    - （c）使用链式法则。提供每个单个偏导数的维度。不需要显式计算偏导数的乘积。$f = \tanh(z)\in \mathbb{R}^{M}$，$z = Ax + b$，$x\in \mathbb{R}^{N}$，$A\in \mathbb{R}^{M\times N}$，$b\in \mathbb{R}^{M}$。这里，$\tanh$应用于$z$的每个组件。
5.9我们定义$g(z,\nu):=\log p(x,z)-\log q(z,\nu)$，$z := t(\epsilon,\nu)$，对于可微函数$p,q,t$以及$x\in \mathbb{R}^{D}$，$z\in \mathbb{R}^{E}$，$\nu\in \mathbb{R}^{F}$，$\epsilon\in \mathbb{R}^{G}$。使用链式法则计算梯度$\frac{d}{d\nu}g(z,\nu)$。
