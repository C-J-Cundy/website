---
title: "Privacy-Constrained Policies via Mutual Information Regularized Policy Gradients"
date: 2020-12-30
publishDate: 2020-12-30
authors: ["Chris Cundy", "Stefano Ermon"]
publication_types: ["3"]
abstract: "As reinforcement learning techniques are increasingly applied to real-world decision problems, attention has turned to how these algorithms use potentially sensitive information. We consider the task of training a policy that maximizes reward while minimizing disclosure of certain sensitive state variables through the actions. We give examples of how this setting covers real-world problems in privacy for sequential decision-making. We solve this problem in the policy gradients framework by introducing a regularizer based on the mutual information (MI) between the sensitive state and the actions at a given timestep. We develop a model-based stochastic gradient estimator for optimization of privacy-constrained policies. We also discuss an alternative MI regularizer that serves as an upper bound to our main MI regularizer and can be optimized in a model-free setting. We contrast previous work in differentially-private RL to our mutual-information formulation of information disclosure. Experimental results show that our training method results in policies which hide the sensitive state."
featured: false
publication: "*Preprint*"
url_pdf: https://arxiv.org/abs/2012.15019.pdf
---

