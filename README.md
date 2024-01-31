# Can we trust LLMs-as-a-judge?
*caution. this is an ongoing experiment. the readme is updated continously with new results. i'm actively looking for suggestions!*


Evaluating LLMs in a open-ended scenario is difficult, there's a growing consensus that existing benchmarks are lacking and seasoned practitioners prefer to _vibe check_ models themselves. I've resorted to anecdotal evaluations from developers and researchers I trust, and [Chatbot Arena](https://arena.lmsys.org/) as an excellent complement. The motivation behind this repo is the increasingly popular method of using strong LLMs as a judge for models. This method has been around for a few months, with models such as [JudgeLM](https://github.com/baaivision/JudgeLM), and more recently [MT-Bench](https://arxiv.org/pdf/2306.05685.pdf).

You may or may not have seen [this thread](https://twitter.com/aparnadhinak/status/1748368364395721128). According to the authors of the tweet at Arize AI, using LLMs-as-a-judge warrants server caution, specifically in regards to the use of numeric score evaluations. It seems that LLMs are very poor at handling continuous ranges, which becomes glaringly obvious when prompting them to evaluate **X** from 1 to 10. This repo is a living document of experiments attempting to understand, capture and perhaps even find an alternative solution. From the limited time I spent researching this problem I found numerous papers using LLMs as a judge in this specific manner and it worries me that a lack of understanding of this *jagged frontier* will hinder continued success.

## Key Findings
T.B.D

## Full Experiment 
Below are the full details and results.

### Methodology

Due to cost constraints, I'll initially focus on the spelling/misspelling task described in the tweets. I'm slightly worried that the quantitative X of this task is going to contaminate the insights of this experiment, but we'll see.  

#### Spelling Dataset

I've generated a spelling or misspelling dataset, not sure which name is more appropriate, from the essays of Paul Graham. This choice was mostly out of convenience as I've used the dataset before when [pressure testing context windows](https://github.com/LeonEricsson/llmcontext). I extracted a context of 3,000 words from the essays and insert spelling errors on random words based on the desired misspelling ratio. In pseudocode:

```
misspell_ratio

words = split context into words
misspell_count = calculate number of words to misspell based on ratio

FOR word = sample(words, misspell_count)
    IF length(word) > 3
        extract random character
    ELSE:
        add random character
END FOR

```

The complete code is readily available as a [notebook](/dataset.ipynb).

#### LLM Evaluator

Given the generated dataset, we prompt LLMs to evaluate the amount of misspelled words in a context using different scoring templates. We're using the following APIs

GPT-4: `gpt-4-0125-preview`

GPT-3.5: `gpt-3.5-turbo-1106`

with temperature = 0. 

### Results
First, let's confirm that LLMs struggle to handle numeric ranges in a zero-shot setting. We prompt GPT-3.5 and GPT-4 with a numeric scoring template, ranging from score 0 to score 10. 

![](/figures/scoring_1_10.png)

As expected, both misjudge severely.

--- 

What happens if we reverse the scoring range? Now, a score of 10 represents a perfectly spelt document.

![](/figures/gpt-3.5_scoring_1_10_reversed.png)
![](/figures/gpt-4_scoring_1_10_reversed.png)

This doesn't seem to make much of a difference.

--- 

What about Chain-of-Thought? 

![](/figures/gpt-3.5_scoring_1_10_cot.png)
![](/figures/gpt-4_scoring_1_10_cot.png)

This definitely seems to improve things. We're getting closer to a linear correlation.

---

As [suggested](https://twitter.com/seungonekim/status/1749289437165769177) by the author of [Prometheus](https://arxiv.org/pdf/2310.08491.pdf): let's make sure to map an explanation to each score in the prompt. Combine this with CoT.

![](/figures/gpt-3.5_scoring_1_10_cot_full.png)
![](/figures/gpt-4_scoring_1_10_cot_full.png)

