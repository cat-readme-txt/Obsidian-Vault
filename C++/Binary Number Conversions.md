## Positive Integer to Binary
Divide the integer by 2, record the remainders from right to left, subtract the remainder from the integer if the remainder is non-zero, and resume the division process until the integer becomes 0.

Example:

Consider:
$$
	\begin{gathered}
		157_{10}\rightarrow \text{Binary}_{2}
	\end{gathered}
$$

Complete process:
$$
	\begin{gathered}
		b=\text{integer in binary}=xxxxxx_2 \\
		\\
		\text{1st Iteration:} \\
		157\pmod{2} \equiv 1 \\
		\left\lfloor  \frac{157}{2}  \right\rfloor = 78 \\
		\\
		\text{2nd Iteration:} \\
		78\pmod{2} \equiv 0 \\
		\lfloor \frac{78}{2} \rfloor = 39 \\
		\\
		\text{3rd Iteration:} \\
		39
	\end{gathered}
$$