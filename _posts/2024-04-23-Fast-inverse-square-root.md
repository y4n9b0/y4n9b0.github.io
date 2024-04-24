---
layout: post
title: 快速平方根倒数
date: 2024-04-23 13:00:00 +0800
categories: algorithm
tags: bit-hack
published: true
---

* content
{:toc}

## 雷神之锤3源码

```C
float Q_rsqrt( float number )
{
    long i;
    float x2, y;
    const float threehalfs = 1.5F;

    x2 = number * 0.5F;
    y  = number;
    i  = * ( long * ) &y;                       // evil floating point bit level hacking
    i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
    y  = * ( float * ) &i;
    y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//  y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

    return y;
}
```

## IEEE 754

* 科学计数法
* normalised numbers
* denormalised numbers
* NaN
* 0 & -0

由 IEEE 754 可知 float 的 long representation 为 $$(E*2^{23} + M)$$


## 牛顿迭代

牛顿迭代公式：

$$ x_{n+1} = x_n - \tfrac{f(x_n)}{f^{\prime}{(x_n)}} $$

欲求 n 的平方根倒数，构造函数 $$ f(x) = \tfrac{1}{x^2} - n = x^{-2} - n $$，令 f(x) = 0：

$$
\begin{aligned}
& \quad\,f(x) = \tfrac{1}{x^2} - n = 0\\
&\Rightarrow \tfrac{1}{x^2} = n\\
&\Rightarrow x = \tfrac{1}{\sqrt{n}}
\end{aligned}
$$

显然当 f(x) = 0 时，x 的值即为 n 的平方根倒数。
令迭代初始值为 $$ x_0 $$：

$$
\begin{aligned}
& \quad\,x_1 = x_0 - \tfrac{f(x_0)}{f^{\prime}(x_0)}\\
&\Rightarrow x_1 = x_0 - \tfrac{x_0^{-2} - n}{-2*x_0^{-3}}\\
&\Rightarrow x_1 = x_0 * (1.5 - \tfrac{n}{2} * x_0^2)
\end{aligned}
$$

泰勒级数展开式：

$$ f(x) = f(x_0) + f^{\prime}(x_0)(x-x_0) + \tfrac{f^{\prime\prime}(x_0)(x-x_0)^2}{2!} + ··· +  \tfrac{f^{(n)}(x_0)(x-x_0)^n}{n!} + R_n(x) $$ 

牛顿迭代提高精度的几种方式：
1. 选取一个合适的初始值 $$x_0$$
2. 增加迭代次数
3. 多阶迭代

## 推导

x 为 float，令 y 为 x 的平方根倒数：

$$
\begin{aligned}
& \quad\,y = \tfrac{1}{\sqrt{x}} = {x^{-\tfrac{1}{2}}}\\
&\Rightarrow\log_2 (y)=-\tfrac{1}{2}\log_2 (x)
\end{aligned}
$$

令 $$E_y$$、$$M_y$$ 和 $$E_x$$、$$M_x$$ 分别为 y 和 x 的指数、小数部分：

$$
\begin{aligned}
&\Rightarrow \log_2 (2^{(E_y-127)}*(1.0+\tfrac{M_y}{2^{23}})) = -\tfrac{1}{2}\log_2 (2^{(E_x-127)}*(1.0+\tfrac{M_x}{2^{23}}))\\
&\Rightarrow {(E_y-127)}+\log_2 (1.0+\tfrac{M_y}{2^{23}}) = -\tfrac{1}{2}(E_y-127)-\tfrac{1}{2}\log_2 (1.0+\tfrac{M_x}{2^{23}})
\end{aligned}
$$

代入 $$ \log(1+x) \approx x+\mu, x \in [0,1] $$：

$$
\begin{aligned}
&\Rightarrow {(E_y-127)} + \tfrac{M_y}{2^{23}} + \mu = -\tfrac{1}{2}(E_y-127) - \tfrac{1}{2}(\tfrac{M_x}{2^{23}} + \mu)\\
&\Rightarrow E_y*{2^{23}} + M_y = \tfrac{3}{2}(127 - \mu)2^{23} - \tfrac{1}{2}(E_x*2^{23} + M_x)
\end{aligned}
$$

记 $$y^{\prime}$$ $$x^{\prime}$$ 分别为 float 数 y 和 x 的 long representation：

$$
\begin{aligned}
&\Rightarrow y^{\prime} = \tfrac{3}{2}(127 - \mu)2^{23} - \tfrac{1}{2}x^{\prime}\\
&\Rightarrow y^{\prime} = \tfrac{3}{2}(127 - \mu)2^{23} - (x^{\prime} >> 1)
\end{aligned}
$$



## 总结

现在已经有了平方根运算器，已经不需要这么写了。

<!-- https://www.youtube.com/watch?v=p8u_k2LIZyo -->
<!-- https://www.zhihu.com/question/26287650 -->
<!-- https://en.wikipedia.org/wiki/Fast_inverse_square_root -->
<!-- https://zh.wikipedia.org/wiki/%E5%B9%B3%E6%96%B9%E6%A0%B9%E5%80%92%E6%95%B0%E9%80%9F%E7%AE%97%E6%B3%95 -->
<!-- https://zhuanlan.zhihu.com/p/400064205 -->
<!-- https://brilliant.org/wiki/newton-raphson-method/ -->
<!-- https://www.cnblogs.com/ywsun/p/14271547.html -->