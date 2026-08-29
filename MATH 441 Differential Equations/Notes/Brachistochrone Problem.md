> [!info] Theorem
> A frictionless particle moving under uniform gravity travels between two fixed points in minimum time along an arc of a cycloid.
>
> Set the starting point at the origin, take $x$ horizontally to the right, and take $y$ positively downward:
>
> $$
> 	P_1=(0,0),\qquad P_2=(x_2,y_2),\qquad x_2>0,\qquad y_2>0.
> $$

---

# Setup

### Speed from Conservation of Energy

Because the system is frictionless, the loss of gravitational potential energy equals the gain in kinetic energy:

$$
	\frac{1}{2}Mv^2=Mgy.
$$

The mass cancels, giving

$$
	v=\sqrt{2gy}.
$$

> [!question]- Why is $y$ positive downward?
> The particle has fallen a vertical distance $y$ below $P_1$, so its loss of potential energy is $Mgy$. This convention keeps $y\geq0$ and makes the speed formula real along the path.

<br>

### Travel-Time Functional

An arc-length element is

$$
	ds=\sqrt{dx^2+dy^2}=\sqrt{1+(y')^2}\,dx.
$$

Since $dt=\frac{ds}{v}$, the total travel time is

$$
	\begin{aligned}
		T[y]&=\int^{P_2}_{P_1} \frac{ds}{v} \\
		&=\int^{x_2}_{0} \frac{\sqrt{1+(y')^2}}{\sqrt{2gy}}\,dx.
	\end{aligned}
$$

Define the Lagrangian

$$
	L(y,y')=\frac{\sqrt{1+(y')^2}}{\sqrt{2gy}}.
$$

The minimizing path must satisfy the result derived in [[Euler-Lagrange Equation#The Euler–Lagrange Equation|the Euler–Lagrange equation]].

Lecture reference: [[Lecture 07 Slides.pdf#page=2|Lecture 07 Slides, pp. 2–6]].

---

# Solution

### Apply the Beltrami Identity

The Lagrangian does not explicitly depend on $x$, so use [[Euler-Lagrange Equation#Beltrami Identity|the Beltrami identity]]:

$$
	L-y'\frac{\partial L}{\partial y'}=c.
$$

First calculate

$$
	\frac{\partial L}{\partial y'}=\frac{y'}{\sqrt{2gy}\sqrt{1+(y')^2}}.
$$

Then

$$
	\begin{aligned}
		L-y'\frac{\partial L}{\partial y'}&=\frac{\sqrt{1+(y')^2}}{\sqrt{2gy}}-\frac{(y')^2}{\sqrt{2gy}\sqrt{1+(y')^2}} \\
		&=\frac{1+(y')^2-(y')^2}{\sqrt{2gy}\sqrt{1+(y')^2}} \\
		&=\frac{1}{\sqrt{2gy}\sqrt{1+(y')^2}}=c.
	\end{aligned}
$$

Square both sides:

$$
	\frac{1}{2gy[1+(y')^2]}=c^2.
$$

Rearrange and define a positive constant $k$ by

$$
	y[1+(y')^2]=\frac{1}{2gc^2}=k^2.
$$

Therefore,

$$
	(y')^2=\frac{k^2}{y}-1.
$$

For the downward-moving branch used in the lecture,

$$
	y'=\sqrt{\frac{k^2}{y}-1}.
$$

<br>

### Trigonometric Substitution

Use

$$
	y=k^2\sin^2(t).
$$

Then

$$
	dy=2k^2\sin(t)\cos(t)\,dt
$$

and

$$
	\sqrt{\frac{k^2}{y}-1}=\sqrt{\frac{1}{\sin^2(t)}-1}=\frac{\cos(t)}{\sin(t)}.
$$

Since $y'=\frac{dy}{dx}$,

$$
	\begin{aligned}
		\frac{dy}{dx}&=\frac{\cos(t)}{\sin(t)} \\
		dx&=\frac{\sin(t)}{\cos(t)}\,dy \\
		&=2k^2\sin^2(t)\,dt.
	\end{aligned}
$$

Using

$$
	\sin^2(t)=\frac{1-\cos(2t)}{2},
$$

integrate:

$$
	\begin{aligned}
		x&=\int 2k^2\sin^2(t)\,dt \\
		&=k^2\int [1-\cos(2t)]\,dt \\
		&=k^2[t-\frac{\sin(2t)}{2}]+A.
	\end{aligned}
$$

Also,

$$
	y=k^2\sin^2(t)=\frac{k^2}{2}[1-\cos(2t)].
$$

Set $P_1$ at the origin and choose $t=0$ there. Since $x(0)=0$, we obtain $A=0$.

Now define

$$
	\theta=2t.
$$

The path becomes

$$
	\begin{gathered}
		x(\theta)=\frac{k^2}{2}[\theta-\sin(\theta)], \\
		y(\theta)=\frac{k^2}{2}[1-\cos(\theta)].
	\end{gathered}
$$

Let

$$
	a=\frac{k^2}{2}.
$$

Then

$$
	\boxed{\begin{gathered}x(\theta)=a[\theta-\sin(\theta)], \\ y(\theta)=a[1-\cos(\theta)].\end{gathered}}
$$

These are the parametric equations of a cycloid.

Lecture reference: [[Lecture 07 Slides.pdf#page=7|Lecture 07 Slides, pp. 7–13]].

<br>

### Fit the Cycloid to the Endpoint

Let the endpoint $P_2$ correspond to $\theta=\theta_2$. Then

$$
	\begin{gathered}
		x_2=a[\theta_2-\sin(\theta_2)], \\
		y_2=a[1-\cos(\theta_2)].
	\end{gathered}
$$

Eliminate $a$ to obtain an equation for $\theta_2$:

$$
	\frac{x_2}{y_2}=\frac{\theta_2-\sin(\theta_2)}{1-\cos(\theta_2)}.
$$

After solving this equation numerically for $\theta_2$, calculate

$$
	a=\frac{y_2}{1-\cos(\theta_2)}.
$$

These two constants determine the unique cycloid arc joining the given endpoints within the selected branch.

<br>

### Minimum Travel Time

From the cycloid equations,

$$
	\frac{dx}{d\theta}=a[1-\cos(\theta)],\qquad \frac{dy}{d\theta}=a\sin(\theta).
$$

Therefore,

$$
	\begin{aligned}
		ds&=\sqrt{(\frac{dx}{d\theta})^2+(\frac{dy}{d\theta})^2}\,d\theta \\
		&=a\sqrt{[1-\cos(\theta)]^2+\sin^2(\theta)}\,d\theta \\
		&=2a\sin(\frac{\theta}{2})\,d\theta.
	\end{aligned}
$$

Also,

$$
	y=a[1-\cos(\theta)]=2a\sin^2(\frac{\theta}{2}),
$$

so

$$
	v=\sqrt{2gy}=2\sqrt{ag}\sin(\frac{\theta}{2}).
$$

Hence,

$$
	dt=\frac{ds}{v}=\sqrt{\frac{a}{g}}\,d\theta.
$$

The minimum travel time to $P_2$ is therefore

$$
	\boxed{T_{\min}=\sqrt{\frac{a}{g}}\,\theta_2}.
$$

---

# Conclusion

The fastest path is the cycloid

$$
	\boxed{\begin{gathered}x(\theta)=a[\theta-\sin(\theta)], \\ y(\theta)=a[1-\cos(\theta)].\end{gathered}}
$$

The constants are determined from the endpoint by

$$
	\begin{gathered}
		\frac{x_2}{y_2}=\frac{\theta_2-\sin(\theta_2)}{1-\cos(\theta_2)}, \\
		a=\frac{y_2}{1-\cos(\theta_2)}.
	\end{gathered}
$$
