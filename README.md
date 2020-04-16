# 线性混合模型的power分析

随着心理学的可重复性日渐受到重视，对统计分析的power计算的需求也日益增多，
多数初级统计方法 (`t test`, `ANOVA` 等) 的power计算可以在某些软件 (比如`jamovi`) 中完成。
但作为一种高级统计方法，线性混合模型 (linear mixed model, LMM) 的power计算尚不能在`jamovi`这样具有可视化界面的软件中进行，
仍需利用`R`中的相关数据包及函数。本文主要介绍在 `R` 中如何进行LMM的power计算，内容包括：数据及其函数介绍，可适用及受限制的情景，
以及具体的示例演示。

## 数据包及其函数

[`simr`](https://cran.r-project.org/web/packages/simr/index.html) 数据包是 `R` 中进行LMM的power分析较为便利的包。
在分析中，依靠蒙泰卡罗模拟的方式来模拟模型数据，并且计算在设定的模拟次数中，达到显著的比例。主要的函数为`powerSim()`。

## 适用情景

* 计算后验效应的power，即根据基于已有数据所建模型中的效应来计算power；

* 计算先验效应的power，即根据已有研究中某效应的大小来计算power；

* 预估被试量/项目量，因为power的大小受到被试或项目数量的影响，具体模式为数量越多，power越大，
这为我们提供了根据预实验的效应来估计正式实验所需被试量/项目量的功能；

## 限制情景

* 整个power分析耗时较长

（目前该数据包作者也在试图解决这个问题，用户也可以采用多线程计算的方式来缓解）。

## 示例

### 计算后验效应的power

这里我们利用一个心理学实验的数据([数据下载地址])来演示，
该实验收集了10名被试(`subj`)在执行一项认知任务时的反应时(`DV`)，实验有两个条件(`CondA`和`CondB`)，为`2*2`被试内和项目(`item`)内设计。
即同时考虑被试和项目两个随机因子。

> 因为这里我们关注固定效应，非随机效应，因此建模时在随机效应部分只考虑随机截距，固定效应部分考察两个主效应及其交互作用，

建模如下：
```
> Model = lmer(data = DemoData, DV~CondA*CondB+(1|subj)+(1|item))
```

#### 考察固定效应的power

模型的固定效应部分为:
```
> summary(Model)$coef
#                  Estimate Std. Error        df    t value     Pr(>|t|)
# (Intercept)     279.43090   23.37537  11.71977 11.9540740 6.422210e-08
# CondAA2          24.29565   13.49694 498.43142  1.8000866 7.245150e-02
# CondBB2          12.18484   13.36445 509.31328  0.9117351 3.623395e-01
# CondAA2:CondBB2 -32.82881   19.29629 509.76684 -1.7013020 8.949610e-02
```

这里以考察二者交互作用的固定效应(`CondAA2:CondBB2`)为例：

```
> PowerAB_ttest = simr::powerSim(fit = Model, # 要考察的模型
                                 test = fixed('CondAA2:CondBB2', # 要考察的固定效应的名称
                                              method = 't'), # 选取检验方法，因为固定效应为t检验，因此method设置为t
                                 nsim=50) # 设置模拟次数，建议设置为500 (此时可以获取到较稳定的power)
```
```
> PowerAB_ttest
# Power for predictor 'CondAA2:CondBB2', (95% confidence interval):
#       46.00% (31.81, 60.68)
# 
# Test: t-test with Satterthwaite degrees of freedom (package lmerTest)
#       Effect size for CondAA2:CondBB2 is -33.
# 
# Based on 50 simulations, (2 warnings, 0 errors)
# alpha = 0.05, nrow = 553
# 
# Time elapsed: 0 h 0 m 8 s
# 
# nb: result might be an observed power calculation
```

交互作用固定效应的power为`0.46`(置信区间为`0.318 – 0.607`)。

同理，计算`CondA`的固定效应的power可通过以下代码实现：
```
> PowerA_ttest = simr::powerSim(fit = Model, test = fixed('CondAA2', method = 't'), nsim=50)
```

#### 计算主效应/交互作用的power

模型的主效应/交互作用部分为：
```
> anova(Model)
# Type III Analysis of Variance Table with Satterthwaite's method
#             Sum Sq Mean Sq NumDF  DenDF F value Pr(>F)  
# CondA         8546    8546     1 489.18  0.6694 0.4137  
# CondB         2453    2453     1 509.78  0.1921 0.6613  
# CondA:CondB  36951   36951     1 509.77  2.8944 0.0895 .
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```

这里以考察`CondA`主效应为例：
```
> PowerA_Ftest = simr::powerSim(fit = Model, # 要考察的模型
                                test = fixed('CondA', # 要考察的主效应/交互作用的名称
                                             method = 'f'), # 选取检验方法，因为主效应为f检验，因此method设置为f
                                nsim=50) # 设置模拟次数，建议设置为500 (此时可以获取到较稳定的power)

```
```
> PowerA_Ftest
# Power for predictor 'CondA', (95% confidence interval):
#       12.00% ( 4.53, 24.31)
# 
# Test: Type-II F-test (package car)
# 
# Based on 50 simulations, (50 warnings, 0 errors)
# alpha = 0.05, nrow = 553
# 
# Time elapsed: 0 h 0 m 58 s
# 
# nb: result might be an observed power calculation
```

`CondA`的主效应的power为`0.12`(置信区间为`0.453 – 0.243`)。

### 计算先验效应的power

如果先前研究中已经得到了较稳定的效应量，比如发现`CondA`的固定效应一般在50左右,也可以根据先验的效应来计算power。步骤如下：

1. 将模型的后验固定效应改为先验固定效应

```
> fixef(Model) # 查看模型固定效应
#     (Intercept)         CondAA2         CondBB2 CondAA2:CondBB2 
#       279.43090        24.29565        12.18484       -32.82881 
```
```
> fixef(Model)[2] = 50 # 修改CondA的效应
> fixef(Model) # 查看修改后的固定效应
#     (Intercept)         CondAA2         CondBB2 CondAA2:CondBB2 
#       279.43090        50.00000        12.18484       -32.82881 
```

2. 根据先验固定效应计算power(与计算后验power的方法相同)

```
> PowerA_ttest = simr::powerSim(fit = Model, test = fixed('CondAA2', method = 't'), nsim=50)
> PowerA_ttest
# Power for predictor 'CondAA2', (95% confidence interval):
#       88.00% (75.69, 95.47)
# 
# Test: t-test with Satterthwaite degrees of freedom (package lmerTest)
#       Effect size for CondAA2 is 50.
# 
# Based on 50 simulations, (3 warnings, 0 errors)
# alpha = 0.05, nrow = 553
# 
# Time elapsed: 0 h 0 m 9 s
```
此时`CondA`的固定效应power为`0.88`.

```
> PowerA_Ftest = simr::powerSim(fit = Model, test = fixed('CondA', method = 'f'), nsim=50)
> PowerA_Ftest
# Power for predictor 'CondA', (95% confidence interval):
#       94.00% (83.45, 98.75)
# 
# Test: Type-II F-test (package car)
# 
# Based on 50 simulations, (50 warnings, 0 errors)
# alpha = 0.05, nrow = 553
# 
# Time elapsed: 0 h 1 m 4 s
```
`CondA`的主效应power为`0.94`.

### 预估被试量

```
> Model = lmer(data = DemoData, DV~CondA*CondB+(1|subj)+(1|item)) # 这里仍以后验power为例
```

前面已知交互作用固定效应的power为`0.46`，我们想考察被试量为多少时，该power可以达到`0.8`。步骤如下：

1. 建立扩充被试后的模型，比如这里扩充为30名被试

```
> Model2 = extend(object = Model, along = 'subj', n = 30)
```

2. 计算此时的power
```
> PowerA_ttest2 = powerSim(Model2, fixed('CondAA2', method = 't'), nsim=50)

> PowerA_ttest2
# Power for predictor 'CondAA2', (95% confidence interval):
#       84.00% (70.89, 92.83)
# 
# Test: t-test with Satterthwaite degrees of freedom (package lmerTest)
#       Effect size for CondAA2 is 24.
# 
# Based on 50 simulations, (6 warnings, 0 errors)
# alpha = 0.05, nrow = 1659
# 
# Time elapsed: 0 h 0 m 11 s
# 
# nb: result might be an observed power calculation
```

此时power已经升到`0.84`。

3. 如果想知道具体的power随被试量增大的变化趋势，可利用`powerCurve()`函数

```
> Pcurve = powerCurve(fit = Model2, test = fixed('CondAA2', method = 't'), along = 'subj', nsim=50)

> Pcurve
# Power for predictor 'CondAA2', (95% confidence interval), by largest value of subj:
#      11: 22.00% (11.53, 35.96) - 180 rows
#      14: 36.00% (22.92, 50.81) - 325 rows
#      17: 50.00% (35.53, 64.47) - 507 rows
#       2: 58.00% (43.21, 71.81) - 667 rows
#      22: 64.00% (49.19, 77.08) - 837 rows
#      25: 58.00% (43.21, 71.81) - 989 rows
#      28: 74.00% (59.66, 85.37) - 1166 rows
#      30: 82.00% (68.56, 91.42) - 1318 rows
#       6: 86.00% (73.26, 94.18) - 1489 rows
#       9: 90.00% (78.19, 96.67) - 1659 rows
# 
# Time elapsed: 0 h 1 m 39 s
```

**注意**，此时每个power前面的数值不准确，真实的数值为:
```
> Pcurve$nlevels
# [1]  3  6  9 12 15 18 21 24 27 30
```

可以看到大概到24名被试时，power可以到`0.8`以上.

可以人为设置样本量的范围:
```
> Pcurve = powerCurve(fit = Model2, test = fixed('CondAA2', method = 't'), along = 'subj', nsim=50, 
                      breaks=c(21, 22, 23, 24, 25, 30)) # 考察样本量为21、22、23、24、25、30时的power

> Pcurve
# Power for predictor 'CondAA2', (95% confidence interval), by largest value of subj:
#      28: 72.00% (57.51, 83.77) - 1166 rows
#      29: 74.00% (59.66, 85.37) - 1220 rows
#       3: 70.00% (55.39, 82.14) - 1262 rows
#      30: 74.00% (59.66, 85.37) - 1318 rows
#       4: 76.00% (61.83, 86.94) - 1369 rows
#       9: 80.00% (66.28, 89.97) - 1659 rows
# 
# Time elapsed: 0 h 1 m 12 s

> Pcurve$nlevels
# [1] 21 22 23 24 25 30
```

### 计算广义线性混合模型中效应的power

广义模型中效应的power计算与上面的类似，主要区别是：

* 考察固定效应时，`fixed()`中`method`参数应设置为`'z'`，因为广义模型中固定效应为z检验;
* 考察主效应时，`fixed()`中`method`参数应设置为`'chisq'`，因为广义模型中固定效应为卡方检验

[数据下载地址]:https://github.com/usplos/YawMMF/raw/master/data/DemoData.rda


