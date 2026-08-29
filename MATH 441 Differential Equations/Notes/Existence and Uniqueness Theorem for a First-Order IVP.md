> [!info] Theorem
> Consider the IVP
> 
> $$  
> 	\begin{cases}
> 		y'=f(x,y), \\
> 		y(0)=0.
> 	\end{cases}
> $$
> 
> Assume there exists a rectangle
> 
> $$
> 	R=\{(x,y):|x|<a,\ |y|<b\}
> $$
> 
> such that $f$ and $f_y$ are continuous on $R$.
> 
> Define
> 
> $$
> 	M=\max_R|f(x,y)|
> $$
> 
> and
> 
> $$
> 	T=\max_R|f_y(x,y)|.
> $$
> 
> Choose
> 
> $$
> 	h=\min\{a,\frac{b}{M}\}.
> $$
> 
> > [!question]- Where did $\frac{b}{M}$ Come From?
> > It is chosen to make sure the solution remains inside the vertical bounds
> > $$
> > 	|y|<b.
> > $$
> > Since
> > $$
> > 	M=\max_R|f(x,y)|,
> > $$
> > we have
> > $$
> > 	|y'|\leq M.
> > $$
> > Starting from $y(0)=0$, the change in $y$ satisfies
> > $$
> > 	|y(x)|\leq M|x|.
> > $$
> > To ensure $|y(x)|\leq b$, require
> > $$
> > 	M|x|\leq b.
> > $$
> > Therefore,
> > $$
> > 	|x|\leq\frac{b}{M}.
> > $$
> > Thus, $\frac{b}{M}$ is the largest horizontal distance allowed before the solution could leave the rectangle through $y=\pm b$.
> 
> Then the IVP has exactly one solution on
> 
> $$
> 	|x|<h.
> $$

---

# Proof
### Integral Equation

Integrating the differential equation from $0$ to $x$ gives

$$
	\begin{gathered}
		\int_0^x y'(s)ds=\int_0^x f(s,y(s))\,ds, \\
		y(x)-y(0)=\int_0^x f(s,y(s))\,ds.
	\end{gathered}
$$

Since $y(0)=0$,

$$
	y(x)=\int_0^x f(s,y(s))\,ds.
$$

<br>

### Picard Iteration

Define

$$
	y_0(x)=0
$$

and

$$
	y_n(x)=\int_0^x f\left(s,y_{n-1}(s)\right)\,ds.
$$

<br>

### Existence of Every Picard Iterate

Assume $y_{n-1}(s)$ remains inside $R$. Then

$$  
	\begin{aligned}  
		|y_n(x)|&=\left|\int_0^x f\left(s,y_{n-1}(s)\right),ds\right| \\
		&\leq
		\int_0^{|x|} \left|f\left(s,y_{n-1}(s)\right)\right|\,ds \\
		&\leq
		\int_0^{|x|} M\,ds \\
		&=  
		M|x|.  
	\end{aligned}  
$$

For $|x|\leq h$,

$$
	|y_n(x)|\leq Mh.
$$

Since

$$
	h\leq\frac{b}{M},
$$

we have

$$
	Mh\leq b.
$$

Therefore,

$$
	|y_n(x)|\leq b.
$$

Also,

$$
	|x|\leq h\leq a.
$$

Thus,

$$
	\left(s,y_n(s)\right)\in R
$$

for every $n$, so each Picard iterate exists and is continuous on $|x|\leq h$.

<br>

### Mean Value Theorem Estimate

For fixed $s$, apply the Mean Value Theorem to $f(s,y)$ between $y_{n-1}(s)$ and $y_{n-2}(s)$.

There exists $\xi_s$ between these two values such that

$$
	f\left(s,y_{n-1}(s)\right)-f\left(s,y_{n-2}(s)\right)=f_y(s,\xi_s)\left[y_{n-1}(s)-y_{n-2}(s)\right].
$$

Since $|f_y|\leq T$ in $R$,

$$  
	\left|f\left(s,y_{n-1}(s)\right)-f\left(s,y_{n-2}(s)\right)\right|\leq T|y_{n-1}(s)-y_{n-2}(s)|.
$$

Therefore,

