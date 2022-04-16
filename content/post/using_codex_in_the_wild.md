+++
title = "Using Codex in the Wild"
author = ["Chris Cundy"]
draft = false
+++

## Introduction {#introduction}

Following on from my previous article about using codex in emacs, I've found my plug-in more and more useful in everyday programming.

Some general impressions

-   At the moment, the results are on par with what I'd expect from a decent undergraduate programmer. For boilerplate code where it's fairly straightforward what to do, but just slightly tedious to write down, codex can save a lot of time.
    This is especially true if you're a bit rusty and would have to look up an API or two. For example, "scan over a folder and check if any of the files contained within called x.npy have any NaNs in them", or "find the optimal policy for env with a stable baselines3 SAC" both give correct results.
-   A particularly good application is plotting. Personally I can never remember all the different keyword arguments for matplotlib, so being able to send off a request to "make the title font bigger" and have it come back right away with the solution is awesome.
-   Using a language model as part of your workflow actually encourages you to comment and document your code better. If a person would understand your code better with documentation, then the LM will too. It will be able to respond to more generic queries like "make a plot showing the results" if you have documented what you expect the results to be, and if you have chosen meaningful variable names.
-   There will be bugs! Some of the time saved by using the language model is correspondingly wasted by fixing bugs that it introduces. There are several flavors of bugs that can be introduced. One that's hard to see how to avoid is when the LM will give you method names or api calls that are correct for previous versions of packages, but are now deprecated. This happened to me when it was suggested I use an outdated method in jax.


## An Example {#an-example}

The following is a set of commands that I gave to the language model as per my previous post.

> Write a function to do tabular Q-learning on a simple gridworld
>
> Write a simple grid environment to use with q\_learning
>
> Change this so the walls are automatically around the edges of the grid
>
> Write better documentation for this function
>
> Make sure that self.state is handled properly
>
> Add a method to represent the state as a unique integer
>
> Make sure that it is not possible to have the starting position in a wall
>
> Plot the Q-value in a grid
>
> Show the initial position and goal on the plot
>
> Make the vlims the same in each frame
>
> Make sure the reset method resets the state

I also did some minor fixing in-between each command, and I also selected different parts of the code as input for each command along with the instruction. I didn't record the input region too, I only recorded the instruction.
The resulting code can be found at [this google colab](https://colab.research.google.com/drive/1yx9VqAxuo1DJsUysBWTtdQ5Cld0IMEKC?usp=sharing). **It works pretty well**.
If the current rate of improvement continues on these language-model based code assistants, I suspect that most rank-and-file programmers will be out of a job within the near future.