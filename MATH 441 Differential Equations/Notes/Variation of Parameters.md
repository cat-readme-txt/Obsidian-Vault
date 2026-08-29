#### First Order Differential Equations

Consider the first-order linear nonhomogeneous equation
$$
	\begin{gather}
		y'+p(x)y=q(x)
	\end{gather}
$$
It's associated homogeneous equation is
$$
	\begin{gather}
		y'+p(x)y=0
	\end{gather}
$$
Solve the homogeneous equation:
$$
	\begin{gather}
		\frac{dy}{y}=-p(x)dx \\
		y_{h}=Ce^{-\int p(x) \, dx }
	\end{gather}
$$

Let
$$
	\begin{gather}
		y_{1}=e^{-\int p(x) \, dx }
	\end{gather}
$$
Now replace $C$ by $u(x)$:
$$
	\begin{gather}
		y=u(x)y_{1}(x)
	\end{gather}
$$
Then
$$
	\begin{gather}
		y'=u'y_{1}+uy_{1}' \\
	\end{gather}
$$
Substitute into the original equation:
$$
	\begin{gather}
		u'y_{1}+uy_{1}'+p(x)uy_{1}=q(x)
	\end{gather}
$$
Since $y_{1}$ solves the homogeneous equation ($y_1'+p(x)y_1=0$), substituting back while grouping terms involving $u$ gives:
$$
	\begin{gather}
		u'y_{1}+u(y_{1}'+p(x)y_{1})=q(x) \\
		u'y_{1}+u(0)=q(x) \\
		u'y_{1}=q(x)
	\end{gather}
$$

Therefore,
$$
	\begin{gathered}
		u'=\frac{q(x)}{y_{1}}=q(x)e^{\int p(x) \, dx } \\
		u=\int q(x)e^{\int p(x) \, dx } \, dx 
	\end{gathered}
$$

The **<u>general solution</u>** is therefore:
$$
	\boxed {
		\begin{gathered}
			y=e^{-\int p(x) \, dx }\left[ C+\int q(x)e^{\int p(x) \, dx } \, dx  \right]
		\end{gathered}
	}
$$
<br>
<br>

$$
\text{Sample Question:}\quad y'-2y=xe^{2x}
$$

Solution:
homogeneous equation:
$$
	y'-2y=0
$$
so
$$
	y_{h}=Ce^{2x}
$$
Set
$$
	y=u(x)e^{2x}
$$
Then
$$
	y'=u'e^{2x}+2ue^{2x}
$$
Substituting
$$
	u'e^{2x}+2ue^{2x}-2ue^{2x}=xe^2x
$$
Therefore,
$$
	u'=x
$$
Integrating,
$$
	u=\frac{x^2}{2}+C
$$
Hence
$$
	\boxed{
		y=e^{2x}(C+\frac{x^2}{2})
	}
$$