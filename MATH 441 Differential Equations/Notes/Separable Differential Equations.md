For the case where the equation takes the form of:
$$
	\begin{align}
		M(x)+N(y)\frac{dy}{dx}=0
	\end{align}
$$
We can apply the method of separation of variables:
$$
	\boxed {
		\int _{x_{0}}^{x}M(s) \, ds +\int _{y_{0}}^{y}N(s) \, ds =0
	}
$$
---
#### Proof: 
suppose $y$ is regarded as a function of $x$ and $H_{1}'(x)=M(x)$, $H_{2}'(y)=N(y)$
	$$
		\begin{gathered}
			H_{1}'(x)+H_{2}'(y)\frac{dy}{dx}=0 \\
			H_{2}'(y)\frac{dy}{dx}=\frac{d}{dy}H_{2}(y)\frac{dy}{dx}=\frac{d}{dx}H_{2}(y)
		\end{gathered}
	$$
	let $E$ be the original equation
	$$
		\begin{aligned}
			E&=\frac{d}{dx}(H_{1}(x)+H_{2}(y))&=0 \\
			&=H_{1}(x)+H_{2}(y)=C \\
		\end{aligned}
	$$
	setting initial condition to: $y(x_{0})=y_{0}$ results in:
	$$
		C=H_{1}(x_{0})+H_{2}(y_{0})
	$$
	substituting back gives:
	$$
		\begin{gathered}
			E=H_{1}(x)+H_{2}(y)-C=0 \\
			H_{1}(x)-H_{1}(x_{0})+H_{2}(y)-H_{2}(y_{0})=0 \\
			\boxed {
				\int _{x_{0}}^{x}M(s) \, ds +\int _{y_{0}}^{y}N(s) \, ds =0
			}
		\end{gathered}
	$$
---
> Reference:
> [[Elementary Differential Equations and Boundary Value Problems 11th Edition.pdf#page=47&selection=5,0,5,32|Elementary Differential Equations and Boundary Value Problems 11th Edition, page 47]]