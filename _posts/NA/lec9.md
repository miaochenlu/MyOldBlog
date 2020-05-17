# Numerical Defferentiation and Integration

# 1. Numerical Differentiation

给定$x_0$, 近似$f'(x_0)$

$$f'(x_0)=\underset{h\rightarrow 0}{lim}\frac{f(x_0+h)-f(x_0)}{h}$$

<br>

* Forward

$$f'(x_0)\approx \frac{f(x_0+h)-f(x_0)}{h}$$

<img src="../../assets/images/image-20200511134731617.png" alt="image-20200511134731617" style="zoom:50%;" />

* Backward

$$f'(x_0)\approx\frac{f(x_0+h)-f(x_0)}{h}$$

<img src="../../assets/images/image-20200511134836704.png" alt="image-20200511134836704" style="zoom:50%;" />



Approximate $f(x)$ by its Lagrange polynomial with interpolating points $x_0$ and $x_0+h$

$f(x)=\frac{(x-(x_0+h))}{x_0-(x_0+h)}f(x_0)+\frac{x-x_0}{(x_0+h)-x_0}f(x_0+h)+\frac{f''(\xi_x)}{2}{(x-x_0)(x-x_0-h)}$

$f'(x)=\frac{f(x_0+h)-f(x_0)}{h}+\frac{2(x-x_0)-h}{2}f''(\xi_x)+\frac{(x-x_0)(x-x_0-h)}{2}\cdot \frac{d}{dx}[f''(\xi_x)]$

$\Rightarrow f'(x_0)=\frac{f(x_0+h)-f(x_0)}{h}-\frac{h}{2}f''(\xi)$

误差项就是$\frac{-h}{2}f''(\xi)$

不管什么插值方法，误差就是remainder求导

<br>

Given 3 points $x_0,x_0+h,x_0+2h$, please derive the three-point formulae for each of the points. Then give the best three-point formula for $f'(x)$

$f(x)=\frac{(x-(x_0+h))(x-(x_0+2h))}{(x_0-(x_0+h))(x_0-(x_0+2h))}f(x_0)+\frac{(x-x_0)(x-(x_0+2h))}{((x_0+h)-x_0)((x_0+h)-(x_0+2h))}f(x_0+h)$

$+\frac{(x-x_0)(x-(x_0+h))}{(x_0+2h-x_0)(x_0+2h-(x_0+h))}f(x_0+2h)+\frac{f^{(3)}(x)}{6}(x-x_0)(x-(x_0+h))(x-(x_0+2h))$

$f'(x_0)=\frac{1}{h}$



# Numerical Integration

目标是Approximate

$$I=\int_a^b f(x)dx$$

有个问题是不知道如何求$f(x)$的原函数。多项式的原函数我们会求，因此我们对$f(x)$进行插值，写成多项式形式。下面，我们对$f(x)$对拉格朗日插值多项式进行积分(当然也可以不是拉格朗日多项式，核心思想是插值)

在区间$[a,b]$上选择一些不同的点$a\leq x_0\leq x_1\leq\cdots\leq b$

$f(x)$的拉格朗日插值多项式为

$$P_n(x)=\sum_{k=0}^n f(x_k)L_k(x)$$

$$\int_a^b f(x)dx\approx \sum_{k=0}^n f(x) \int_a^b L_k(x)dx$$

令$A_k=\int_a^b L_k(x)dx=\int_a^b \Pi_{j\not=k}\frac{x-x_j}{x_k-x_j}dx$

注意到$A_k$只受当前插值点的取值影响。

<br>

Error $R[f]$

$$\begin{aligned}&=\int_a^b f(x)dx-\sum_{k=0}^n A_kf(x_k)\\&=\int_a^b[f(x)-P_n(x)]dx=\int_a^bR_n(x)dx\\&=\int_a^b\frac{f^{(n+1)}(\xi_x)}{(n+1)!}\Pi_{k=0}^n(x-x_k)dx \end{aligned}$$

积分的误差就是误差的积分

<br>

`Definition`{:.warning}

积分式的 ***degree of accuracy***(***precision***) 是最大的正整数$n$使得积分式对$x^k$来说是精确的,$k=0,1,\cdots,n$

结合下面的例子来理解这个概念

Example:

考虑在$[a,b]$区间线性插值，我们有

$$P_1(x)=\frac{x-b}{a-b}f(a)+\frac{x-a}{b-a}f(b)$$

$\int_a^bf(x)dx\approx f(a)\int_a^b\frac{x-b}{a-b}dx+f(b)\int_a^b\frac{x-a}{b-a}dx=\frac{b-a}{2}[f(a)+f(b)]$

