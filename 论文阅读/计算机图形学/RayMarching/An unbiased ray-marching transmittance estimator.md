# An unbiased ray-marching transmittance estimator

![image-20210530132048687](An unbiased ray-marching transmittance estimator.assets/image-20210530132048687.png)

## 1. INTRODUCTION

场景中两点之间的**可见度**是**光传输模拟**中的一个基本量值。在真空中，它有一个二进制的值。然而，在参与介质中，`scalar radiative transfer`被用来统计散射和吸收粒子的存在。与给定射线相交的粒子的数量是一个随机变量，可见度成为一个分数量。

![image-20210530132323522](An unbiased ray-marching transmittance estimator.assets/image-20210530132323522.png)

在本文中，作者提出了对==方程1==进行**无偏估计**的新方法。作者提出的计算器是**基于指数函数的低阶泰勒级数扩展**。使用**低阶扩展**可以释放**采样预算**，以便更准确地计算**扩展点**和**泰勒级数项**，这进一步允许降低**计算顺序**并改善样本。这种自我强化的循环导致了一个**无偏的低方差估计器**，在大多数情况下只评估已经相当准确的**二阶项**。对这个项的评价可以与**经典的抖动射线行进方案**相提并论，而其余的项可以被看作是**概率采样的修正项**，使其成为无偏的，所以作者把技术称为**无偏射线行进**。



## 2. BACKGROUND AND RELATED WORK

透过率是**负光学深度**的指数：

![image-20210530132944738](An unbiased ray-marching transmittance estimator.assets/image-20210530132944738.png)

使用射线行进法)或通过蒙特卡洛方法可以很容易拟合**光学深度**，但指数将导致有偏估计[Raab等人2006]。jackknife方法及其概括[Miller 1974]在某些情况下可以用来减少偏差，但对于某些应用来说，误差可能仍然不能接受。因此，**透射率估计的关键挑战**是如何在只给定点样本的情况下形成**无偏的估计**。

### Poisson point processes

许多无偏方法被设计出来，以计算类似**方程1**的指数积分，这些方法与`point processes`的**零阶估计问题**密切相关。一个点过程$N(l)$是在某个时间`ℓ`内（或沿着单位长度射线）发生的事件数量的**随机计数过程**。对于**泊松点过程**（`PPP`）来说，事件是独立的，并且是泊松分布的，`rate`为$\lambda _ℓ$。这个`rate`是该过程的**强度函数**在区间内的积分：

![image-20210530133547471](An unbiased ray-marching transmittance estimator.assets/image-20210530133547471.png)

众所周知，由于假设了**独立的散射中心**，这个PPP正是**经典辐射传递**中支配散射和吸收事件的过程。**两者之间的对应关系**是通过将点过程的**$\lambda(x)$**等同于从`a`出发时在区间内移动时的**消光系数**$\mu(x)$来建立的，$\lambda(x)=\mu(a+x)$。然后，**透射率**是沿区间没有发现**点/粒子的概率**：

![image-20210530134153312](An unbiased ray-marching transmittance estimator.assets/image-20210530134153312.png)

由于PPP的平均值是`rate`：(下式1)，`classical radiative transfer `的**指数自由路径**遵循**泊松分布的零阶概率**（具有rate $\tau$的**泊松分布**的**概率质量函数**是 $e^{-\tau}\tau^k/k!$)。

![image-20210530134450041](An unbiased ray-marching transmittance estimator.assets/image-20210530134450041.png)

### Tracking estimators

最著名的`unbiased transmittance estimators`被称为`tracking estimators`，这是因为它们通过对`PPP`进行采样，确定与**介质碰撞**的有序序列，从而跟踪从`a`到`b`的粒子。对于**恒定密度的介质**，碰撞之间**指数分布的自由路径长度**很容易被采样。对于非均质介质，可以使用`delta-tracking`方法对PPP进行采样。可以对一个**更密集的过程**进行采样，其**rate/光深**很容易被计算出来

![image-20210530135449425](An unbiased ray-marching transmittance estimator.assets/image-20210530135449425.png)

