---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "F Divergences"
subtitle: ""
summary: "A first try at blogging, I explore some interesting properties of f divergences"
authors: []
tags: []
categories: []
date: 2020-06-02T13:40:46-07:00
lastmod: 2020-06-02T13:40:46-07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

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

There are two very common `closeness' measures for 