determine the precision of this formula:

考虑$x^k$, $(k=0,1,\cdots)$

$x^0=1$: $\int_a^b1dx=b-a=\frac{b-a}{2}[1+1]$

$x^1=x$: $\int_a^b xdx=\frac{b^2-a^2}{2}=\frac{b-a}{2}[a+b]$

$x^2=x^2$: $\int_a^bx^2dx=\frac{b^3-a^3}{3}\not=\frac{b-a}{2}[a^2+b^2]$

那么precision的阶为1，因为($x^0,x^1$可以，$x^2$不可以)

<br>

For equally spaced nodes:

$x_i=a+th,h=\frac{b-a}{n},t=0,1,\cdots,n$

$$\begin{aligned}A_i=&\int_{x_0}^{x_0}\Pi_{j\not=i}\frac{x-x_j}{x_i-x_j}dx\\=&\int_0^n\Pi_{i\not=j}\frac{(t-j)h}{(i-j)h}hdt\\=&\frac{(b-a)(-1)^{n-i}}{ni!(n-i)!}\int_0^n\Pi_{i\not=j}(t-j)dt \end{aligned}$$

我们称$\frac{(-1)^{n-i}}{ni!(n-i)!}\int_0^n\Pi_{i\not=j}(t-j)dt$为cote coefficients $C_i^{(n)}$, n表示最大插值点下标，i表示当前插值点下标

这个系数既不依赖于$f(x)$, 也不依赖于$[a,b]$, 仅仅有$n,i$决定。

因此我们可以构建一张表，然后查表获得这些系数的值。

<br>

$n=1$时，也就是有两个插值点

$C_0^{(1)}=\frac{1}{2},C_1^{(1)}=\frac{1}{2}$

$\int_a^b f(x)dx\approx \frac{b-a}{2}[f(a)+f(b)]$

$R[f]=\int_a^b\frac{f''(\xi_x)}{2!}(x-a)(x-b)dx=-\frac{1}{12}h^3f''(\xi)$

其中$\xi\in[a,b],h=b-a$

Precision=1

<br>

$n=2$时

$C_0^{(2)}=\frac{1}{6},C_1^{(2)}=\frac{2}{3},C_2^{(2)}=\frac{1}{6}$

$\int_a^b f(x)dx=\frac{1}{6}(b-a)f(a)+\frac{2}{3}(b-a)f(\frac{a+b}{2})+\frac{1}{6}(b-a)f(b)$

Precision=3

<br>

$n=3$, precision=3

$n=4$, precision=5





# Composite Numerical Integration

在spline插值中，我们知道高阶多项式会振荡，所以可以一小段一小段的去approximate $f(x)$

#### Composite Trapezoidal Rule:

$h=\frac{b-a}{n},x_k=a+kh\;(k=0,\cdots,n)$

用梯形公式计算$[x_{k-1},x_k]$区间的面积

$$\int_{x_{k-1}}^{x_k}f(x)dx\approx \frac{x_k-x_{k-1}}{2}[f(x_{k-1})+f(x_k)]\quad k=1,2,\cdots,n$$

因此

$$\int_a^bf(x)dx\approx \sum_{k=1}^n\frac{h}{2}[f(x_{k-1})+f(x_k)]=\frac{h}{2}[f(a)+2\sum_{k=1}^{n-1}f(x_k)+f(b)]=T_n$$

$R[f]=\sum_{k=1}^n[-\frac{h^3}{12}f''(\xi_k)]=-\frac{h^2}{12}(b-a)\frac{\sum_{k=1}^nf''(\xi_k)}{n}$

$=-\frac{h^2}{12}(b-a)f''(\xi)\quad \xi\in(a,b)$

#### Composite Simpson's Rule:

$h=\frac{b-a}{n},x_k=a+kh\;(k=0,\cdots,n)$

$$\int_a^bf(x)dx\approx\frac{h}{6}[f(a)+4\sum_{k=0}^{n-1}f(x_{k+\frac{1}{2}})+2\sum_{k=0}^{n-2}f(x_{k+1}) +f(b)]=S_n$$

$R[f]=-\frac{b-a}{180}(\frac{h}{2})^4f^{(4)}(\xi)$

为了简化表示，不如令$n'=2n$, 那么$h'=\frac{b-a}{n'}=\frac{h}{2},x_k=a+kh'$

$$S_n=\frac{h'}{3}[f(a)+4\sum_{odd\;k}f(x_k)+2\sum_{even\;k}f(x_k)+f(b)]$$



结论：Composite integration techniques are all **stable**


