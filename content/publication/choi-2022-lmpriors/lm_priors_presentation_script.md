Hello I am Chris Cundy, a PhD student from Stanford University.
I am going to talk today about our papaer LMPriors: pre-trained language mdoels as task-specific priors.
This is joint work with Kristy Choi, Sanjari Srivasta and our advisor Stefano Ermon.

* Overview
- We are probably all aware that in regimes where we have little data, a suitable prior distribution can be very useful, adding information about the problem which we know before looking at the data. However, especially in modern high-dimensional problems, finding a suitable prior distribution can be challenging.
- We could choose a generic prior such as a sparsity or simplicity prior, but these do not take into account a large amount of information that we are able to infer by looking at metadata such as variable names. 
- While we could try and elicit these detailed priors from domain experts, this is likely to be very expensive, and could be subjective and time consuming. For instance, it's impractical to ask for prior distribution over thousands of quantities of interest, which we might require for a modern problem.
- We address this problem with our method Language Model Priors (LMPriors). We essentially use a foundation model to serve as a plug-in replacement for an expert, using the full semantic meaning of any available metadata such as variable names or descriptions. This gives us a tailored prior for each different task, and leads to improved performance over generic priors in settings as diverse as feature selection for regression, causal inference, and safe reinforcement learning. 

* Method
** Assumptions
- We assume we have a dataset \({\cal D}\) and want to carry out some sort of learning procedure.
- Often, we have additional *metadata* such as variable names, variable descriptions, or details of how the data was collected.
- This is illustrated in the right-hand panel. On the left we have the raw data, with noninformative variable labels \(x\) and \(y\). On the right we have the same data but with D_meta added in. In this task, the goal is to infer if \(x\) causes \(y\) or \(y\) causes \(x\). Purely from the variable names and a bit of common sense, we already have a strong prior that the altitude is likely to cause the temperature to change, and not vice versa.

** The LMPrior framework
  
- We represent the LMPrior framework as an abstract functional transformation, where we take the metadata and use it to transform a learning procedure \(f\) into a new learning procedure f tilde, which is going to have an inductive bias (in the statistical sense ) towards results which are consistent with the metadata D_meta. This is shown on the right in our diagram.
  This might seem a bit abstract, so we will cover a concrete example in more detail in the next slide.

** A Detailed Example
- In this example, we assume we have a dataset consisting of \(x, y\) pairs and our job is to infer if \(x\) causes \(y\) or \(y\) causes \(x\). The metadata is the actual variable names and their descriptions. 
- We use the following prompt to construct an LMPrior for this task. We give the foundation model the two variables, description, and a context. We then ask the model to predict if A causes B or B causes A. We use a few-shot prompt with several common-sense examples which we made up ourselves. We evaluate the likelihood of the continuation A -> B as opposed to B -> A. We add this prior (log) probability directly to the (log) likelihood ratio obtained from a probabilistic method run on the data. 
- In this case, incorporating an LMPrior increases accuracy from 83.3% with purely the data to 84.5% with the metadata.

* Experimental Results
I will now go over some more extensive experimental results, but due to lack of time I would encourage those wanting extra details to look at the paper. 
** Proof of concept
- For time reasons I will skip over this proof of concept and go straight to the main result on feature selection. 
** Census
- We tackle a challenging variable selection problem, using the 286 variables from the detailed US Census Microdata dataset. We aim to predict whether a person's commute is over 20 minutes. We examine several methods of adding inductive bias or regularization, and observe that the LMPrior outperforms all baselines, over diverse set of machine learning models such as SVMs and Random Forests.
** Reinforcement Learning
- We also find that we can use an LMPrior to provide effective reward shaping for reinforcement learning, reducing the number of safety violations during training. Again I can't go into this in detail due to time, but please read the paper or chat to us for the detailed results. 
* Limitations
It's worth spending a second to reflect on the limitations of our method. 
- We are all aware that foundation models can encode toxic and harmful points of view. As such, it's especially important with our method that we don't end up propogating harmful biases into downstream models via the LMPrior.
- Secondly, we should be careful about people changing the prompt they use in the LMPrior until they achieve the desired prior, in a parallel to the $p$-hacking problem. This could be avoided partially by agreeing before the analysis to use a set of standardized prompts.
- Finally, our method has little use when there is no semantic meaning to be obtained from the metadata, possibly because the true variable labels have to be obfuscated for privacy reasons. 
* Conclusion
To conclude, I have given a brief overview of LMPriors, a method to extract effective, task-specific prior distributions in a scalable fashion by using Foundation models. Please read the paper or our poster for the full details.

Thank you for your time, and we would really welcome any feedback via email or speaking to us at the workshop.