$$
	\begin{aligned}
		|y_n(x)-y_{n-1}(x)|	&=\left|\int_0^x \left[f\left(s,y_{n-1}(s)\right)-f\left(s,y_{n-2}(s)\right)\right]\,ds\right| \\
		&\leq
		T\int_0^{|x|} |y_{n-1}(s)-y_{n-2}(s)|\,ds.
	\end{aligned}
$$

<br>

<br>

### Estimate of Consecutive Iterates

We prove by induction that

$$
	|y_n(x)-y_{n-1}(x)|\leq\frac{MT^{n-1}|x|^n}{n!}.
$$

For $n=1$,

$$
	\begin{aligned}
		|y_1(x)-y_0(x)|&=\left|\int_0^x f(s,0),ds\right| \\
		&\leq
		\int_0^{|x|} |f(s,0)|\,ds \\
		&\leq
		\int_0^{|x|} M\,ds \\
		&=
		M|x| \\
		&=
		\frac{MT^0|x|}{1!}.
	\end{aligned}
$$

Assume

$$
	|y_{n-1}(s)-y_{n-2}(s)|\leq\frac{MT^{n-2}|s|^{n-1}}{(n-1)!}.
$$

Then

$$
\begin{aligned}
	|y_n(x)-y_{n-1}(x)|	&\leq T\int_0^{|x|}	|y_{n-1}(s)-y_{n-2}(s)|\,ds \\
	&\leq
	T\int_0^{|x|} \frac{MT^{n-2}s^{n-1}}{(n-1)!}\,ds \\
	&=
	\frac{MT^{n-1}}{(n-1)!}\int_0^{|x|} s^{n-1}\,ds \\
	&=
	\frac{MT^{n-1}}{(n-1)!}\frac{|x|^n}{n} \\
	&=
	\frac{MT^{n-1}|x|^n}{n!}.
\end{aligned}
$$

Therefore,

$$
\boxed{
	|y_n(x)-y_{n-1}(x)|\leq\frac{MT^{n-1}|x|^n}{n!}
}
$$

for every $n\geq1$.

<br>

### Uniform Convergence

Write $y_n$ as a telescoping sum:

$$
\begin{aligned}
	y_n(x)&=y_0(x)+\left[y_1(x)-y_0(x)\right]+\cdots+\left[y_n(x)-y_{n-1}(x)\right] \\
	&=
	y_0(x)+	\sum_{k=0}^{n-1} \left[y_{k+1}(x)-y_k(x)\right].
\end{aligned}
$$

For $|x|\leq h$,

$$
	|y_{k+1}(x)-y_k(x)|\leq\frac{MT^kh^{k+1}}{(k+1)!}.
$$

Consider the numerical series

$$
\sum_{k=0}^{\infty} \frac{MT^kh^{k+1}}{(k+1)!}.
$$

For $T>0$,

$$
	\begin{aligned}
		\sum_{k=0}^{\infty} \frac{MT^kh^{k+1}}{(k+1)!}&=\frac{M}{T}\sum_{k=0}^{\infty} \frac{(Th)^{k+1}}{(k+1)!} \\
		&=
		\frac{M}{T}\sum_{j=1}^{\infty} \frac{(Th)^j}{j!} \\
		&=
		\frac{M}{T}\left(e^{Th}-1\right).
	\end{aligned}
$$

Thus, the numerical series converges.

If $T=0$, then

$$
	|y_n-y_{n-1}|=0
$$

for every $n\geq2$, so convergence is immediate.

By the Weierstrass $M$-test,

$$
	\sum_{k=0}^{\infty} \left[y_{k+1}(x)-y_k(x)\right]
$$

converges uniformly on $|x|\leq h$.

Therefore, the sequence ${y_n}$ converges uniformly to a function $y$:

$$
	\lim_{n\to\infty}y_n(x)=y(x).
$$

<br>

### The Limit Is a Solution

From the Picard iteration,

$$
	y_n(x)=\int_0^x f\left(s,y_{n-1}(s)\right)\,ds.
$$

Taking the limit as $n\to\infty$,

$$
	\lim_{n\to\infty}y_n(x)=\lim_{n\to\infty}\int_0^x f\left(s,y_{n-1}(s)\right)\,ds.
$$

Since $y_n\to y$ uniformly and $f$ is continuous,

$$
	f\left(s,y_{n-1}(s)\right)\to f(s,y(s))
