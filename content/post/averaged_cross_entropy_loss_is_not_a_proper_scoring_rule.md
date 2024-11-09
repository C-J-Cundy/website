+++
title = "(Averaged) Cross-Entropy Loss is not a Proper Scoring Rule"
author = ["Foo X. Bar"]
draft = false
+++

In [a recent paper](https://arxiv.org/abs/2306.05426) we wrote when referring to the KL divergence that `In some domains, the length of the sequences n differs in each example, which can be incorporated by choosing an effective length N = max n, and treating all sequences shorter than N as having a sequence of padding tokens appended`, with a footnote that `Some care is required here, as averaging the loss of each example over its length leads to an inconsistent estimator`.
Although we wrote `an inconsistent estimator` in the paper, more precisely we should have specified that it is not a \`proper scoring rule': in other words, the minimizer of a certain type of cross-entropy loss with variable-length sequences (in fact, the default loss in PyTorch) is not the data generating distribution.

Here I will elaborate on this point a bit more, and discuss why I doubt it's a problem for modern sequence modelling in practice.


## Preliminary {#preliminary}

We will start with some definitions.


### Proper Scoring Rule {#proper-scoring-rule}

A scoring rule is a way to evaluate how good a probabilistic prediction is. We say a scoring rule is "proper" if the best score (lowest loss) is always achieved by predicting the true probabilities (the probability of the data-generating process).


### The KL-divergence {#the-kl-divergence}

The KL-divergence between two distributions \\(P, Q\\) is

\begin{align\*}
D\_\text{KL}(P\\|Q) = \mathbb{E}\_{x \sim P}\left[\log P(x) - \log Q(x)\right].
\end{align\*}

As with all \\(f\\)-divergences defined with a strictly convex \\(f\\), the divergence is nonnegative with \\(D\_\text{KL}(P\\|Q) = 0\\) if and only if \\(P=Q\\).

If our random variable \\(x\\) is a collection of variables, such as a sequence \\(x\_{1:n}\\), we can write

\begin{align\*}
D\_\text{KL}(P\\|Q) = \mathbb{E}\_{x\_{1:n} \sim P}\left[\log P(x\_1) + \log P(x\_2|x\_1) + \ldots - \log Q(x\_1) - \log Q(x\_2|x\_1) - \ldots\right],
\end{align\*}

or

\begin{align\*}
D\_\text{KL}(P\\|Q) = \mathbb{E}\_{x\_{1:n} \sim P}\left[ \sum\_{i=1}^n\log P(x\_i|x\_{<i}) - \log Q(x\_i|x\_{<i})\right],
\end{align\*}


### The Cross-Entropy Loss {#the-cross-entropy-loss}

Since we usually don't have density estimates for a target distribution \\(P\\), we instead typically minimize the cross-entropy loss \\({\cal L}\_{\text{CE}}(P,Q)\\),

\begin{align\*}
{\cal L}\_\text{CE}(P,Q) = \mathbb{E}\_{x\_{1:n} \sim P}\left[- \log Q(x\_i|x\_{<i})\right],
\end{align\*}

It is clear that

\begin{align\*}
{\cal L}\_{\text{CE}}(P,Q) = \mathbb{E}\_{x \sim P}\left[ - \log Q(x)\right] = D\_\text{KL}(P,Q) - \mathbb{E}\_{x\sim P}\left[\log P(x)\right] = D\_\text{KL}(P,Q) + H(P),
\end{align\*}

from which we can see that the cross-entropy loss is simply a shifted KL-divergence. Since the shift doesn't depend on \\(Q\\), minimizing the cross-entropy loss with respect to \\(Q\\) will also minimize the KL-divergence. Finally, when we achieve this minimum, the numerical value of the cross-entropy loss will be equal to the entropy of \\(P\\).


### The Averaged Cross-Entropy Loss {#the-averaged-cross-entropy-loss}

Another variant of the cross-entropy loss is the averaged form. In particular, let's say we have data sequences which are of varying length. We can imagine they have a generative process where we first pick the length of the sequence \\(n\\), and then sample from sequences of that length. Since we want the losses to all be relatively similar in scale, we divide by the length of the sequence in our loss. In other words,

\begin{align\*}
{\cal L}\_\text{Avg CE}(P,Q) = \mathbb{E}\_{n \sim P} \left[\frac{1}{n} \mathbb{E}\_{x\_{1:n} \sim P\_n}\left[- \log Q(x\_i|x\_{<i})\right]\right],
\end{align\*}

As discussed below, this is effectively the default cross-entropy loss available for sequence modelling in PyTorch, when using a batch (or micro-batch) size of 1 and default keyword argument `reduction=mean`.
Below, I'll show that this loss is not a proper scoring rule. We'll do this by demonstrating a case where \\({\cal L}\_\text{Avg CE}(P,Q) < {\cal L}\_\text{Avg CE}(P,P)\\), for \\(Q \neq P\\).


## A Simple Sequential Model {#a-simple-sequential-model}

Let's say we have the following setup: we have a categorical variable \\(x\_i\\), taking on \\(m+1\\) values. Sequences of these variables form our dataset. The first \\(m\\) values are from our 'alphabet', but the last value, which we will call `<eos>` is the end-of-sentence token. If this is generated, the sequence ends.

We have a simple data-generating process \\(P\\). With probability \\(1/2\\), the `<eos>` token is generated. With probability \\(1/2\\), one of the alphabetical tokens is generated, so each alphabetical token has \\(1/2m\\) probability of being generated. The sequence continues generating until the `<eos>` token is generated, so almost surely has finite length.

We are going to fit this with another distribution \\(Q\\), which chooses `<eos>` with probability \\(q\\). With probability \\((1-q)/m\\) it chooses an alphabetical token, so each alphabetical token has probability \\((1-q)/m\\). Of course, \\(P=Q\\) when \\(q=1/2\\).

We'll calculate the KL-divergence, cross-entropy, and averaged cross-entropy between \\(P\\) and \\(Q\\). As expected, the KL-divergence and cross-entropy loss is minimized when \\(P=Q\\), but perhaps surprisingly, the averaged cross-entropy is not.


### The KL-divergence {#the-kl-divergence}

Computing the KL-divergence between \\(P\\) and \\(Q\\) gives

\begin{align\*}
D\_\text{KL}(P\\|Q) &= \mathbb{E}\_{x\_{1:n} \sim P}\left[ \sum\_{i=1}^n\log P(x\_i|x\_{<i}) - \log Q(x\_i|x\_{<i})\right]\\\\
              & = \frac{1}{2} \left[ -\log 2 - \log q \right] + \frac{1}{4}\left[ -\log 2 + \log \frac{1}{2m} - \log q - \log \frac{1-q}{m}\right] + \ldots\\\\
              & = \frac{1}{2} \left[ -\log 2 - \log q \right] + \frac{1}{4}\left[ -2\log 2  - \log q - \log 1-q \right] + \frac{1}{8}\left[ -3\log 2  - \log q - 2\log 1-q \right] + \ldots \\\\
              & = \sum\_{i=1}^n 2^{-i}\left[ -i \cdot \log 2 -\log q -(i-1)\log 1-q \right]\\\\
              & = -\log q + \log 1 - q + \sum\_{i=1}^n 2^{-i}\left[ -i (\log 2 -\log 1-q) \right]\\\\
              & = -\log q + \log 1 - q + -2(\log 2 -\log 1-q) = -2\log 2 - \log (1-q) - \log q.
\end{align\*}

where we have used the result that \\(\sum\_{i=1}^\infty 2^{-i} = 1\\), and that \\(\sum\_{i=1}^\infty 2^{-i}i = 2\\). The second result can be observed via a trick: writing out the terms \\(\sum\_{i=1}^\infty 2^{-i}i = 2^{-1} + 2 \* 2^{-2} + 3 \* 2^{-3} + \ldots\\), we can re-arrange the sum (as it is surely absolutely convergent) into a series of geometric series, \\(\sum\_{i=1}^\infty 2^{-i}i = \sum\_{i=1}^\infty 2^{-1} + \sum\_{i=1}^\infty 2^{-2} + \sum\_{i=1}^\infty 2^{-3} \ldots = 1 + 1/2 + 1/4 + \ldots = 2\\).

We can see that \\(D\_\text{KL}(P\\|Q) = -2\log 2 - \log (1-q) - \log q\\). Some rearrangement gives \\(D\_\text{KL}(P\\|Q) = 0\\) iff \\(q=1/2\\), as we would expect!


### Cross-Entropy {#cross-entropy}

As a check, we'll first compute the entropy of our sequences:

\begin{align\*}
H[P] & = \mathbb{E}\_{x \sim P}\left[ -\log P(x)\right]\\\\
     & = -\frac{1}{2} \left[ -\log 2\right] - \frac{1}{4}\left[ -\log 2 + \log \frac{1}{2m} \right] + \ldots\\\\
     & = \sum\_{i=1}^n 2^{-i}\left[ i \cdot \log 2 + (i-1) \log m \right]\\\\
     & = 2\log 2 + \log m\\\\
\end{align\*}

Now we can compute the cross-entropy loss:

\begin{align\*}
{\cal L}\_\text{CE}(P \\|Q) & = -\mathbb{E}\left[\log Q(x)\right]\\\\
                          & = -\log q + \log \frac{1-q}{m} - \sum\_{i=1}^\infty 2^{-i} \left[-i\log \frac{1-q}{m}\right]
                          & = -\log q - \log \frac{1-q}{m}.
\end{align\*}

We see again that the loss will be minimized when \\(q=1/2\\), whereupon the value of the loss will be \\(2\log 2 + \log m\\).
So good so far.


### Averaged Cross-Entropy Loss {#averaged-cross-entropy-loss}

Finally we can look at the averaged cross-entropy loss. Here we have

\begin{align\*}
{\cal L}\_\text{Avg CE}(P,Q) & = \mathbb{E}\_{n \sim P} \left[\frac{1}{n} \mathbb{E}\_{x\_{1:n} \sim P}\left[- \log Q(x\_i|x\_{<i})\right]\right]\\\\
                            & = -\frac{1}{2}\log q - \frac{1}{2}\cdot\frac{1}{4}\left[\log q + \log \frac{1-q}{m}\right] - \frac{1}{3}\cdot\frac{1}{8}\left[\log q + 2\log \frac{1-q}{m}\right] \\\\
                            & = -\sum\_{i=1}^\infty2^{-i}\frac{1}{i}\left[\log q - \log \frac{1-q}{m} \right] - \sum\_{i=1}^\infty2^{-i}\left[\log \frac{1-q}{m} \right]\\\\
                            & = -\log 2\left[\log q - \log \frac{1-q}{m} \right] - \log \frac{1-q}{m}\\\\
\end{align\*}

Where we have used the result that \\(\sum\_{i=1}^\infty2^{-i}\frac{1}{i} = \log 2\\), which follows by observing the power series \\(\log (1-x) = -x - \frac{x^2}{2}-\frac{x^3}{3}\\) and setting \\(x=\frac{1}{2}\\). We can compute the minimizing value for \\(q\\) by differentiating and setting to zero:

\begin{align\*}
\nabla\_q {\cal L}\_\text{Avg CE}(P,Q) & = -\frac{\log 2}{q} + (1 - \log 2)\frac{1}{1-q} \\\\
                                & 0 = -\frac{\log 2}{q} + (1 - \log 2)\frac{1}{1-q} \implies q = \log 2.
\end{align\*}

So, to minimize the averaged cross-entropy loss, the model has to give a probability which is not equal to the ground-truth probability!
A bit more algebra shows that if we have a probability \\(\beta\\) of an `<eos>` token, we should set \\(q = \frac{\beta-1}{\beta}\log (1 - \beta)\\).


#### Some Intuition {#some-intuition}

After going through the derivations, we can see that the averaged cross-entropy is not a proper scoring rule. This might seem a bit strange, since we can usually scale losses by a constant and the minimum is the same. However, the problem here is that the scaling depends on the content of the sequences. In other words, when choosing \\(q\\), we know that if the `<eos>` token appears, this current timestep will have a weighting of 1. However, if `<eos>` is not chosen, and is chosen at the next timestep, the current timestep will have a weighting of \\(1/2\\). So we can achieve higher likelihood by increasing the probability put on `<eos>`. Indeed, I believe that in the case where the content of the sequence is **independent** from the length, the averaged cross-entropy is still a proper scoring rule. Of course in contemporary sequence modelling, the content depends heavily on the length--indeed, for a sequence with length \\(n\\), the token at position \\(n-1\\) is always given by an `<eos>` token or similar.


## Practical Implications {#practical-implications}

Why is this discussion about averaged cross entropy relevant? I think it's worth thinking about because the [default implementation of the cross entropy loss in Pytorch](https://pytorch.org/docs/stable/generated/torch.) is an averaged cross entropy loss. Specifically, when dealing with a single masked sequence where the masking indices are set to `ignore_index`, the default behavior is to carry out an averaged cross entropy. The key advantage of this behavior is that the losses are on the same scale for each sequence (with the same order of magnitude of the entropy of the text). Practically speaking, having losses that are similar in scale is likely important for stable training.

However, it's worth pointing out that the behaviour under batching is quite close to the behaviour of the correct cross-entropy loss. If we have two sequences, one with length 1 and 1023 padding tokens, corresponding to loss \\(l\_1\\), and one with length 1024 and zero padding tokens with loss \\(l\_2\\), the cross-entropy loss is \\(l\_1 + l\_2\\). When computed on singleton batches and added up, the Pytorch default cross-entropy would be \\(l\_1 + l\_2 / 1024\\).
However, the loss computed by the default cross-entropy on this sequence when they are batched together would be \\((l\_1 + l\_2) / 1025\\), which is indeed a multiple of the correct loss for the pair. However, it's still a biased estimator of the population loss, which is what we want to minimize in principle.

In principle, this issue would arise more frequently with high amounts of gradient accumulation, such as a micro-batch size of 1.

I tried to demonstrate this issue in a practical setting by training a small transformer on a corpus and comparing the learned models. I used Andrej Kaparthy's Shakespeare corpus, and a vanilla 4-layer transformer. I trained with a minibatch size of 128, first using a per-device \`microbatch' size of 128 and then using a microbatch size of 1 with a factor of 128 gradient accumulation. I didn't use any dropout, so these two training setups should have been completely equivalent if non-averaged cross-entropy was being used.
As we can see, there are some differences between the trained models that are obtained. The model with smaller microbatches should be more likely to generate shorter sequences.
However, the effect is not particularly strong, given the large error bars. The test perplexity of the different models is also very similar across microbatch size.
Therefore, it seems that the effect of the non-proper loss is quite small in practice.

{{< figure src="/ox-hugo/shakespeare_plot.png" caption="<span class=\"figure-number\">Figure 1: </span>Different Training Statistics with different microbatch size." >}}


## References {#references}
