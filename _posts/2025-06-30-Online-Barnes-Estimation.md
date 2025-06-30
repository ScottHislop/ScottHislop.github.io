# Online Barnes Estimation
In this post we will briefly introduce Barnes interpolation and then cover a method of applying Barnes interpolation to an online context.

## Barnes Interpolation
Say we have a collection of data $(X_1, Y_1), \cdots, (X_n,Y_n)$ which are iid pairs of covariates $X_i$ and responses $Y_i$. Also say that $(X_1, Y_1) \in R^d \times R$. For this problem want to learn about the function $\mu(x) = E[Y | X = x]$, the mean of $Y$ given that $X = x$. 

Barnes interpolation is a method of doing this that allows for unevenly spaced input data. The general idea is that, when computing an estimate $\hat \mu (x)$ for $\mu(x)$, we care more about samples $Y_i$ where $X_i$ is close to $x$ than those where the samples are far from $x$. We therefore compute $\hat\mu(x)$ as a weighted average of the $Y_i$ with weighting $e^{-(d_i(x)^2/2k^2)}$. Here k is some scaling parameter that determines how rough or smooth our final interpolation surface is and $d_i(x)$ is a measure of the distance between $x$ and the sample $X_i$. In this text we wil primarily consider the (fairly standard) case where $d_i(x) = ||X_i - x||_2$ is the euclidean distance between the two points. 

The algorithm for a (single pass) Barnes interpolation is as follows:

$$\hat\mu(x) = \frac{\sum_{i=1}^n Y_i e^{-\frac{d_i(x)^2}{2k^2}}}{\sum_{i=1}^n e^{-\frac{d_i(x)^2}{2k^2}}} $$



where new data enters the problem quickly and we want to have access to an interpolated surface at any time. 

