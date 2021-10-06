---
title: "IQ-Learn: Inverse soft-Q Learning for Imitation"
date: 2021-12-01
publishDate: 2021-12-01T03:12:38.030694Z
authors: ["Divyansh Garg", "Shuvam Chakraborty", "Chris Cundy", "Jiaming Song", "Stefano Ermon"]
publication_types: ["2"]
abstract: "In many sequential decision-making problems (e.g., robotics control, game playing, sequential prediction), human or expert data is available containing useful information about the task. However, imitation learning (IL) from a small amount of expert data can be challenging in high-dimensional environments with complex dynamics. Behavioral cloning is a simple method that is widely used due to its simplicity of implementation and stable convergence but doesn't utilize any information involving the environment's dynamics. Many existing methods that exploit dynamics information are difficult to train in practice due to an adversarial optimization process over reward and policy approximators or biased, high variance gradient estimators. We introduce a method for dynamics-aware IL which avoids adversarial training by learning a single Q-function, implicitly representing both reward and policy. On standard benchmarks, the implicitly learned rewards show a high positive correlation with the ground-truth rewards, illustrating our method can also be used for inverse reinforcement learning (IRL). Our method, Inverse soft-Q learning (IQ-Learn) obtains state-of-the-art results in offline and online imitation learning settings, surpassing existing methods both in the number of required environment interactions and scalability in high-dimensional spaces"
featured: false
publication: "*Neural Information Processing Systems (NeurIPS)*"
url_pdf: https://arxiv.org/pdf/2106.12142.pdf
---

