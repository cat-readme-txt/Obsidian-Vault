## The IEEE-754 Convention ($1.xxxx_2$)
Floating-point stores a number in a binary scientific-notation form:
$$
	\begin{gathered}
		\text{value}=(-1)^{s}\times 1.m\times 2^{E}
	\end{gathered}
$$

or

$$
	\begin{gathered}
		\text{value}=(-1)^{s}\times (1+m)\times 2^{e-\text{bias}}
	\end{gathered}
$$

where

- $s=\text{signed bit}$
- $e=\text{the stored exponent field}$
- $\text{bias}=\text{fixed offset}$
- $E=e-\text{bias}=\text{the actual exponent}$
- $m=\text{stored fractional part of the significand}$

> [!question]- Why do we need a bias?
> The actual exponent is **not stored in two's complement in IEEE-754 floating point** and uses a **Bias**.
> For example:
> 
> $$
> 	\begin{gathered}
>		1.01_{2}\times 2^3
>	\end{gathered}
> $$
> 
> and:
> 
> $$
>	\begin{gathered}
>		1.01_{2}\times 2^{-3}
>	\end{gathered}
> $$
> 
> The exponent field itself is just a sequence of bits. IEEE-754 avoids storing it using a separate sign bit or ordinary two's complement. Instead, it stores the exponent after adding a fixed number called the **bias**.
> 
> For an 8-bit exponent field, as in a 32-bit `float`:
> 
> $$
>	\begin{gathered}
>		\text{bias}=2^{8-1}-1=127
>	\end{gathered}
> $$
> 
> So:
> 
> $$
>	\begin{gathered}
>		e=E+127
>	\end{gathered}
> $$
> 
> or equivalently:
> 
> $$
>	\begin{gathered}
>		E=e-127
>	\end{gathered}
> $$
> 
> Examples:
> 
> |Actual exponent E|Stored exponent e|
> |---|---|
> |-3|124|
> |-1|126|
> |0|127|
> |1|128|
> |3|130|

> [!question]- Why $1+m$?
> The IEEE-754 convention follows so that the decimal point is placed behind/after the first non-zero digit of a binary number, therefore, the $1$ in front of the decimal point is guaranteed of existence and is therefore ignored when storing the mantissa.
> 
> See the below note: "Difference Between Conventions" for a detailed example.

> [!note] Difference Between Conventions:
> Consider $3.14\approx11.001000111101011..._{2}\approx11.001_{2}$:
> 
> ### Legacy 0.x Normalized Exponential Form:
> $3.14\approx0.11001_{2}\times2^{2}$
> 
> ### IEEE-754 1.x Normalized Exponential Form:
> $3.14\approx1.1001_{2}\times2^{1}\quad \text{(similar to the Scientific Notation)}$
> 
> This is preferred because every normalized non-zero binary number begins with $1.$, therefore that leading $1$ doesn't need to be stored.

<br>

### Bits Stored For Each Part of a Floating-Point Number Data Type
|Type|Total bits|Sign|Exponent|Fraction / mantissa field|
|---|--:|--:|--:|--:|
|`float`|32|1|8|23|
|`double`|64|1|11|52|

### IEEE-754 Interpretation Rules
|Category|Sign bit|Exponent field|Fraction / mantissa field|Meaning|Formula / interpretation|Important notes|
|---|---|---|---|---|---|---|
|Normal positive|`0`|neither all 0 nor all 1|anything|Positive finite normalized number|$(+2^{e-\text{bias}}(1+m))$|Most ordinary floating-point values|
|Normal negative|`1`|neither all 0 nor all 1|anything|Negative finite normalized number|$(-2^{e-\text{bias}}(1+m))$|Same magnitude rules as positive|
|Positive zero|`0`|all `0`|all `0`|`+0.0`|exactly zero|Compares equal to `-0.0`|
|Negative zero|`1`|all `0`|all `0`|`-0.0`|exactly zero with negative sign|Sign can matter in some operations|
|Positive subnormal|`0`|all `0`|not all `0`|Very small positive finite number|$(+2^{1-\text{bias}}m)$|No implicit leading `1`|
|Negative subnormal|`1`|all `0`|not all `0`|Very small negative finite number|$(-2^{1-\text{bias}}m)$|Allows gradual underflow|
|Positive infinity|`0`|all `1`|all `0`|$(+\infty)$|special encoding|Can result from overflow or division by positive zero|
|Negative infinity|`1`|all `1`|all `0`|$(-\infty)$|special encoding|Can result from negative overflow etc.|
|NaN|either|all `1`|not all `0`|Not-a-Number|special encoding|Used for undefined floating-point results|
|Quiet NaN|either|all `1`|nonzero fraction with appropriate NaN encoding|NaN that propagates quietly|implementation/standard-defined encoding details|Most NaNs seen in normal programs|
|Signaling NaN|either|all `1`|nonzero fraction with signaling encoding|NaN intended to signal invalid operation|platform-dependent handling|Less commonly encountered|