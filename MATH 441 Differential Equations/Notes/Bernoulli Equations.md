In general, a Bernoulli equation has the form:
$$
	y'+p(x)y=q(x)y^\alpha
$$
where $\alpha\neq 0,1$ (If $\alpha$ is 0 or 1, this would be a linear equation)

We can use the substitution $v(x)=y^{1-\alpha}$ and the equation becomes <u>linear</u> for v(x) and we can use an [[Integrating Factor]].

---
Sample question:
$$
	y'=\frac{y}{x}+\sqrt{y}
$$
Solution:
$$
	\begin{gathered}
		\rightarrow y'-\frac{1}{x}y=y^{\frac{1}{2}} \\
		\alpha=\frac{1}{2}, \quad v(x)=y^{1-\frac{1}{2}}=y^{\frac{1}{2}}; \quad y=v^2; \quad y'=2vv' \\
	\end{gathered}
$$
substituting back:
$$
	2vv'-\frac{1}{x}v^2=v\quad \rightarrow \quad v'-\frac{1}{2x}v=\frac{1}{2}
$$
(assuming $v\ne0$. If $v=0$ then $y=0$ and you can verify this is another solution)


Integrating factor:
$$
	\mu(x)=e^{-\int \frac{1}{2x} \, dx }=e^{-\frac{1}{2}\ln(x)}=x^{1\frac{1}{2}}=\frac{1}{\sqrt{ x }}
$$
substituting back:
$$
	\begin{gathered}
		\frac{v'}{\sqrt{ x }}-\frac{1}{2x\sqrt{ x }}v=\frac{1}{2\sqrt{ x }}\quad \rightarrow\quad \left( \frac{1}{\sqrt{ x }} \right)'=\frac{1}{2\sqrt{ x }}\quad \rightarrow\quad \frac{1}{\sqrt{ x }}v=\sqrt{ x }+C \\
		v=x+C\sqrt{ x }=\sqrt{ y }\quad \rightarrow\quad \boxed{y(x)=(x+C\sqrt{ x })^2}
	\end{gathered}
$$
to which we need to add the singular solution $\boxed{y(x)=0}$.