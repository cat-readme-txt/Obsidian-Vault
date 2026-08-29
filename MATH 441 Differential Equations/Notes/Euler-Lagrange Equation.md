> [!note] Definition
> A **functional** assigns a number to an entire function. For a first-order variational problem, write
>
> $$
> 	I[y]=\int^{x_2}_{x_1} F(x,y(x),y'(x))\,dx.
> $$
>
> A function $y(x)$ is **stationary** when the first-order change of the functional is zero for every admissible variation:
>
> $$
> 	\delta I=0.
> $$

> [!info] Theorem
> Suppose $y(x)$ makes
>
> $$
> 	I[y]=\int^{x_2}_{x_1} F(x,y,y')\,dx
> $$
>
> stationary among sufficiently smooth functions satisfying the fixed endpoint conditions
>
> $$
> 	y(x_1)=y_1,\qquad y(x_2)=y_2.
> $$
>
> Then $y(x)$ must satisfy the Euler–Lagrange equation
>
> $$
> 	\boxed{\frac{\partial F}{\partial y}-\frac{d}{dx}(\frac{\partial F}{\partial y'})=0}.
> $$

---

# Proof

### Fixed-Endpoint Variation

Choose an arbitrary sufficiently smooth function $\eta(x)$ that vanishes at the endpoints:

$$
	\eta(x_1)=0,\qquad \eta(x_2)=0.
$$

Construct a nearby function

$$
	\bar y(x)=y(x)+\varepsilon\eta(x),
$$

where $\varepsilon$ controls the size of the variation. The endpoint conditions remain fixed because

$$
	\begin{gathered}
		\bar y(x_1)=y(x_1)+\varepsilon\eta(x_1)=y(x_1), \\
		\bar y(x_2)=y(x_2)+\varepsilon\eta(x_2)=y(x_2).
	\end{gathered}
$$

Differentiating with respect to $x$ gives

$$
	\bar y'(x)=y'(x)+\varepsilon\eta'(x).
$$

Define an ordinary function of $\varepsilon$ by

$$
	\Phi(\varepsilon)=I[\bar y]=\int^{x_2}_{x_1} F(x,y+\varepsilon\eta,y'+\varepsilon\eta')\,dx.
$$

Since $y$ corresponds to $\varepsilon=0$ and is stationary,

$$
	\Phi'(0)=0.
$$

<br>

### First Variation

Differentiate $\Phi(\varepsilon)$ using differentiation under the integral sign and the multivariable chain rule:

$$
	\begin{aligned}
		\Phi'(\varepsilon)&=\int^{x_2}_{x_1} \frac{\partial}{\partial\varepsilon}F(x,\bar y,\bar y')\,dx \\
		&=\int^{x_2}_{x_1} [\frac{\partial F}{\partial\bar y}\frac{\partial\bar y}{\partial\varepsilon}+\frac{\partial F}{\partial\bar y'}\frac{\partial\bar y'}{\partial\varepsilon}]\,dx.
	\end{aligned}
$$

Because

$$
	\frac{\partial\bar y}{\partial\varepsilon}=\eta,\qquad \frac{\partial\bar y'}{\partial\varepsilon}=\eta',
$$

we obtain

$$
	\Phi'(\varepsilon)=\int^{x_2}_{x_1} [\frac{\partial F}{\partial\bar y}\eta+\frac{\partial F}{\partial\bar y'}\eta']\,dx.
$$

At $\varepsilon=0$, we have $\bar y=y$ and $\bar y'=y'$, so

$$
	\delta I=\Phi'(0)=\int^{x_2}_{x_1} [\frac{\partial F}{\partial y}\eta+\frac{\partial F}{\partial y'}\eta']\,dx=0.
$$

> [!question]- Why may the derivative pass through the integral?
> This step is valid under the standard regularity assumptions used in the derivation: $F$ and the required partial derivatives are continuous on the relevant region, and the interval $[x_1,x_2]$ is fixed.

<br>

### Integration by Parts

The term containing $\eta'$ must be rewritten in terms of $\eta$. Apply integration by parts with

$$
	u=\frac{\partial F}{\partial y'},\qquad dv=\eta'\,dx.
$$

Then

$$
	v=\eta,\qquad du=\frac{d}{dx}(\frac{\partial F}{\partial y'})\,dx,
$$

and therefore

$$
	\int^{x_2}_{x_1} \frac{\partial F}{\partial y'}\eta'\,dx=[\frac{\partial F}{\partial y'}\eta]^{x_2}_{x_1}-\int^{x_2}_{x_1} \frac{d}{dx}(\frac{\partial F}{\partial y'})\eta\,dx.
$$

Since $\eta(x_1)=\eta(x_2)=0$,

$$
	[\frac{\partial F}{\partial y'}\eta]^{x_2}_{x_1}=0.
$$

Thus,

$$
	\int^{x_2}_{x_1} \frac{\partial F}{\partial y'}\eta'\,dx=-\int^{x_2}_{x_1} \frac{d}{dx}(\frac{\partial F}{\partial y'})\eta\,dx.
$$

Substitute this into the first variation:

$$
	\begin{aligned}
		0&=\int^{x_2}_{x_1} \frac{\partial F}{\partial y}\eta\,dx-\int^{x_2}_{x_1} \frac{d}{dx}(\frac{\partial F}{\partial y'})\eta\,dx \\
		&=\int^{x_2}_{x_1} [\frac{\partial F}{\partial y}-\frac{d}{dx}(\frac{\partial F}{\partial y'})]\eta(x)\,dx.
	\end{aligned}
$$

See [[Integration by Parts]].

<br>

### The Euler–Lagrange Equation

The function $\eta(x)$ is arbitrary except for the endpoint conditions. By the [[Fundamental Lemma of the Calculus of Variations]],

$$
	\int^{x_2}_{x_1} G(x)\eta(x)\,dx=0
$$

for every admissible $\eta$ implies

$$
	G(x)=0.
$$

Here,

$$
	G(x)=\frac{\partial F}{\partial y}-\frac{d}{dx}(\frac{\partial F}{\partial y'}).
$$

Therefore,

$$
	\boxed{\frac{\partial F}{\partial y}-\frac{d}{dx}(\frac{\partial F}{\partial y'})=0}.
$$

> [!question]- Why must the integrand be zero?
> If $G$ were positive or negative on some small subinterval, an admissible variation $\eta$ could be chosen with the same sign and supported inside that subinterval. The integral would then be nonzero, contradicting the condition that it vanishes for every $\eta$.

Lecture reference: [[Lecture 07 Slides.pdf#page=4|Lecture 07 Slides, pp. 4–6]].

---

# Useful Special Case

### Beltrami Identity

Suppose the integrand has no explicit dependence on $x$:

$$
	\frac{\partial F}{\partial x}=0.
$$

The total derivative of $F(x,y,y')$ is

$$
	\frac{dF}{dx}=\frac{\partial F}{\partial y}y'+\frac{\partial F}{\partial y'}y''.
$$

The Euler–Lagrange equation gives

$$
	\frac{\partial F}{\partial y}=\frac{d}{dx}(\frac{\partial F}{\partial y'}).
$$

Now differentiate $F-y'\frac{\partial F}{\partial y'}$:

$$
	\begin{aligned}
		\frac{d}{dx}[F-y'\frac{\partial F}{\partial y'}]&=\frac{dF}{dx}-y''\frac{\partial F}{\partial y'}-y'\frac{d}{dx}(\frac{\partial F}{\partial y'}) \\
		&=\frac{\partial F}{\partial y}y'-y'\frac{d}{dx}(\frac{\partial F}{\partial y'}) \\
		&=0.
	\end{aligned}
$$

Therefore,

$$
	\boxed{F-y'\frac{\partial F}{\partial y'}=C}.
$$

This is the **Beltrami identity**. The equivalent form

$$
	y'\frac{\partial F}{\partial y'}-F=C
$$

is obtained by absorbing a minus sign into the constant.

Lecture reference: [[Lecture 07 Slides.pdf#page=7|Lecture 07 Slides, pp. 7–9]].

---

# Conclusion

For a stationary path with fixed endpoints,

$$
	\boxed{\frac{\partial F}{\partial y}-\frac{d}{dx}(\frac{\partial F}{\partial y'})=0}.
$$

When $F$ does not explicitly depend on $x$, this reduces once to

$$
	\boxed{F-y'\frac{\partial F}{\partial y'}=C}.
$$