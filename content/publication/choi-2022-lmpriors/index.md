---
title: "LMPriors: Pre-Trained Language Models as Task-Specific Priors"
date: 2022-12-01
publishDate: 2022-12-01T00:00:00.0
authors: ["Kristy Choi", "Chris Cundy", "Sanjari Srivasta", "Stefano Ermon"]
publication_types: ["2"]
abstract: "Particularly in low-data regimes, an outstanding challenge in machine learning is developing principled techniques for augmenting our models with suitable priors. This is to encourage them to learn in ways that are compatible with our understanding of the world. But in contrast to generic priors such as shrinkage or sparsity, we draw inspiration from the recent successes of large-scale language models (LMs) to construct task-specific priors distilled from the rich knowledge of LMs. Our method, Language Model Priors (LMPriors), incorporates auxiliary natural language metadata about the task -- such as variable names and descriptions -- to encourage downstream model outputs to be consistent with the LM's common-sense reasoning based on the metadata. Empirically, we demonstrate that LMPriors improve model performance in settings where such natural language descriptions are available, and perform well on several tasks that benefit from such prior knowledge, such as feature selection, causal inference, and safe reinforcement learning."
featured: false
publication: "*First Workshop on Foundation Models for Decision Making, Neurips 2022*"
url_pdf: https://arxiv.org/abs/2210.12530
url_poster: https://nips.cc/media/PosterPDFs/NeurIPS%202022/59640.png?t=1667865786.2184136
url_video: https://nips.cc/virtual/2022/59640
url_slides: lm_priors_presentation.pdf
url_transcript: lm_priors_presentation_script.md
---
