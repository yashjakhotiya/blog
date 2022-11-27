---
title: "pinv >>> inv"
description: "A flow on matrix ranks, invertibility, singular value decomposition, Moore-Penrose inverse and the least squares method"
layout: post
toc: true
comments: true
hide: false
search_exclude: false
image: /images/moore-penrose.png
categories: [linear algebra]
author: "<a href='https://www.linkedin.com/in/yash-jakhotiya/'>Yash Jakhotiya</a>"
---
In this post, let's go all the way back to something as "simple" as linear regression and it's closed form solution through the least squares method in an effort to unearth a couple of linear algebra gems.

# When explicit formulation failed
In an ML class, we were told to find out weights to a linear regression problem through the least squares method before we were introduced to gradient descent. My first instinct was to use - 

$$\theta = (X^TX)^{-1}X^TY$$

(derivation in the [slides](https://mahdi-roozbahani.github.io/CS46417641-fall2022/course/15-linear-regression-note.pdf))

But I didn't pass our autograder tests.

**Why?**

Because - 

1. The above form has an assumption that $X^TX$ has to be invertible. This will happen if the rank of $X^TX$, a DxD matrix, is equal to D.  $rank(X^TX)$ is always equal to  $rank(X)$. So we effectively need  $rank(X)$ to be D.  $X$ has rank  $D$ when $N >= D$ and  $X$ has linearly independent columns (i.e. linear independence in the  $D$ dimension). **But**, 

2. The autograder tests don't have such assumptions. For example, if $N < D$ then $rank(X^TX) <= N < D$, or if $N >= D$ and the columns (i.e. $X$'s features) are not linearly independent then  $rank(X)$ is again less than D, in which case we were told to use `pinv`.

_But why `pinv`?_ Read on to find out!

# The minimum norm solution - with derivatives
The objective of a linear regression problem is to get to a minimum norm solution, that is, a solution which has the square of the norm - equal to the square of the L2 distance between predicted and true values - minimum. One approach to do this is by taking a derivative of the norm and equating it to zero, which we did in class and got to - 

$$X^TX\theta = X^TY$$

(derivation [here](https://mahdi-roozbahani.github.io/CS46417641-fall2022/course/15-linear-regression-note.pdf))

This, again, has the same invertibility problems of $$X^TX$$.

# The (better) minimum norm solution - with SVD
Another approach to find a minimum norm solution consists of first finding the singular value decomposition of  $X$ 

$$X = UΣV^T$$

followed by, defining $X^+$ to be 

$$X^+=VΣ^+U^T$$

where $Σ^+$ is obtained by taking the reciprocal of each non-zero element on the diagonal, leaving the zeros in place, and then transposing the matrix.

and finally, finding theta by 

$$\theta = X^+Y = VΣ^+U^TY$$

The proof on why this theta gives the minimum norm solution is given [here](http://web.cs.ucla.edu/~chohsieh/teaching/CS260_Winter2019/notes_linearregression.pdf).

# Enter Moore-Penrose
The NumPy `pinv` we were using to solve this problem was NOT

$$(X^TX)^{-1}X^T$$

but the SVD $X^+$ defined above, also known as the Moore-Penrose inversion or 'pseudoinverse'. Don't believe me? Look for yourself [here](https://numpy.org/doc/stable/reference/generated/numpy.linalg.`pinv`.html)!

This is why our autograder tests passed when using `pinv`, because we were giving a minimum norm solution without relying on $X^TX$'s invertibility.

# The relation between the two approaches
But how does this Moore-Penrose inversion - the NumPy `pinv` - relate to what we proved in class? Well, it reduces to  

$$(X^TX)^{-1}X^T$$

when $X^TX$ is invertible!

Read more about the Moore-Penrose inversion [here](https://en.wikipedia.org/wiki/Moore%E2%80%93Penrose_inverse). The article defines Moore-Penrose inversion based on 4 Moore-Penrose conditions and then arrives at the SVD-based computation, but both of these are equivalent. You can define by the SVD-based computation and then prove those 4 conditions. Also, the A* notation there is known as Hermitian transpose, and is equal to the 'vanilla' transpose for real numbers.

Please feel free to correct me if you feel there's a mistake anywhere.

Thanks!