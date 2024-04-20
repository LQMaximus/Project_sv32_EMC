# class SvABC

这是所有波动率模型的抽象类，继承了smile.OptSmileABC, abc.ABC

## method params_kw

这段代码的目的是将父类和当前类中定义的模型参数合并到一个字典中，并返回这个合并后的字典。

## method init_benchmark

这段代码的作用是根据给定的参数集合初始化一个 SV 模型，并返回相应的参数和定价数据。



# class CondMcBsmABC

这是conditional MC的抽象类，用于所有的BSM基础上的sv模型

## method set_num_params

设置了MC的参数

这个方法实现了设置蒙特卡洛模拟（Monte Carlo simulation）参数的功能。具体来说：

- `n_path` 参数用于设置模拟的路径数量。
- `dt` 参数用于设置每步的时间间隔，通常用于欧拉（Euler）或米尔斯坦（Milstein）步骤。
- `rn_seed` 参数是随机数种子，用于生成随机数。
- `antithetic` 参数用于指定是否使用反向变量（antithetic variables）来减小模拟误差。

在方法内部，做了以下操作：

- 将传入的 `n_path` 参数转换为整数，并将其赋值给 `self.n_path` 属性，表示模拟的路径数量。
- 将传入的 `dt` 参数赋值给 `self.dt` 属性，表示每步的时间间隔。
- 将传入的 `rn_seed` 参数赋值给 `self.rn_seed` 属性，表示随机数种子。
- 将传入的 `antithetic` 参数赋值给 `self.antithetic` 属性，表示是否使用反向变量。
- 使用 `np.random.default_rng()` 创建了一个随机数生成器对象，并将其赋值给 `self.rng` 属性，用于生成随机数。
- 使用 `np.random.SeedSequence()` 创建了一个种子序列对象，并根据该序列生成了 6 个随机数生成器对象，并将它们放入列表中，然后将列表赋值给 `self.rng_spawn` 属性，用于生成一系列随机数。
- 初始化了一个空字典，并将其赋值给 `self.result` 属性，用于存储模拟结果。

## method base_model

​	return bsm.Bsm(vol, intr=self.intr, divr=self.divr, is_fwd=self.is_fwd)

## method tobs

返回均匀大小的观测时间数组。

## method rv_normal

用于生成服从标准正态分布的随机数

- 如果 `antithetic` 参数为 True（表示要使用反向变量），则会从对应的随机数生成器对象中生成半数数量的随机数（`self.n_path // 2`），然后将其与其相反数组成的数组通过 `np.stack` 函数沿着列的方向叠加在一起，最后通过 `flatten()` 函数将其展平，得到一维数组。
- 如果 `antithetic` 参数为 False，则直接从对应的随机数生成器对象中生成全部数量的随机数（`self.n_path`）。
- 返回生成的随机数数组 `zz`

## method rv_uniform

用于生成服从均匀分布的随机数

## method _bm_incr

用于计算增量布朗运动路径。具体来说：

- `tobs` 参数是观察时间点的数组，表示不同的观察时间。其中时间点 0 不包括在内。
- `cum` 参数是一个布尔值，如果为 True，则返回累积值；如果为 False，则返回增量值。
- `n_path` 参数表示路径数量，如果为 None，则使用存储的路径数量。

函数首先计算观察时间点之间的时间间隔 `dt`，然后根据观察时间点和时间间隔生成增量布朗运动路径。具体步骤如下：

1. 如果 `n_path` 为 None，则使用存储的路径数量 `self.n_path`。
2. 如果 `antithetic` 参数为 True，则先生成半数数量的标准正态分布随机数矩阵，并将其与其相反的补数相乘，然后通过 `np.sqrt(dt[:, None])` 将每个时间点的时间间隔开方，并将随机数与开方后的时间间隔相乘得到增量布朗运动路径的增量值。最后通过 `reshape((-1, n_path))` 将结果展平为二维数组。
3. 如果 `antithetic` 参数为 False，则直接生成相应数量的标准正态分布随机数矩阵，并进行类似的操作得到增量值。
4. 如果 `cum` 参数为 True，则对每条路径进行累积求和操作。

最终返回增量布朗运动路径的增量值或累积值。



