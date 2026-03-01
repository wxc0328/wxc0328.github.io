---
title: LaTex简要规则
date: 2026-03-01 16:28:00
tags: LaTex
categories:
  - [笔记工具, LaTex]
---

一个非常好用的工具网站[LaTeX公式编辑器](https://www.latexlive.com/##)
# 排版格式
$$
\begin{equation}
  1 + 2 = 3
\end{equation}
$$
```
\begin{排版环境}
...作用域
\end{排版环境}
```
{equation} 单行等式

$$
\begin{align*}
1 + 2 &= 3\\
1 &= 3 - 2
\end{align*}
$$
```
\begin{align*}
1 + 2 &= 3\\
1 &= 3 - 2
\end{align*}
```
{align}  多行等式对齐
\\\\  等式的行结尾
&  对齐标识位

$$
\begin{align}
f(x) &= x^2\\
g(x) &= \frac{1}{x}\\
F(x) &= \int^a_b \frac{1}{3}x^3
\end{align}
$$
```
\begin{align}
f(x) &= x^2\\
g(x) &= \frac{1}{x}\\
F(x) &= \int^a_b \frac{1}{3}x^3
\end{align}
```

# 阶数
```
X^n //阶数
```
$$
\begin{equation}
X^n
\end{equation}
$$

# 分式
```
\frac{numerator}{denominator} //分数表达,{分子}{分母}
```
$$
\begin{equation}
\frac{numerator}{denominator}
\end{equation}
$$

# 积分
```
\int //积分
\int^{up}_{down} //定积分, ^{up}上界, _{down}下界,可以换序，可以部分声明
\int_{down}^{up} //定积分, ^{up}上界, _{down}下界,等价
\int^a_b //定积分，单字符可忽略{},^a上界, _b下界
\int_D //域积分
\iint //双重积分
\iiint //三重积分
\oint //曲面积分
```
$$
\begin{align}
{不定积分:}
& \int \\
\\
{定积分:}
& \int^{up}_{down}
or
\int_{down}^{up}
or
\int^a_b\\
{域积分:}
& \int_D \\
{双重积分:}
& \iint\\
{三重积分:}
& \iiint \\
{曲面积分:}
& \oint
\end{align}
$$

# 根号
```
\sqrt{x} //根号
\sqrt[n]{x} //n阶根号
```
$$
\begin{align}
\sqrt{x} \\
\sqrt[n]{x}
\end{align}
$$
# 对数
```
\log{X} //对数
\log_a{b} //以a为底
\ln{X} //以e为底
\lg{X} //以2为底
```
$$
\begin{align}
\log{X}
\\
\log_a{b}\\
\ln{X}\\
\lg{X}
\end{align}
$$

# 矩阵
```
\begin{matrix} //矩阵排版格式
1 & 0\\
0 & 1
\end{matrix}
```
$$
\begin{matrix}
1 & 0 & 0\\
0 & 1 & 0\\
0 & 0 & 1
\end{matrix}
$$
```
\left[ //左侧添加'['
\begin{matrix}
1 & 0\\
0 & 1
\end{matrix}
\right] //右侧添加']'
```
$$
\left[
\begin{matrix}
1 & 0\\
0 & 1
\end{matrix}
\right]
$$

# 三角函数
```
\sin{X}
\cos{X}
\tan{X}
\cot{X}
\sec{X}
\csc{X}
```
$$
\begin{align}
\sin{X}\\
\cos{X}\\
\tan{X}\\
\cot{X}\\
\sec{X}\\
\csc{X}
\end{align}
$$

# 希腊字母
| 字母      |             | LaTex语法           |     |
| ------- | ----------- | ----------------- | --- |
| Alpha   | α A         | \alpha A          |     |
| Beta    | β B         | \beta B           |     |
| Gamma   | γ Γ         | \gamma \Gamma     |     |
| Delta   | δ Δ         | \delta \Delta     |     |
| Epsilon | ϵ E         | \epsilon E        |     |
| Zeta    | ζ Z         | \zeta Z           |     |
| Eta     | η E         | \eta E            |     |
| Theta   | θ Θ         | \theta \Theta     |     |
| Iota    | ι I         | \iota I           |     |
| Kappa   | κ K         | \kappa K          |     |
| Lambda  | λ Λ         | \lambda \Lambda   |     |
| Mu      | μ M         | \mu M             |     |
| Nu      | ν N         | \nu N             |     |
| Omicron | ο O         | \omicron O        |     |
| Pi      | π Π         | \pi \Pi           |     |
| Rho     | ρ R         | \rho R            |     |
| Sigma   | σ Σ         | \sigma \Sigma     |     |
| Tau     | τ T         | \tau T            |     |
| Upsilon | υ Υ         | \upsilon \Upsilon |     |
| Phi     | ϕ Φ         | \phi \Phi         |     |
| Chi     | χ X         | \chi X            |     |
| Psi     | ψ Ψ         | \psi \Psi         |     |
| Omega   | ω Ω         | \omega \Omega     |     |
| Phi     | φ           | \varphi           |     |
| Xi      | $\xi \ \Xi$ | \xi               |     |
$$
\begin{align}
\alpha \ A\\
\beta \ B \\
\gamma \ \Gamma\\
\delta \ \Delta\\
\epsilon \ E\\
\zeta \ Z\\
\eta \ E\\
\theta \ \Theta\\
\iota \ I\\
\kappa \ K\\
\lambda \ \Lambda\\
\mu \ M\\
\nu \ N\\
\omicron \ O\\
\pi \ \Pi\\ 
\rho \ R\\
\sigma \ \Sigma\\
\tau \ T\\
\upsilon \ \Upsilon\\
\phi \ \Phi \ \varphi \\
\chi \ X\\
\psi \ \Psi\\
\omega \ \Omega\\
\xi \ \Xi\\
\end{align}
$$

# 不等号
```
\neq //不等号
```
$$
\begin{equation}
\neq
\end{equation}
$$

# 加减符
```
\pm //plus, minus
\mp
```
$$
\begin{align}
\pm \\
\mp
\end{align}
$$

# 偏导数
```
\partial //偏导数符号
```
$$
\begin{align}
\partial \\
\end{align}
$$

# 无穷符号
```
\infty
```
$$
\begin{align}
\infty\\
+\infty\\
-\infty
\end{align}
$$

# 数列和
```
\sum_{n=1}^{\infty}
```
$$
\begin{equation}
\sum_{n=1}^{\infty}
\end{equation}
$$

# 极限
```
\lim_{n\to \infty}
```
$$
\begin{align}
\lim_{n \to \infty}\\
\lim_{n \to 0}
\end{align}
$$

# 箭头
```
\leftarrow &\ \rightarrow\\
\Leftarrow &\ \Rightarrow\\
\Longleftarrow &\ \Longrightarrow \\
```
$$
\begin{align}
\leftarrow &\ \rightarrow\\
\Leftarrow &\ \Rightarrow\\
\Longleftarrow &\ \Longrightarrow \\
\end{align} 
$$

# 渐近符
```
\sim
```
$$
\begin{align}
\sim
\end{align}
$$

# 分段函数
```
\begin{equation}
	T(m)=
	\left\{
	\begin{array}{ll}
	1, & \mbox{\emph{succeed}}\\
	1+T(m'), & \mbox{\emph{failed}}\\
	\end{array}
	\right.
\end{equation}
```
$$
\begin{equation}
	T(m)=
	\left\{
	\begin{array}{ll}
	1, & \mbox{\emph{succeed}}\\
	1+T(m'), & \mbox{\emph{failed}}\\
	\end{array}
	\right.
\end{equation}
$$

# 交集、并集
```
\begin{align}
A \cap B
A \cup B
\end{align}
```
$$
\begin{align}
A \cap B \\
A \cup B
\end{align}
$$

# 边界项
```
\biggl. uv \biggr|_a^b
```
$$
\biggl. uv \biggr|_a^b
$$