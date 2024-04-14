Reference：Beskos, A., Roberts, G.O., 2005. Exact simulation of diffusions. Ann. Appl. Probab. 15, 2422–2444. https://doi.org/10.1214/105051605000000485



## Introduction

To be more precise, [5] first sample the variance at the final time point, using the known distribution of the solution of the square-root process, and consequently derive the Laplace transform of the conditional distribution of the integrated variance, relying on a result from [27].

更精确地说，[5]首先利用平方根过程解的已知分布对最终时间点的方差进行抽样，然后根据[27]的结果推导出积分方差条件分布的拉普拉斯变换。

In this paper, we adopt an analogous approach: we also sample the variance at the final time point first, and consequently derive the Laplace transform of the conditional distribution of the integrated variance. The technique is analogous to the one employed by [5] and is found in [9], where Lie Symmetry Methods are employed to derive Laplace transforms of functionals of diffusions, such as squared Bessel processes. Having obtained the Laplace transform of the distribution, we have reduced the problem to sampling from the lognormal distribution, which is trivial.

在本文中，我们采用了类似的方法:我们也对方差进行抽样的拉普拉斯变换，从而得到积分方差的条件分布。该技术类似于[5]使用的方法，在[9]中可以找到，其中李对称方法是用来推导扩散泛函的拉普拉斯变换，例如平方
贝塞尔过程。得到了分布的拉普拉斯变换，我们将问题简化为从对数正态分布中抽样。

From a mathematical point of view, the problem is also interesting: though not an affine process, the 3/2 model is still analytically tractable, in particular, the characteristic function of the logarithm of the stock price still has a closedform solution, see [19, 24]. Invoking a result obtained via Lie Symmetry Analysis, which also allows us to recover the result presented in [27], we manage to obtain the Laplace transform of the conditional distribution of the integrated variance. This emphasizes that results from Lie Symmetry Analysis can also be employed when designing Monte Carlo methods, whereas so far the main application of Lie Symmetry Analysis to finance has been in the derivation of closed-form pricing formulae, see e.g. [8], and [22].

从数学角度来看，这个问题也很有趣：尽管不是一个仿射过程，但 3/2 模型仍然具有解析可追踪性，特别是，股票价格对数的特征函数仍然有一个封闭形式的解，参见[19, 24]。通过调用通过 Lie 对称分析获得的结果，我们还可以恢复[27]中呈现的结果，从而得到积分方差的条件分布的拉普拉斯变换。这强调了 Lie 对称分析结果在设计蒙特卡罗方法时的应用，而迄今为止 Lie 对称分析在金融领域的主要应用是在推导封闭形式定价公式方面，参见例如[8]和[22]。

## The Monte Carlo Algorithm

![image-20240408194042504](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408194042504.png)

![image-20240408194906541](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408194906541.png)

Here Xt is a CIR Process (Mathematical Methods for Financial Markets (Springer Finance),pp pdf 381-383). 

![image-20240408210526277](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408210526277.png)

![image-20240408213624468](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408213624468.png)

These relationships are correct after being checked.

### Alhorithm 1

![image-20240408200945784](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408200945784.png)

![](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408203605471.png)

## The Distribution of the Conditional Integrated Variance

![image-20240408215935658](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240408215935658.png)

Using $A(t) = \frac{4t}{\epsilon^2}$, we can get

![image-20240411201554543](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411201554543.png)

To develop a formula for

$$E(exp\{-a\int_0^t\frac{ds}{X_s}\}|X_t, X_u)$$

To eliminate the random component of the drift term,

![image-20240411202037279](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411202037279.png)

![image-20240411205906014](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411205906014.png)

For 3.3

![image-20240411205934279](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411205934279.png)

For 3.2

![image-20240411210019428](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411210019428.png)

![image-20240411210044090](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411210044090.png)

这里相当于把条件期望乘以转移密度看作是 一个转移密度，在引用【5】的对应地方有

![image-20240411210222779](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411210222779.png)

最后可以得到

![image-20240411210801228](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411210801228.png)

## Implementation

![image-20240411224115154](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411224115154.png)

![image-20240411224144601](C:\Users\27261\AppData\Roaming\Typora\typora-user-images\image-20240411224144601.png)

## Variance Reduction Techniques

quasi-Monte Carlo