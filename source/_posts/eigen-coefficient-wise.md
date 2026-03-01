---
title: Eigen中作用于系数的函数
date: 2026-03-01 15:07:00
tags: Eigen
categories:
  - [C++, Eigen]
---

对于作用于Array类型下文表示为`a.XXX(...)`,
对于作用于Matrix类型下文表示为`m.XXX(...)`,
# 基础函数

| 调用方法                                                   | 功能描述                        |
| ------------------------------------------------------ | --------------------------- |
| `a.abs();`<br>`abs(a);`<br>`m.cwiseAbs();`             | 绝对值($\|a_i\|$)              |
| `a.inverse();`<br>`inverse(a);`<br>`m.cwiseInverse();` | 求倒数($1/a_i$)                |
| `a.conjugate();`<br>`conj(a);`<br>`m.conjugate();`     | 求共轭复数($\bar{a_i}$)<br>对实数无效 |
| `a.arg();`<br>`arg(a);`<br>`m.cwiseArg();`             | 复数的相位角                      |
# 指数对数函数

| 调用方法                    | 功能描述                  |
| ----------------------- | --------------------- |
| a.exp();<br>exp(a);     | 以e为底的指数($e^{a_i}$)    |
| a.log();<br>log(a);     | 以e为底的对数($\ln({a_i})$) |
| a.log1p();<br>log1p(a); | $\ln({1+a_i})$        |
| a.log10();<br>log10(a); | $\log_{10}({a_i})$    |
# 幂函数

| 调用方法                                    | 功能描述              |
| --------------------------------------- | ----------------- |
| a.pow(b);<br>pow(a,b);                  | $a_i ^ {b_i}$     |
| a.sqrt();<br>sqrt(a);<br>m.cwiseSqrt(); | $\sqrt a_i$       |
| a.cbrt();<br>cbrt(a);<br>m.cwiseCbrt(); | $\sqrt[3]{ a_i }$ |
| a.rsqrt();<br>rsqrt(a);                 | $1/{\sqrt a_i}$   |
| a.square();<br>square(a);               | $a_i^2$           |
| a.cube();<br>cube(a);                   | $a_i^3$           |
| a.abs2();<br>abs2(a);<br>m.cwiseAbs2(); | $\|a_i\|^2$       |
# 三角函数

| 调用方法                  | 功能描述 |
| --------------------- | ---- |
| a.sin();<br>sin(a);   | 计算正弦 |
| a.cos();<br>cos(a);   | 计算余弦 |
| a.tan();<br>tan(a);   | 计算正切 |
| a.asin();<br>asin(a); | 反正弦  |
| a.acos();<br>acos(a); | 反余弦  |
| a.atan();<br>atan(a); | 反正切  |
# 双曲函数

| 调用方法                    | 功能描述  |
| ----------------------- | ----- |
| a.sinh();<br>sinh(a);   | 双曲正弦  |
| a.cohs();<br>cosh(a);   | 双曲余弦  |
| a.tanh();<br>tanh(a);   | 双曲正切  |
| a.asinh();<br>asinh(a); | 反双曲正弦 |
| a.cohs();<br>acosh(a);  | 反双曲余弦 |
| a.atanh();<br>atanh(a); | 反双曲正切 |
# 浮点数取整函数

| 调用方法                    | 功能描述           |
| ----------------------- | -------------- |
| a.ceil();<br>ceil(a);   | 取不小于的最近整数      |
| a.floor();<br>floor(a); | 取不大于的最近整数      |
| a.round();<br>round(a); | 四舍五入，中间值取远离0的值 |
| a.rint();<br>rint(a);   | 舍入到最近偶数        |
# 数值检测

| 调用方法                          | 功能描述     |
| ----------------------------- | -------- |
| a.isFinite();<br>isfinite(a); | 数值为有限值   |
| a.isInf();<br>isinf(a);       | 数值为无限值   |
| a.isNaN();<br>isnan(a);       | 数值为非数NaN |
# 非正式功能函数
位于 `#include <unsupported/Eigen/SpecialFunctions>`中
## 误差与伽马函数

| 调用方法                        | 功能描述                                                                                                                                              |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| a.erf();<br>erf(a);         | 误差函数（标准正态分布下，对称区间的概率）<br>$\operatorname{erf}(x) = \frac{2}{\sqrt{\pi}} \int_0^x e^{-t^2} dt$                                                      |
| a.erfc();<br>erfc(a);       | 补误差函数（误差函数结果的补集）<br>$\operatorname{erfc}(x)=1-\operatorname{erf}(x)$<br>$\operatorname{erfc}(x) = \frac{2}{\sqrt{\pi}} \int_x^\infty e^{-t^2} dt$ |
| a.lgamma();<br>lgamma(a);   | 伽马函数绝对值的自然对数<br>$\text{lgamma}(x) = \ln {\| \Gamma(x) \|}$                                                                                        |
| a.digamma();<br>digamma(a); | Gamma 函数对数的导数<br>$\psi(x) = \frac{d}{dx} \ln \Gamma(x) = \frac{\Gamma'(x)}{\Gamma(x)}$                                                            |
| igamma(a,x);                | 下不完全伽马函数<br>$\gamma(a_i,x_i)= \frac{1}{\|a_i\|} \int_{0}^{x_i}e^{\text{-}t} t^{a_i-1} \mathrm{d} t$                                               |
| igammac(a,x);               | 上不完全伽马函数<br>$\Gamma(a_i,x_i) = \frac{1}{\|a_i\|} \int_{x_i}^{\infty}e^{\text{-}t} t^{a_i-1} \mathrm{d} t$                                         |
## 特殊函数

| 调用方法                     | 功能描述                                                                                                                                                  |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| polygamma(n,x);          | 多伽玛函数<br>伽马函数的对数的高阶导数<br>$\psi^{(n)}(z) = \frac{d^{n+1}}{dz^{n+1}} \ln \Gamma(z) = \frac{d^n}{dz^n} \psi(z)$                                          |
| betainc(a,b,x);          | 正则化不完全贝塔函数<br>$I_x(a, b) = \frac{\Gamma(a+b)}{\Gamma(a)\Gamma(b)} \int_0^x t^{a-1}(1-t)^{b-1} dt$                                                     |
| zeta(a,b);<br>a.zeta(b); | 赫尔维茨ζ函数<br>${\displaystyle \zeta (s,q)=\sum _{n=0}^{\infty }{\frac {1}{(q+n)^{s}}}.}$<br>s,q都是复数，且${\displaystyle Re(q)>0}$，${\displaystyle Re(s)>0}$ |
| a.ndtri();<br>ndtri(a);  | 正态分位数函数<br>标准正态分布的“逆”累积分布函数<br>$Φ(x) = P(Z ≤ x)$<br>$\text{ndtri}(p) = \Phi^{-1}(p)$                                                                  |
