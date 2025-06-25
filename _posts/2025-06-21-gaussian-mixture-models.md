# Gaussian Mixture models

In many contexts one might have some underlying data with a complex distribution. We could have some unknown shape that doesn't fit into any classical family or a bimodal distribution where there are multiple peaks. For example, we might consider the distribution of heights of humans. Men typically have heights around 5'9 and women's heights are around 5'3. This leads to a distribution of heights with peaks at both 5'3 and 5'9 - a property not commonly seen in most families of distributions.

One way to deal with this is to consider mixtures of distributions. In general, we might have a mixture of $m$ distributions, $D_1, \cdots, D_m$. Then, one can view the samples in our dataset as samples from:
1. Sample $\pi \sim Multi(1, p_1, \cdots, p_m)$. This determines which of the $m$ distributions in our mixture this sample comes from.
2. Sample $Y \sim D_{\pi}$.

