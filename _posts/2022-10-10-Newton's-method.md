---
layout: post
title:  牛顿迭代
date:   2022-10-10 18:00:00 +0800
categories: publish
tags: Algorithm
published: true
---

* content
{:toc}

## 前言

牛顿迭代法 (Newton's method) 又称牛顿-拉夫逊方法（Newton-Raphson method）


无限循环，间隔1秒，最后一帧3秒
![taylor formula]({{ '/styles/images/Newton-Raphson/taylor-series.gif' | prepend: site.baseurl }})


<script type="math/tex">E=mc^2</script>

This sentence uses `$` delimiters to show math inline:  $\sqrt{3x-1}+(1+x)^2$

**The Cauchy-Schwarz Inequality**

$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$

**Here is some math!**

$\sqrt{3x-1}+(1+x)^2$

$$\sqrt{3x-1}+(1+x)^2$$

\sqrt{3}

$$
\begin{aligned}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{aligned}
$$


\(E=mc^2\)

\[E=mc^2\] 

test1$E=mc^2$test2 test3 $E=mc^2$

$$E=mc^2$$

$$
E=mc^2
$$

$$
\boxed{E=mc^2}
$$


$x^2$

$x_2$

$10^{10}$
$10_{10}$
${10^5}^6$

$$10^{10}$$
$$10_{10}$$
$${10^5}^6$$

$x_i^2$
$x_{i^2}$
${x_i}^2$
${x^i}^2$
${x_i}^2$

$$x_i^2$$
$$x_{i^2}$$
$${x_i}^2$$
$${x^i}^2$$
$$x^{i^2}$$

$sqrt{b}$
$sqrt[a]{b}$

$\frac {a}{b}$
$a+1 \over b+1$


https://zhuanlan.zhihu.com/p/400064205
https://brilliant.org/wiki/newton-raphson-method/
https://www.sciencedirect.com/topics/mathematics/newton-raphson-method
https://en.wikipedia.org/wiki/Newton%27s_method