## abc method cond_spot_sigma

用于在给定波动率路径（例如，sigma_T，积分方差）的情况下返回新的远期价格和波动率。具体来说：

- `texp` 参数表示到期时间。
- `var_0` 参数表示初始方差（或波动率）。

该方法返回一个元组 `(forward, volatility)`，其中：

- `forward` 是新的标准化远期价格，按照 F_0 = 1 进行标准化，因此需要乘以原始的 F_0 值。
- `volatility` 是新的标准化波动率，按照 sigma_0 = 1 进行标准化，因此需要乘以原始的 sigma_0 值。

**这个方法返回的是在使用conditional MC计算期权价格的时候，它的等价期初价格F_0和等价波动率σ**

## method price

1. `kk` 是行权价格 `strike` 与标的资产当前价格 `spot` 的比率。这样做是为了将行权价格标准化，使其相对于当前价格。
2. `fwd_mean = fwd_cond.mean()`：计算未来期望标的资产价格的均值。
3. `self.result['spot error'] = fwd_mean - 1`：将修正后的期望标的资产价格均值与1进行比较，并将结果存储在 `self.result` 字典中的 `'spot error'` 键中。这个值表示修正后的均值与1之间的偏差，可以用来评估修正的效果。
4. `if self.correct_fwd: fwd_cond /= fwd_mean`：如果 `correct_fwd` 标志为真，则对未来期望标的资产价格进行修正，使其除以均值 `fwd_mean`。这个修正的目的是确保未来期望标的资产价格的均值为1。
5. `sigma = np.sqrt(self.sigma) if self.var_process else self.sigma`：如果 `var_process` 为真，则将 `self.sigma` 开平方，否则保持不变，这是因为在某些模型中，波动率参数是方差，需要转换为标准差。这个标准差 `sigma` 将用于构建基础模型。
6. `base_model = self.base_model(sigma * sigma_cond)`：使用修正后的标准差 `sigma * sigma_cond` 初始化基础模型，其中 `sigma_cond` 是在给定波动率路径后的条件波动率。
7. `price_grid = base_model.price(kk[:, None], fwd_cond, texp=texp, cp=cp)`：调用基础模型的 `price` 方法来计算期权价格的网格。这个网格是在不同的行权价格（`kk`）、不同的未来期望标的资产价格（`fwd_cond`）和给定的到期时间（`texp`）下计算的。
8. `price = spot * np.mean(price_grid, axis=1)`：计算每个行权价格下的期权价格。将期权价格乘以当前的标的资产价格 `spot`，因为期权的价格通常是相对于标的资产价格的。然后计算每个行权价格下的期权价格的均值。
9. `return price[0] if scalar_output else price`：如果 `scalar_output` 为真，则返回期权价格的第一个值（通常在只有一个行权价格时使用）。否则返回完整的期权价格数组。

**这里的思路主要是，在最后一步前都使用标准化的值，即，之前的函数返回的是期初价格是 乘期初价格之前的值，之前函数返回的sigma是 乘sigma之前的值**



# class Sv32McABC

  model_type = "3/2"

  var_process = True

  scheme = None

  _m_heston = None

## method set_num_params

1. 首先，调用父类的 `set_num_params` 方法，将参数传递给父类，确保模型的参数正确设置。
2. 然后，根据已有的模型参数计算其他参数，例如 `mr`（均值回归率）、`theta`（长期波动率方差）等。
3. 使用计算得到的参数，初始化 Heston 模型 (`self._m_heston`)，并调用其 `set_num_params` 方法，将路径数、时间步长、随机数种子和是否使用反向变量传递给 Heston 模型，以确保 Heston 模型也使用相同的参数配置。

## static method iv_complex

定义了一个静态方法 `iv_complex`，用于计算具有复数参数的修正的第一类贝塞尔函数（Modified Bessel function of the first kind）。贝塞尔函数在数学和物理中都有广泛的应用，通常用于描述振动、波动和横截面分布等问题。

具体来说，这个静态方法接受两个参数：

- `nu`：贝塞尔函数的指数，可以是任意实数或复数。
- `zz`：函数的自变量，可以是任意复数。

然后，根据给定的参数，该方法使用递推关系计算并返回具有复参数的修正的第一类贝塞尔函数的值。

