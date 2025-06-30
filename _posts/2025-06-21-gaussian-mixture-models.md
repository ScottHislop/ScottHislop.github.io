# Mixture models

In many contexts one might have some underlying data with a complex distribution. We could have some unknown shape that doesn't fit into any classical family or a bimodal distribution where there are multiple peaks. For example, we might consider the distribution of heights of humans. Men typically have heights around 5'9 and women's heights are around 5'3. This leads to a distribution of heights with peaks at both 5'3 and 5'9 - this bimodality property is not found in most families of distributions.

One way to deal with this is to consider mixtures of distributions. In general, we might have a mixture of $m$ distributions, $D_1, \cdots, D_m$, and samples being from a particular distribution / group / class with probability $p_i$ (the $p_i$ are known as the class probabilities). Then, one can view the samples in our dataset as samples from the following process:

1. Sample $\pi \sim Multi(1, p_1, \cdots, p_m)$. This determines which of the $m$ distributions in our mixture this sample comes from.
2. Sample $Y \sim D_{\pi}$.

For instance, we could have $m=2, p_1 = p_2 = 0.5$ and then if $D_1$ is the distribution of heights of men, and $D_2$ is the distribution of heights of women, then $Y$ sampled from the above would be the distribution of heights of all humans.

## The Gaussian mixture model
We can choose many types of distribution for the $D_i$ above. A special case is the Gaussian mixture model where each of the $D_i$ are Gaussians. More specifically, the i-th distribution is $N(\mu_i, \sigma_i^2)$. One can consider multivariate gaussian mixture models as well by changing these to multivariate gaussians with some covariance $\Sigma_i$ instead. 

## Fitting mixture models
We have described above how one can sample from a Gaussian mixture model if we have known means, covariances and class probabilities. In practice, we might have data in the form of unlabelled samples from a distribution we think is a mixture of Gaussians and want to determine the means, covariances and class probabilities. We now detail how one might go about this. We will look for the maximum likelihood estimator. 

A common approach, and the one we use here, when trying to find the MLE in a messy context is to compute the MLE of one variable / set of variables and then plug this in to compute the "MLE " of another set of variables.

That's not EM though?