然后，**一个拒绝过程**被用来将**密集的过程**稀释到**所需的结果**，即每个采样点$x_i$，以概率$\mu(x_i)/\overline{\mu}(x_i)$被保留。这种拒绝体现了`transport`文献中的**虚构/空洞碰撞概念**（`fictitious/null collision concept`）。我们注意到，这相当于`point process`文献中的一种方法，被称为`thinning`。

虽然**主要成分**$\overline{\mu}$往往是一个常数，但如果用一个更严格限定**目标密度**的大数，`delta-tracking`的效率就会提高。片断-线性`Piecewise-linear`或片断-多项式`piecewise-polynomial`主项可以被有效采样。关于**非均质PPPs采样方法**的研究见论文。

值得注意的是，在不知道$\tau$的情况下，`delta-tracking`可以从**泊松分布**$N(ℓ)∼Po(\tau)$中采样**碰撞次数**。给出**均值**为$\overline{v}$的`n`个$N(ℓ)$样本，==透射率==的**最小方差无偏估计值**（仅给出$\overline{v}$）为：

![image-20210607153628132](An unbiased ray-marching transmittance estimator.assets\image-20210607153628132-1623051391242.png)

#### Ratio tracking

加权跟踪（也称为图形中的Ratio tracking），将期望值优化应用于`n=1`的delta Tracking estimators，形成密度比率的乘积（空密度$\mu_n(x)=\overline{\mu}(x-\mu(x))$到总密度$\overline{\mu}(x)$)。这与一种被称为**加权delta跟踪**的**距离抽样方案**密切相关。与**delta跟踪**一样，majorant PPP在$(a,b)$中进行泊松采样。`Ratio tracking`不是在采样到一个真实的粒子后立即返回`0`，而是根据每个采样粒子的**虚构概率**，为其赋予一个分数的不透明度：{7}

![image-20210607154509274](An unbiased ray-marching transmittance estimator.assets\image-20210607154509274.png)

在大多数情况下，==Ratio tracking==优于delta跟踪。然而，delta跟踪可以在第一个实数部分被采样后，使用早期终止，避免许多不必要的密度评估。因此，在上诉方程中的**运行积**低于某个**阈值**之后，切换到**delta跟踪**。



### Control variates

==透射率估计==的一个共同主题是利用**辅助密度函数**（空碰撞密度，控制密度）。这些辅助函数可以达到不同的目的，例如促进**碰撞的采样**或**减少方差**，但它们（或它们的组合）可以被解释为==控制变量==（CV）。给定一个可分析的、可积分的控制变量$\mu_c(x)$，$\tau_c=\int^b_a{\mu_cdx}$，**光学深度积分**可以被改写为：

![image-20210607155342002](An unbiased ray-marching transmittance estimator.assets\image-20210607155342002.png)

我们将把$\mu_c(x)$和$\mu_r(x)=\mu(x)-\mu_c(x)$称为==控制和残余消光系数==，把它们各自的积分$\tau_c$和$\tau_r$称为==控制和残余光深==。主残留系数$$\overline{\mu}_r(x)=\overline{\mu}(x)-\mu_c(x)$$和光学深度$\overline{\tau}_r=\overline{\tau}-\tau_c$。然后，透射率为

![image-20210607160007650](An unbiased ray-marching transmittance estimator.assets\image-20210607160007650.png)

这种转换可以极大地减少方差。



#### Residual ratio tracking / Poisson estimator.

**控制变量**在**透射率估计**中的第一个应用是Ratio tracking，以产生**剩余比率跟踪（RRT）估计器**。应用公式9中的转换，估计器为

![](An unbiased ray-marching transmittance estimator.assets\image-20210607160311967.png)

### Power-series formulation

另一个**无偏的透射率估计系列**来自于方程1中**指数的幂级（泰勒）扩展**。Bhanot和Kennedy提出的这种形式，用于估计粒子物理学**马尔科夫链蒙特卡洛模拟**中的**哈密尔顿指数**，可以直接应用于**透射率估计**。

Georgiev等人[2019]首次将这一表述引入计算机图形学，表明它实际上可以被看作是表达和分析所有透射率估计器的一个非常通用的框架。根据他们的推导，透射率（1）可以表示为：

![image-20210615145031117](An unbiased ray-marching transmittance estimator.assets/image-20210615145031117.png)