## static method iv_d12

用于计算修正的第一类贝塞尔函数（Modified Bessel function of the first kind）关于指数 `nu` 的一阶和二阶导数。

具体来说，这个静态方法接受两个参数：

- `nu`：贝塞尔函数的指数，可以是任意实数或复数。
- `zz`：函数的自变量，可以是任意复数。

然后，根据给定的参数，该方法使用数值递推的方法计算修正的第一类贝塞尔函数关于指数 `nu` 的一阶和二阶导数，并将结果作为元组 `(iv1, iv2)` 返回。

在代码中，通过递推关系和数值计算的方法来逐步计算一阶导数 `iv1` 和二阶导数 `iv2`，以求得指定参数下的贝塞尔函数的导数值。

## method cond_avgvar_mv

1. `phi`是一个函数的输出结果，但我们在这里只使用了它的值。
2. `nu`是一个常数，计算了Heston模型的卡方自由度的一半减去1。
3. `vov2dt`是Heston模型中波动率的方差乘以时间步长。
4. `d1_nu_bb`和`d2_nu_bb`是根据给定的参数计算的中间变量。
5. `zz`是根据`phi`、初始方差和最终方差计算的中间变量。
6. 如果未提供`eta`，则使用修正的第一类贝塞尔函数（`iv`）、其一阶和二阶导数（`iv_d1`和`iv_d2`）计算`d1`和`var`。
7. 如果提供了`eta`，则使用一些其他函数和参数计算`d1`和`var`。
8. 修正可能出现的负值
9. 返回计算得到的`d1`和`var`。

**该函数返回的是avgvar的均值和方差**

## method cond_spot_sigma

​    tobs = self.tobs(texp)

​    dt = np.diff(tobs, prepend=0)

​    n_dt = len(dt)

​    var_t = np.full(self.n_path, var_0)

​    avgvar = np.zeros(self.n_path)

​    for i in range(n_dt):

​      var_t, avgvar_inc = self.cond_states_step(dt[i], var_t)

​      avgvar += avgvar_inc * dt[i]

​    avgvar /= texp

​    spot_cond = (np.log(var_t/var_0) - texp * (self.mr * self.theta - (self.mr + self.vov**2/2)*avgvar)) / self.vov\

​      \- self.rho * avgvar * texp / 2

​    np.exp(self.rho * spot_cond, out=spot_cond)

​    sigma_cond = np.sqrt((1.0 - self.rho**2) * avgvar / var_0)  # normalize by initial variance

​    \# return normalized forward and volatility

​    return spot_cond, sigma_cond

**如果未重新定义，那么这里是使用步进的方法来得到var_t和avgvar，每步执行的步骤在子类中的cond_states_step中定义，**

**注意此处得到的var_t是在var_0基础上的，得到的avgvar是除以texp的，最后的sigma_cond是除以var_0的**



# class Sv32McTimeStep

milstein scheme

## method var_step_euler

使用 Euler/Milstein scheme来进行每步操作



## method cond_states_step

self.scheme对应的值：

self.scheme == 1，则为Euler/Milstein scheme， 调用var_step_euler来生成var_t和avgvar

self.scheme == 2，则为Exact method来模拟dt的每一步，通过NCX2来模拟var_t

self.scheme == 3，则为Almost exact method来模拟dt的每一步，通过Poison-Gamma来模拟var_t

avgvar = (var_0 + var_t)/2



# class Sv32McBaldeaux2012Exact

## method cond_avgvar_laplace

**在给出var_0, var_t时，得到对应的laplace变换**

**注意这里的var是$\frac{1}{X_t}$**，也即v_t，$X_t$对应的是分布，而它的倒数对应的是真正的var

## method draw_cond_avgvar

在给出var_0, var_t时，得到对应的avgvar

## method cond_states_step

进行每一步的步进，得到每一步的var_t和avgvar_t



# class Sv32McChoiKwok2023Ig

## method draw_from_mv

通过均值和方差来拟合avgvar，得到的就是avgvar的最终结果

## method cond_avgvar_mv_numeric

通过Laplace变化的数值导数来得到均值和方差

## method cond_states_step_invlap