$$

uniformly. Therefore, the limit may pass through the integral:

$$
	y(x)=\int_0^x f(s,y(s))\,ds.
$$

At $x=0$,

$$
	y(0)=\int_0^0 f(s,y(s))\,ds=0.
$$

By the Fundamental Theorem of Calculus,

$$
	y'(x)=f(x,y(x)).
$$

Thus, $y$ satisfies

$$
	\begin{cases}
		y'=f(x,y), \\
		y(0)=0.
	\end{cases}
$$

Therefore, a solution exists on $|x|\leq h$.

<br>

### Uniqueness for ($x\geq0$)

Suppose $\phi$ and $\psi$ are two solutions:

$$
	\phi(x)=\int_0^x f(s,\phi(s))\,ds
$$

and

$$
	\psi(x)=\int_0^x f(s,\psi(s))\,ds.
$$

Then

$$
	\begin{aligned}
		|\phi(x)-\psi(x)|&=\left|\int_0^x \left[f(s,\phi(s))-f(s,\psi(s))\right]ds\right| \\
		&\leq
		\int_0^x |f(s,\phi(s))-f(s,\psi(s))|\,ds \\
		&\leq
		T\int_0^x |\phi(s)-\psi(s)|\,ds.
	\end{aligned}  
$$

Define

$$
	U(x)=\int_0^x |\phi(s)-\psi(s)|\,ds.
$$

Then

$$
	\begin{gathered}
		U(0)=0, \\
		U(x)\geq0,
	\end{gathered}
$$

and

$$
	U'(x)=|\phi(x)-\psi(x)|.
$$

Therefore,

$$
	U'(x)\leq TU(x),
$$

so

$$
	U'(x)-TU(x)\leq0.
$$

Multiply by the positive integrating factor $e^{-Tx}$:

$$
	e^{-Tx}U'(x)-Te^{-Tx}U(x)\leq0.
$$

Thus,

$$
	\left(e^{-Tx}U(x)\right)'\leq0.
$$

Since

$$
	e^{-T\cdot0}U(0)=0,
$$

the function $e^{-Tx}U(x)$ is nonincreasing from $0$. Therefore,

$$
	e^{-Tx}U(x)\leq0.
$$

Since $e^{-Tx}>0$,

$$
	U(x)\leq0.
$$

But $U(x)\geq0$, so

$$
	U(x)=0.
$$

Therefore,

$$
	\int_0^x |\phi(s)-\psi(s)|\,ds=0.
$$

Since the integrand is continuous and nonnegative,

$$
	|\phi(s)-\psi(s)|=0
$$

for every $s\in[0,x]$. Hence,

$$
	\phi(x)=\psi(x).
$$

<br>

### Uniqueness for ($x\leq0$)

For $x\leq0$, define

$$
	U(x)=\int_x^0 |\phi(s)-\psi(s)|\,ds.
$$

Then

$$
	\begin{gathered}
		U(0)=0, \\
		U(x)\geq0,
	\end{gathered}
$$

and

$$
	U'(x)=-|\phi(x)-\psi(x)|.
$$

From

$$
|\phi(x)-\psi(x)|\leq T\int_x^0 |\phi(s)-\psi(s)|\,ds,
$$

we obtain

$$
	-U'(x)\leq TU(x).
$$

Therefore,

$$
	U'(x)+TU(x)\geq0.
$$

Multiplying by $e^{Tx}>0$ gives

$$
	\left(e^{Tx}U(x)\right)'\geq0.
$$

For $x\leq0$,

$$
	e^{Tx}U(x)\leq e^0U(0)=0.
$$

Since $e^{Tx}U(x)\geq0$,

$$
	U(x)=0.
$$

Hence,

$$
	\phi(x)=\psi(x).
$$

Therefore, the solution is unique on

$$
	|x|\leq h.
$$

---

# Conclusion

If $f$ and $f_y$ are continuous in

$$
	R=\{(x,y):|x|<a,\ |y|<b\},
$$

then, with

$$
	h=\min\{a,\frac{b}{M}\},\qquad  M=\max_R|f|,
$$

the IVP

$$
	\begin{cases}
		y'=f(x,y), \\
		y(0)=0
	\end{cases}
$$

has exactly one solution on

$$
	|x|<h.
$$