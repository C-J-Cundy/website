---
title: "Preference Learning with Lie Detectors can Induce Honesty or Evasion"
date: 2025-05-20
publishDate: 2025-05-20
authors: ["Chris Cundy", "Adam Gleave",]
publication_types: ["2"]
abstract: "As AI systems become more capable, deceptive behaviors can undermine evaluation and mislead users at deployment. Recent work has shown that lie detectors can accurately classify deceptive behavior, but they are not typically used in the training pipeline due to concerns around contamination and objective hacking. We examine these concerns by incorporating a lie detector into the labelling step of LLM post-training and evaluating whether the learned policy is genuinely more honest, or instead learns to fool the lie detector while remaining deceptive. Using DolusChat, a novel 65k-example dataset with paired truthful/deceptive responses, we identify three key factors that determine the honesty of learned policies: amount of exploration during preference learning, lie detector accuracy, and KL regularization strength. We find that preference learning with lie detectors and GRPO can lead to policies which evade lie detectors, with deception rates of over 85\%. However, if the lie detector true positive rate (TPR) or KL regularization is sufficiently high, GRPO learns honest policies. In contrast, off-policy algorithms (DPO) consistently lead to deception rates under 25\% for realistic TPRs. Our results illustrate a more complex picture than previously assumed: depending on the context, lie-detector-enhanced training can be a powerful tool for scalable oversight, or a counterproductive method encouraging undetectable misalignment."
featured: true
publication: "*Preprint*"
url_pdf: https://arxiv.org/abs/2505.13787
---
 
