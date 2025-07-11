# Online Barnes Estimation
16:03 test In this post we will briefly introduce Barnes interpolation and then cover a method of applying Barnes interpolation to an online context.

## Barnes Interpolation
Say we have a collection of data $$(X_1, Y_1), \cdots, (X_n,Y_n)$$ which are iid pairs of covariates $$X_i$$ and responses $$Y_i$$. Also say that $$(X_1, Y_1) \in R^d \times R$$. For this problem want to learn about the function $$\mu(x) = E[Y | X = x]$$, the mean of $$Y$$ given that $$X = x$$. 

Barnes interpolation is a method of doing this that allows for unevenly spaced input data. The general idea is that, when computing an estimate $$\hat \mu (x)$$ for $$\mu(x)$$, we care more about samples $$Y_i$$ where $$X_i$$ is close to $$x$$ than those where the samples are far from $$x$$. We therefore compute $$\hat\mu(x)$$ as a weighted average of the $$Y_i$$ with weighting $$e^{-(d_i(x)^2/2k^2)}$$. Here k is some scaling parameter that determines how rough our final interpolation surface is and $$d_i(x)$$ is a measure of the distance between $$x$$ and the sample $$X_i$$. In this text we wil primarily consider the (fairly standard) case where $$d_i(x) = \vert\vert X_i - x\vert\vert_2$$ is the euclidean distance between the two points. 

The algorithm for a (single pass) Barnes interpolation is as follows:

$$\hat\mu(x) = \frac{\sum_{i=1}^n Y_i e^{-\frac{d_i(x)^2}{2k^2}}}{\sum_{i=1}^n e^{-\frac{d_i(x)^2}{2k^2}}} $$

Note that as we vary $$x$$, the estimate $$\hat\mu(x)$$ is smooth (i.e. infinitely differentiable). However, we have the parameter $$k$$ that determines how "rough" the function is in a sense. If $$k$$ is very large, then the term in the exponent: 

$$-\frac{d_i(x)^2}{2k^2}$$

is roughly $$0$$ and so the weights are all roughly 1. Hence, this behaves similarly to $$\hat\mu(x) = \bar Y$$. On the other hand if $$k$$ is very small, the closest point will dominate and this interpolation method behaves like a 1-NN (1 - nearest neighbour) model. In practice, we can select $$k$$ with cross validation. Barnes interpolation's outputs are not hugely sensitive to the exact value of $$k$$ so typically we might consider a few values of slightly different orders of magnitude when performing cross validation. We would centre these around the "lengthscale" of our data - the typical distance between two points. If this is around $$10$$ then we might try cross validation with $$k = 1,3,10,30,100$$.

## An online approach
Our goal is to apply Barnes interpolation in an online setting. This is a problem where we have some (possibly none) of our data at the start but then over time new data comes in. We want to, when $$m$$ samples have arrived, be able to quickly compute a Barnes estimate $$\hat\mu^{(m)}(x)$$ using these $$m$$ samples. Say we have some prespecified set of points $$z_1, \cdots, z_p$$ (for instance a grid of points in $$R^d$$) and we want to determine estimates for $$\hat\mu(x)$$ at these points. 

To achieve this, we can initialize our method with $$S_0 = 0, W_0 = 0$$ and then when data $$X_i, Y_i$$ comes in, we compute:

$$ S^{(i)}_j = S^{(i-1)}_j + Y_i \times \exp\left(-\frac{d_i(z_j)^2}{2k^2}\right)$$

as well as:
$$ W^{(i)}_j = W^{(i-1)}_j + \exp\left(-\frac{d_i(z_j)^2}{2k^2}\right)$$

for each $$j \in 1, \cdots, p$$. Then, if we want the Barnes estimate at a point $$z_j$$ on our grid we can compute $$\hat \mu^{(m)}(z_j) = S^{(m)}_j / W^{(m)}_j$$.

## Time complexity
Imagine a case where data enters our system over time and we want to have (relatively) up to date values of the Barnes interpolated surface. We'll say that we want to update our estimate of the surface every $$q$$ steps, and that there are a total of $$N = nq$$ datapoints. 

With the standard approach we simply need to store $$X_i, Y_i$$ at each step (which will generally be insignificant in comparison to the gaps in time between incoming data). To evaluate any particular point $$z_j$$ in our grid when $$m$$ datapoints have been received we need order $$m$$ computations. Therefore, if we compute an estimate for the whole grid after collecting $$qr$$ datapoints (for $$r=1,2,3,\cdots, N/q = n$$) then we would perform approximately $$p \times q \times N/q \times (N/q+1)/2$$ computations. This is on the order of $$pqn^2$$. 

On the other hand if we use the online approach then at each step we update the $$2p$$ values of the $$S^{(i)}$$ and $$W^{(i)}$$. In cases where $p$ is very large and data enters very quickly there is a concern that this may not be fast enough, but for reasonably coarse grids this is likely not a huge issue. To evaluate a particular point $$z_j$$ after $$m$$ datapoints have been received we only need a constant number of operations (dividing $$S^{(i)}_j$$ by $$W^{(i)}_j$$. Hence, if we are computing an estimate for the whole grid after collecting $$qr$$ datapoints for $$r=1,2,3,\cdots, N/q = n$$ then we would perform approximately $$pqn$$ operations. 

We can see that for large $$n$$, i.e. for online cases where the method runs for a long time and lots of data is collected, the latter option is far faster. In the standard approach we can see that eventually the time taken to compute the estimate will grow to be larger than the time before another $q$ datapoints are received, in which case we do not finish computing the current estimate before we have enough data for the next estimate.

## Space complexity
In the standard approach we must store all $$(X_i, Y_i)$$ pairs until the very end meaning $$O(n)$$ storage space. For the online version, we only need to store the $$W^{(i)}_j$$ and $$S^{(i)}_j$$ values for the current $$i$$ and every $$j \in \{1, \cdots, p\}$$. Hence, this is $$O(p)$$ storage space. 
