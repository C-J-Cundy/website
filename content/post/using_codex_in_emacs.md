+++
title = "Using Codex in Emacs"
author = ["Chris Cundy"]
draft = false
date: "2022-03-26T13:00:00+00:00"
+++

## Introduction {#introduction}

Recently OpenAI  [released](https://beta.openai.com/playground) their 'editing mode' API for their language models. In this mode (which you can select by clicking on the 'mode' selector on the right-hand-side), we are able to put a piece of context (such as a code snippet) called the 'input' and an instruction (such as change the sort to use a heapsort'). This is different from the previous completion-based API, which simply generated the most likely tokens given some initial context. In principle we could implement this input/instructions setup by simply appending the instructions to the input, but given the really good performance that this mode achieves, I suspect there's more going on here with additional fine-tuning and/or model changes. We can also use this interface to do \`zero-shot\` modification by leaving the input blank. In the initial beta stage of this editing API, it's completely free to use and we also have access to the `Codex` language models, which I don't currently have access to through the regular waitlist.

The first question is, does it work?
Let's try a few tests:


### General Code Generation {#general-code-generation}

```python
#Input: ""
#Instructions: "Sort a list of names, `names`"

#Output:
#!/usr/bin/env python

import sys

def sort_names(names):
    """
    Sort a list of names, `names`
    """
    names.sort()
    return names

if __name__ == '__main__':
    names = sys.argv[1:]
    print sort_names(names)
```

```python
#Input: ""
#Instructions: "Sort a list of names, `names` which is given as a csv file, #`./names.csv`"

#Output:
import csv

with open('./names.csv', 'r') as f:
    reader = csv.reader(f)
    names = list(reader)

names.sort()

print(names)
```

```python
#Input: ""
#Instructions: "Compute the Fibonacci numbers recursively"

#Output:
def fib(n):
    if n == 0:
	return 0
    elif n == 1:
	return 1
    else:
	return fib(n-1) + fib(n-2)

print(fib(10))
```

So that's pretty impressive, certainly much more impressive than I could have imagined a few years ago.
But the real power of this approach comes from iterating on the code by progressively editing it, taking the output and setting it as the next input.
For example:

```python
#Input: "[Previous Input]"
#Instructions: "Convert to using memoization with diskcache"

#Output:
import diskcache

cache = diskcache.Cache('/tmp/fib')

@cache.memoize()
def fib(n):
    if n == 0:
	return 0
    elif n == 1:
	return 1
    else:
	return fib(n-1) + fib(n-2)

print(fib(100))
```

```python
#Input: "[Previous Input]"
#Instructions: "Write a comment stating the time-complexity of this approach"

#Output:
#[Previous Output, plus...]
# This approach has a time-complexity of O(n)
```

Now if you do the same instruction for the recursive approach, the comment reads 'This is an exponential-time algorithm". Interesting!


### Writing Boilerplate {#writing-boilerplate}

Now let's try a question that's more related to something I might use this for,
saving time on plotting code.

```python
#Input: "
import matplotlib.pyplot as plt
import numpy as np

xs = np.random.normal(size=(1000,))

plt.hist(xs)
plt.show()
#Instructions: "Make the histogram bar edges thicker and add a title saying #"Histogram of samples from a unit Gaussian""

#Output:
#[Same as before, with diff]
plt.hist(xs, edgecolor='black', linewidth=1.2)
plt.title("Histogram of samples from a unit Gaussian")
```

While this worked very well for specific instructions such as "make the histogram bar edges thicker", it didn't work very well for more vague requests like "make the plot look better", "make the plot publication-quality", or "make the plot higher-quality". For all those instructions, the output was exactly the same as the input. While this level of abstraction might be out of reach, it is able to comprehend a reasonably-complicated query where we ask to sort out an issue with colorblindness:

```python
#Input:
import matplotlib.pyplot as plt
import numpy as np

x0s = np.random.normal(size=(1000,))
x1s = np.random.normal(size=(500,)) * 2 - 3
plt.hist(x0s, color="blue", edgecolor="black", linewidth=1.2)
plt.hist(x1s, color="orange", edgecolor="black", linewidth=1.2)
plt.title("Histogram of samples from a unit Gaussian")
plt.ylabel("Number of samples")
plt.show()

#Instructions: "Make the two classes distinguishable to people with colorblindness"

#Output:
#[Same as before, with diff]
plt.hist(x0s, color="blue", edgecolor="black", linewidth=1.2)
plt.hist(x1s, color="orange", edgecolor="black", linewidth=1.2)
```

However, the model did not manage to make this change when the colors were represented by the rgb codes `(1, 0, 0)` and `(0, 1, 0)` respectively.


### Bug-Fixing {#bug-fixing}

What about bug-fixing? Let's start with a simple one

```python
#Input:
#!/usr/bin/python3

def notify(x):
    print x

#Instructions: "Fix the bug"

#Output:
#[Same as before, with print x replaced by print(x)]
```

If I omit the shebang at the top, it's not clear what \`fix the bug\` means, so the model doesn't give the right result. However, if I use the instruction "Fix the bug [missing parentheses..]", which is the error that python3 gives, we get the same output. This shows that the model will probably do better with more specific context. If you add on "and write a comment explaining the solution", we get this comment:

> The problem was that the print function was missing parentheses.
> The print function is a function, so it needs to be called with parentheses.
> The print statement is a statement, so it doesn't need parentheses.
> The print statement was removed in Python 3.

which frankly perfectly describes the problem. If I then ask "Add a web link where I can read more", I get [this link](https://docs.python.org/3/reference/simple%5Fstmts.html#the-print-statement), which is actually a valid link. However, it doesn't say anything about the print statement (if we replace the 2 in the link with a 3, we do get a valid and informative link).

Let's try a more complicated bug based on [implicit string concatenation](https://peps.python.org/pep-3126/). The following is the input:

```python
def fruit_calculator(fruit_1: str, fruit_2=None):
    if fruit_1.lower() == "apple":
	fruit_cost = 1
    elif fruit_1.lower() == "orange":
	fruit_cost = 2
    else:
	raise NotImplementedError(f"{fruit_1} is not a fruit")

    if fruit_2 is not None:
	if fruit_2.lower == "banana":
	    fruit_cost += 5
    return fruit_cost


fruit_cost = fruit_calculator("apple"
		 "banana"
		 )


print(fruit_cost)
```

The bug is that I've missed a comma after "apple". Due to the python string concatenation, this means `fruit_calculator` is called with the single argument "applebanana". This isn't caught by my type checker, because calling `fruit_calculator` with a single fruit is perfectly allowed. Putting this into the API with the instruction "Fix the bug" does indeed fix the bug, adding a comma between the two fruits.


### Final thoughts {#final-thoughts}

There are a huge number of questions raised by this editing model, which I think indicates that code written by language models will soon be very common. I do not see people writing their own boilerplate code in ten years, for instance. Since these models are fairly new, I expect the progress to be pretty swift. I can already see quite a few potential gains by combining the current approach with type checkers, linters, formatters, etc. Indeed, If you have a problem that is strictly determined by the ability to pass certain test-cases, it seems possible to simply generate a huge number of sample codes with these algorithms and check to see which of them solve the problem. I expect to have some further thoughts after using these models for a while.


## Implementation in Emacs {#implementation-in-emacs}

I was able to write this post quite quickly by wrapping the API call into an emacs macro. Now, when I hit Control-c, Control-x, Control-e, I'm prompted for an instruction and the current region (if it's active) is sent to the OpenAI API. The result then replaces the current region (if it's active) or is inserted-at-point.

To do this, I have a helper python file, `codex_0.py`:

```python
#!/Users/chris/miniconda3/envs/python3/bin/python
import os
import openai
import sys

openai.api_key = os.getenv("OPENAI_API_KEY")

response = openai.Edit.create(
  engine="code-davinci-edit-001",
  input=f"{sys.argv[1]}",
  instruction=f"{sys.argv[2]}",
  temperature=0,
  top_p=1
)
response_text = response.choices[0].text
sys.stdout.write(response_text)
```

Then in my `.emacs` file I put

```emacs-lisp
(setenv "OPENAI_API_KEY" "[REDACTED]")


(setq codex-helper-location "/Users/chris/.emacs.d/codex_0.py")

(defun maybe-get-region ()
  "If region is active, return region, else return the empty string."
(if (use-region-p)
      (buffer-substring-no-properties (region-beginning) (region-end)) '""))

(defun get-codex-output (filepath input instructions)
  "Call the codex API to get the 'edit completion'.
FILEPATH is the path to the python file calling the API
INPUT and INSTRUCTIONS are the strings that will go into the respective API fields"
  (shell-command-to-string (format "python %s '%s' '%s'" filepath input instructions)))

(defun codex-edit (instructions)
  "Interactively asks for INSTRUCTIONS, combines with region or empty string."
  (interactive "sInstructions:")
  (let ((resulting-text (get-codex-output codex-helper-location (maybe-get-region) instructions)))
    (if (use-region-p)
	(progn
	  (kill-region (region-beginning) (region-end))
	  (insert resulting-text))
	(insert resulting-text))))

(global-set-key "\C-c\C-x\C-e" 'codex-edit)
```

And it's that simple!
