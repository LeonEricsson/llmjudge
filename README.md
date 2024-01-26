# llmjudge

Evaluating LLMs in a open-ended scenario is difficult, there's a growing consensus that existing benchmarks are lacking and seasoned practitioners prefer to _vibe check_ models themselves. I've resorted to anecdotal evaluations from developers and researchers I trust, and [Chatbot Arena](https://arena.lmsys.org/) as an excellent complement. The motivation behind this repo is the increasingly popular method of using strong LLMs as a judge for models. This method has been around for a few months, with models such as [JudgeLM](https://github.com/baaivision/JudgeLM), and have seen increased attention with [MT-Bench](https://arxiv.org/pdf/2306.05685.pdf).

It turns out however that using LLMs as a judge warrants severe caution, especially in regards to the use of numeric score evaluations. As highlighted through some great work out of [Arize AI](https://twitter.com/aparnadhinak/status/1748368364395721128) it seems that LLMs are frankly quite horrible at handling continuous ranges, which bodes poorly when you're asking your LLM to score an output between for example 1 and 10. To me this is extremely alarming, as the thought of asking LLMs to judge something based on a numeric range seems intuitively straight-forward and something that lies within it's _jagged frontier_? Aparna illustrated some excellent points in the Twitter thread and I'm certain that this has implications published research, current research and just everyday use. They did a great job pointing out the problems with numeric evaluation and I want to further this exploration, comparing numeric evaluations to alternatives!

## Goal

Explore the findings of [Arize AI](https://twitter.com/aparnadhinak/status/1748368364395721128) further, what are some viable alternatives to numeric scoring. I'll carry out my experiments using a similar Spelling evaluation and with both gpt-4 and gpt-3.

- Confirm the results established by Arize; are LLMs really that bad at handling numeric ranges?
- Establish alternative evaluation templates.
- Compare the capabilities of GPT-4 and GPT-3 in regards to such templates.

## Methodology

### Spelling Dataset

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

The complete code is readily available in as a [notebook](/dataset.ipynb). You're free to create your own dataset.

### LLM Eval Scores

Given the generated dataset, we prompt LLMs to evaluate the amount of misspelled words in a context using different scoring templates. The default template is the illustrated by Arize, with a score of 10 indicating 100% misspelled words and a score of 1 being 0% misspelled words.

## Results
