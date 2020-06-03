Intuition behind f-divergences
============
This is a quick article to test out the blogging capability of the
site, and draw some attention to an interesting part of information theory,
$latex f $-divergences. 

A very natural question that arises in machine learning and statistics is how to compare two
different probability distributions. In statistics, we might want to compare the empirical
distribution of observed results to the distribution we would expect if some phenomenon was (or was
not) present. In machine learning, we might want to try to learn the distribution generating a set of
data by choosing from a set of candidate explanatory distributions. In both cases, we would like a
notion of `closeness' for two probability distributions. 

For parametric distributions we can compare the parameters of the distributions directly. For two
univariate Gaussian distributions \(\mathcal{N}(\theta_1, \sigma_1^2)\),
\(\mathcal{N}(\theta_2, \sigma_2^2)\) a reasonable measure of their closeness would be e.g.
\(\sqrt{(\theta_1 - \theta_2)^2 + (\sigma_1^2 - \sigma_2^2)}\). However, our ideal measure
of closeness would work for any two distributions (assuming they have the same support), and
wouldn't require a parametric form of the distribution to be available.