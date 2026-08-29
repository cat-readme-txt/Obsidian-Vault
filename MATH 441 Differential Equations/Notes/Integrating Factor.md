For equations in the following format:
$$
	y'+p(t)y=q(t)
$$
We can apply the method of Integrating Factors:
$$
	\begin{align}
		\mu(t)&=e^{\int p(t)dt} \\
		\mu(t)y'+\mu(t)p(t)y&=\mu(t)q(t) \\
		\mu(t)y&=\int \mu(t)q(t)+C \\
		y&=\frac{1}{\mu(t)}\int \mu(t)q(t)+\frac{C}{\mu(t)}
	\end{align}
$$
> Reference:
> [[Elementary Differential Equations and Boundary Value Problems 11th Edition.pdf#page=42&selection=132,0,132,73|Elementary Differential Equations and Boundary Value Problems 11th Edition, page 42]]
---
#### Special Example:
$$
	\begin{align}
		2y'+ty&=2 \\
		y(0)&=1
	\end{align}
$$
Solution:
$$
	\begin{gather}
		\begin{aligned}
			y'+\frac{1}{2}ty&=1 \\
			\mu(t)&=\exp\left( \int \frac{1}{2}t \, dt  \right) \\
			\mu(t)&=\exp\left( \frac{1}{4}t^4 \right) \\
			e^\left( \frac{1}{4}t^2 \right)y&=\int e^\left( \frac{1}{4}t^2 \right) \, dt +C_{1} \\
		\end{aligned}
	\end{gather}
$$
> [!note] Notice 
> $\int e^\left( \frac{1}{4}t^2 \right)dt$ isn't integrable
$$
	\begin{gather}
		\begin{aligned}
			\int e^\left( \frac{1}{4}t^2 \right) \, dt &=\int _{0}^{t}e^{ \frac{1}{4}s^2 } \, ds +C_{2} \\
			e^\left( \frac{1}{4}t^2 \right)y&=\int _{0}^{t}e^\left( \frac{1}{4}s^2 \right) \, ds +C_{3} \\
			y&=e^\left( -\frac{1}{4}t^2 \right)\left( \int _{0}^{t}e^\left( \frac{1}{4}s^2 \right) \, ds  \right) +C_{3}
		\end{aligned}
	\end{gather}
$$
> Reference:
> [[Elementary Differential Equations and Boundary Value Problems 11th Edition.pdf#page=44&selection=64,0,64,9|Elementary Differential Equations and Boundary Value Problems 11th Edition, page 44]]