# LLMs-as-a-judge : Exploring limitations and capabilties
*caution. this is an ongoing experiment. the readme is updated continously with new results. i'm actively looking for suggestions!*


Evaluating LLMs in a open-ended scenario is difficult, there's a growing consensus that existing benchmarks are lacking and seasoned practitioners prefer to _vibe check_ models themselves. I've resorted to anecdotal evaluations from developers and researchers I trust, with [Chatbot Arena](https://arena.lmsys.org/) being an excellent complement. The motivation behind this repo is the increasingly popular method of using strong LLMs as a judge for models. This method has been around for a few months, with models such as [JudgeLM](https://github.com/baaivision/JudgeLM), and more recently [MT-Bench](https://arxiv.org/pdf/2306.05685.pdf).

You may or may not have seen [this thread](https://twitter.com/aparnadhinak/status/1748368364395721128). According to the authors of the tweet at Arize AI, using LLMs-as-a-judge warrants server caution, specifically in regards to the use of numeric score evaluations. It seems that LLMs are very poor at handling continuous ranges, which becomes glaringly obvious when prompting them to evaluate **X** from 1 to 10. This repo is a living document of experiments attempting to understand and capture the *jagged frontier* of this problem. Recent [work](https://twitter.com/gblazex/status/1746295870792847562) has established **a strong correlation between MT-Bench and Human Judgment (Arena Elo)**, meaning that LLMs are capable of being judges, so what's going on here?

## Key Findings
T.B.D

## Full Experiment 
Below are the full details and results.

### Methodology

Due to cost constraints, I'll initially focus on the spelling/misspelling task described in the tweets. I'm slightly worried that the quantitative X of this task is going to contaminate the insights of this experiment, but we'll see. I welcome a more full flegded analysis of this phenomena, my results should be taken with a grain of salt given the limited experiment

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

at temperature = 0. 

### Results
**Test 1.** Let's confirm that LLMs struggle to handle numeric ranges in a zero-shot setting. We prompt GPT-3.5 and GPT-4 with a numeric scoring template, ranging from score 0 to score 10. 

![](/figures/scoring_1_10.png)

As expected, both misjudge severely.

--- 

**Test 2.** What happens if we reverse the scoring range? Now, a score of 10 represents a perfectly spelt document.

![](/figures/scoring_1_10_reversed.png)

This doesn't seem to make much of a difference.

--- 
**Test 3.** If we were to believe the hypothesis from Arize, we may see improvements if we avoid a scoring rubric and instead use 'labeled grades'. In this case I decided to move down to a 5-point grading scale.

![](/figures/scoring_grades.png)

Perhaps slight improvements? Difficult to say honestly. I'm not impressed.

---

**Test 4.** What about zero-shot Chain-of-Thought? 

![](/figures/scoring_1_10_cot.png)

gpt-3.5 devolved into gibberish for two of the prompts. As expected, gpt-4 sees improvement when prompted to think-out-loud. Notice how it get's very
hesitant to assign a score of 10. 

---

**Test 5.** As [suggested](https://twitter.com/seungonekim/status/1749289437165769177) by the author of [Prometheus](https://arxiv.org/pdf/2310.08491.pdf); mapping each score with it's own explanation likely improves the LLMs ability to grade across the entire numeric
range. This, combined with CoT, results in:

![](/figures/scoring_1_10_full_cot.png)

Continued improvements for gpt-4. It's still very relucent to assign boundary scores 0 & 10. 


## Discussion (free-form, continuously updated)

#### MT Bench
I've been going through the internals of MT-Bench, and was very surprised to find they simply ask GPT-4 to score outputs on a scale of 1-10. They do supply alternative grading options such as pairwise comparisons against a baseline but the recommended option is the numeric one. The judgement prompt is also unexpectedly simple: 

*Please act as an impartial judge and evaluate the quality of the response provided by an AI assistant to the user question displayed below. Your evaluation should consider factors such as the helpfulness, relevance, accuracy, depth, creativity, and level of detail of the response. Begin your evaluation by providing a short explanation. Be as objective as possible. After providing your explanation, you must rate the response on a scale of 1 to 10 by strictly following this format: [rating], for example: "Rating: 5". [Question] {question} [The Start of Assistant's Answer] {answer} [The End of Assistant's Answer]*

If one is to believe that this is all there is to judging in MT-Bench, then I'm beginning to question the use of the misspelling task as a proxy task... 
