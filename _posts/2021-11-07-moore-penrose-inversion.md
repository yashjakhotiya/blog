---
title: "pinv >>> inv"
description: "A flow on matrix ranks, invertibility, singular value decomposition, Moore-Penrose inversion and Least Squares Method"
layout: post
toc: true
comments: true
hide: false
search_exclude: false
image: /images/2020-12-20-container-runtimes/runc_in_docker.png
categories: [linear algebra]
author: "<a href='https://www.linkedin.com/in/yash-jakhotiya/'>Yash Jakhotiya</a>"
---
In this post, let's go all the way back to something as "basic" as linear regression and it's closed form solution through least squares method and try to unearth a couple of linear algebra gems.

# When explicit formulation failed
In an ML class, we were told to find out weights to a linear regression problem through least squares method, before we went to gradient descent. My first instinct was to use the formulation - 

$$(X^TX)^{-1}X^T$$

But I didn't pass our autograder tests.

**Why?**

Because - 

1. The above formulation has an assumption that $X^TX$ has to be invertible. 

This will happen if rank of $X^TX$, a DxD matrix, is equal to D. 

 $rank(X^TX)$ is always equal to  $rank(X)$. So we effectively need  $rank(X)$ to be D.  $X$ has rank  $D$ when $N >= D$ and  $X$ has linearly independent columns (i.e. linear independence in the  $D$ dimension).

**BUT**

2. The autograder tests don't have such assumptions. For example, if $N < D$ then $rank(X^TX) <= N < D$ OR if $N >= D$ and the columns (i.e. $X$'s features) are not linearly independent, which makes  $rank(X)$ again less than D, in which case we were told to use `pinv`.

_But why `pinv`? What is so special about it?_ Read on to find out!

# Minimum norm solution - with derivatives
Our aim in a linear regression problem to form a minimum norm solution, i.e. a solution which will have our loss (square of the norm, square of the L2 distance between predicted and true values) minimum. One approach to do this is by taking a derivative of the norm and equating to zero, which we did in class (slides [here](https://mahdi-roozbahani.github.io/CS46417641-fall2021/course/15-linear-regression-note.pdf)!), and which leads to the formulation - 

$$X^TX\theta = X^TY$$

But the problem here is $X^TX$ can be invertible or can not be invertible, as stated above in the first section.

# (Better) minimum norm solution - with SVD
Another approach to find a minimum norm solution is by first finding singular value decomposition of  $X$ 

$$X = UΣV^T$$

and defining $X^+$ to be 

$$X^+=VΣ^+U^T$$

where $Σ^+$ is obtained by taking the reciprocal of each non-zero element on the diagonal, leaving the zeros in place, and then transposing the matrix.

and finding theta by 

$$\theta = X^+y = VΣ^+U^Ty$$

The proof on why this theta gives the minimum norm solution is given [here](http://web.cs.ucla.edu/~chohsieh/teaching/CS260_Winter2019/notes_linearregression.pdf).

# Enter Moore-Penrose
The NumPy `pinv` we were using to solve the assignment was NOT

$$(X^TX)^{-1}X^T$$

but the SVD $X^+$ defined above, also known as the Moore-Penrose inversion or 'pseudoinverse'. Don't believe me? Look for yourself [here](https://numpy.org/doc/stable/reference/generated/numpy.linalg.`pinv`.html)!

This is why our autograder tests passed when using `pinv`, because we were giving a minimum norm solution without relying on $X^TX$ invertibility.

# Relation between the two approaches
But how does this Moore-Penrose inversion, the SVD based X^+ definition, the NumPy `pinv` relate to what we proved in class? Well, it reduces to  

$$(X^TX)^{-1}X^T$$

when $X^TX$ is invertible!

Read more about Moore-Penrose inversion [here](https://en.wikipedia.org/wiki/Moore%E2%80%93Penrose_inverse). The article defines Moore-Penrose inversion based on 4 Moore-Penrose conditions and then arrives at SVD based computation, but both of these are equivalent. You can define by SVD based computation and then prove those 4 conditions. Also, don't worry about the A* notation there. It is known as Hermitian transpose, and is equal to the 'vanilla' transpose for real numbers.

Please feel free to correct me if you feel there's a mistake anywhere.

Thanks!