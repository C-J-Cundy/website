+++
title = " Running GPT-J On Several Smaller GPUs"
author = ["Chris Cundy"]
draft = false
date = "2021-06-24T13:00:00+00:00"
+++

## Introduction {#introduction}

Recently several large language models have been open-sourced. Particularly interesting is [GPT-J](https://6b.eleuther.ai/), which
has completely open-sourced weights and provides pre-trained weights. The model itself has performance
comparable to the smallest version of GPT3.

However, the model is pretty big, sitting at around 24GB in GPU memory. Most of our GPUs don't have this
much memory. If you're in this boat too, here's a quick method to try out GPT-J on your GPUs, if you have
e.g. several 10GB GPUs. Note: this is very much not-optimized, and a bit hacky, but sufficient to experiment a bit with this cool
language model!


## Method {#method}

1.  [Follow this notebook](https://github.com/paulcjh/gpt-j-6b/blob/main/gpt-j-t4.ipynb) from [Paul Hetherington](https://www.getneuro.ai/) to download the GPT-J slim weights and turn them into a pytorch binary weights file. Once this is done, copy the weights file into the `gpt-j-hf` folder with the `config.json` file that the notebook also downloads.
2.  Install [this fork](https://github.com/C-J-Cundy/transformers) of the [🤗 Transformers](https://huggingface.co/) library, on the `gpt-j` branch. I'm not sure how this will interact with existing installations of the transformers library, so best to do this on a fresh conda env. You can install with `pip install --upgrade git+https://github.com/C-J-Cundy/transformers@gpt-j`. Note that on this fork, we essentially replace the old GPT-Neo with GPT-J.
3.  Finally, load GPT-J as follows:

<!--listend-->

```python
import torch
from transformers import GPTNeoForCausalLM, GPT2Tokenizer
from torch import nn
import time

def filtered_to_cuda(model: nn.Module) -> None:
    """Does model.cuda(), but doesn't reassign tensors that are already on gpu"""

    def conditional_cuda(x):
	if x.device.type != "cuda":
	    return x.to("cuda:0")
	else:
	    return x

    return model._apply(conditional_cuda)

model = GPTNeoForCausalLM.from_pretrained("./gpt-j-hf")
model = filtered_to_cuda(model.half())
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

input_text = "Thanks very much to Paul for his awesome notebook"
input_ids = tokenizer.encode(str(input_text), return_tensors="pt").cuda()

t0 = time.time()
n = 200
output = model.generate(
    input_ids, do_sample=True, max_length=n, top_p=0.7, top_k=0, temperature=1.0,
)
print(tokenizer.decode(output[0], skip_special_tokens=True))
print(f"Took {time.time()- t0}s to generate {n} tokens")
```


## Explanation {#explanation}

What's going on here? Essentially we use a very naive form of [model parallelism](https://pytorch.org/tutorials/intermediate/model%5Fparallel%5Ftutorial.html), where we partition the parameters of the model and load them onto different GPUs. We split the transformer layers into \\(n\\) parts for our \\(n\\) GPUs. We then ensure that inside the forward method, we properly move the data onto the correct GPU. As implemented, this is quite wasteful, since only one GPU is actually doing anything at one time: we're using \\(n\\) GPUs with memory \\(r\\) and \\(k\\) flops to simulate a single GPU with \\(nr\\) memory and \\(k\\) flops. By using [pipeline parallelism](https://pytorch.org/docs/stable/pipeline.html) we can more closely approach \\(nr\\) memory and \\(nk\\) flops, although this seems potentially more fiddly to get working than the very naive approach described here.
