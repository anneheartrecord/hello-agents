# Chapter 3 Reference Answers

> These reference answers are for reference only. The Chapter 3 exercises span model principles, engineering practice, and system design. It is best to study the conceptual explanation and the implementation-oriented reasoning together.

## 1. From Bigram to RNN/LSTM and Transformer

### Compute the Bigram probability of `agent works` using the mini corpus

The corpus is:

```text
datawhale agent learns
datawhale agent works
```

If we define sentence probability in the standard way, we usually add a sentence-start token `<s>`:

```text
<s> datawhale agent learns </s>
<s> datawhale agent works </s>
```

Then:

```text
P(agent works) = P(agent | <s>) x P(works | agent)
```

Counts:

- `C(<s>, agent) = 0`
- `C(<s>) = 2`
- `C(agent, works) = 1`
- `C(agent) = 2`

So:

```text
P(agent | <s>) = 0 / 2 = 0
P(works | agent) = 1 / 2 = 0.5
P(agent works) = 0 x 0.5 = 0
```

Conclusion: under an unsmoothed Bigram model, the sentence probability of `agent works` is `0`.

This directly reveals the sparsity problem of N-gram models. Even though `works` can indeed follow `agent`, the training corpus never saw the bigram `<s> -> agent`, so the model treats the whole sentence as impossible.

Additional note:

- If the question only asks for the local conditional probability `P(works | agent)`, then the answer is `1/2`.
- If it asks for sentence probability in the standard language-model sense, the answer is `0`.

### What does the Markov assumption mean?

The Markov assumption says that the probability of the next word depends only on a limited number of previous words, not on the full history.

For example:

- Unigram: `P(w_n)`
- Bigram: `P(w_n | w_{n-1})`
- Trigram: `P(w_n | w_{n-2}, w_{n-1})`

In essence, it approximates the true full-history dependency of language using a fixed-size context window.

### Fundamental limitations of N-gram models

- Sparsity: the number of possible high-order N-grams grows combinatorially, and many combinations never appear in data.
- Weak long-range dependency modeling: information outside the fixed window is lost.
- No semantic understanding: the model counts co-occurrences of strings without understanding meaning.
- High storage cost: high-order N-grams need large tables to store counts.

### How do RNN/LSTM and Transformer overcome these limitations?

#### RNN/LSTM

The key idea of RNNs is to use hidden state to carry historical information instead of relying on a fixed window.

- In principle, the hidden state can summarize arbitrarily long context.
- Word embeddings allow semantically similar words to have nearby vector representations, improving generalization.

However, plain RNNs suffer from vanishing gradients. LSTMs alleviate this using gates and a separate memory cell, which helps preserve useful context over longer spans.

Advantages of RNN/LSTM:

- Better sequence-history modeling than N-grams
- Stronger semantic generalization than purely statistical models

#### Transformer

The key breakthrough of Transformer is self-attention.

- Any two positions can directly interact.
- It does not rely on sequential recurrence, so it can be trained in parallel.
- Each token gets a dynamic contextual representation, so the same word can be represented differently in different contexts.

Advantages of Transformer:

- Stronger long-range dependency modeling
- Much higher training parallelism, which supports large-scale scaling
- Better foundation for strong context understanding and generation

## 2. Core mechanisms of the Transformer

### What is the core idea of self-attention?

In one sentence: each position in the sequence dynamically measures its relevance to all other positions and then aggregates information accordingly.

It is usually implemented using three vectors: `Q`, `K`, and `V`.

- Query: what information the current position is looking for
- Key: what information each position can offer
- Value: the actual content of each position

The usual computation is:

```text
Score = Q x K^T / sqrt(d_k)
Attention = softmax(Score)
Output = Attention x V
```

Intuitively, when a word tries to understand itself, it looks across the sentence, identifies which other words matter most, and absorbs their information with learned weights.

### Why can Transformer run in parallel while RNN must be sequential?

RNN is recursive:

```text
h_t = f(h_{t-1}, x_t)
```

So step `t` depends on step `t-1`, which creates a strict serial dependency.

Transformer does not depend on the previous hidden state in that way. It processes the full input using matrix operations, so all positions can be computed in parallel. That is a major reason it can scale efficiently on GPUs.

### What does positional encoding do?

Self-attention by itself does not understand order. For it, the difference between "dog chases cat" and "cat chases dog" is not inherently encoded if only token identity is used.

So positional encoding injects sequence order into the model. Common forms include:

- Sinusoidal positional encoding in the original Transformer
- Learned positional encoding
- RoPE, which is widely used in modern LLMs

### What is the difference between Decoder-Only and Encoder-Decoder?

#### Encoder-Decoder

- The encoder uses bidirectional attention to understand the input.
- The decoder generates output while attending both to previous output tokens and to the encoder representation.
- This is well-suited for explicit sequence-to-sequence tasks like translation and summarization.

#### Decoder-Only

- It uses only causal, one-directional attention.
- In essence, it turns all tasks into next-token completion conditioned on previous context.
- It is well-suited for general generation, dialogue, and code completion.

### Why do mainstream LLMs mostly adopt Decoder-Only?

- The training objective is unified and simple.
- Every token directly contributes training signal, which improves training efficiency.
- It naturally supports in-context learning, turning translation, QA, and summarization into completion problems.
- Industry scaling-law evidence has been especially strong on this architecture.

## 3. Why not use characters or words directly, and what problem does BPE solve?

### Problems with character-level input

- Sequences become very long, increasing training and inference cost.
- Individual characters carry weak semantics.
- Long documents waste context budget because too many tokens are spent on tiny units.

### Problems with word-level input

- The vocabulary becomes huge, which increases parameter cost.
- Out-of-vocabulary problems are severe for rare words, new words, typos, and proper nouns.
- Word variants cannot share structure well, for example `run`, `running`, and `ran`.

### What BPE solves

BPE starts from characters and repeatedly merges the most frequent adjacent symbol pairs, gradually forming frequent subwords.

This gives a practical middle ground between characters and full words:

- Frequent common words can stay as whole tokens for efficiency.
- Rare words can be decomposed into subwords, avoiding hard OOV failures.
- Related words can share substructure, which improves generalization.

For example:

```text
understanding -> under + stand + ing
unhappiness -> un + happiness
```

So BPE fundamentally solves the granularity problem: characters are too fine, full words are too coarse.

## 4. Local deployment of open-source models and prompt strategy comparison

> This is a practice-oriented question. What follows is a reproducible reference workflow plus example conclusions. Actual outcomes depend on hardware, model version, and prompt details.

### Deploy a lightweight local model

Using `Qwen3-0.6B` as an example, you can load it with `transformers`:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "Qwen/Qwen3-0.6B"
tokenizer = AutoTokenizer.from_pretrained(model_name, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    trust_remote_code=True,
    device_map="auto"
)

def generate(prompt, temperature=0.7, top_p=0.9, top_k=50, max_new_tokens=256):
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    outputs = model.generate(
        **inputs,
        max_new_tokens=max_new_tokens,
        temperature=temperature,
        top_p=top_p,
        top_k=top_k,
        do_sample=True
    )
    return tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
```

### Effects of sampling parameters

| Parameter | Low-value behavior | High-value behavior |
|-----------|--------------------|---------------------|
| temperature | More stable and conservative | More diverse and more erratic |
| top_p | Tighter candidate set | Looser candidate set |
| top_k | Chooses only among a few top candidates | Uses a wider candidate pool |

Practical rule of thumb:

- Precise extraction or code generation: `temperature=0.1-0.3`
- General question answering: `temperature=0.5-0.8`
- Creative generation: `temperature=0.8-1.2`

### Comparing prompt strategies

Take sentiment classification as an example:

- Zero-shot: cheapest in prompt length, but small models often drift in format.
- Few-shot: a few examples usually provide the best cost-performance gain for small models.
- Chain-of-Thought: often helps on more complex reasoning tasks, but on small models it can sometimes create more opportunities to go off track.

A simple summary:

- Small models usually benefit most from Few-shot prompting.
- Larger models benefit more from CoT on genuinely reasoning-heavy tasks.

### Closed-source vs open-source models

| Dimension | Closed-source models | Open-source models |
|-----------|----------------------|--------------------|
| Performance ceiling | Usually stronger today | Catching up quickly |
| Ease of use | API and go | Needs deployment and operations |
| Cost structure | Pay per token | Higher upfront hardware cost, lower long-run marginal cost |
| Privacy | Data goes to a third party | Can be kept on-premise |
| Controllability | Constrained by provider | More freedom in versioning, fine-tuning, and quantization |
| Compliance | Restricted in some industries | Better for high-compliance environments |

### What kind of model should an enterprise customer-service agent use?

For enterprise customer service, I would generally prefer: private deployment of an open-source model + RAG + safety review + human escalation.

Important decision factors:

- Data privacy and regulatory compliance
- Daily call volume and long-term cost
- Whether latency must remain tightly controlled
- Whether the system must learn enterprise-private knowledge
- Whether style fine-tuning is needed
- Whether there is a safe human fallback path

## 5. Methods for mitigating hallucination

### Explain one method and its applicable scenarios

Here I choose RAG.

The basic RAG pipeline is:

1. Convert the user question into a vector.
2. Retrieve relevant document chunks from an external knowledge base.
3. Feed both the retrieved evidence and the question into the model.
4. Let the model answer based on retrieved evidence.

Why it reduces hallucination:

- It shifts the model away from pure memory and toward evidence-grounded answering.

Typical use cases:

- Enterprise knowledge QA
- Customer support
- Legal or medical assistance
- News, current events, and academic question answering

### Other frontier approaches

Besides RAG, hallucination can also be reduced from three layers: inference, training, and output design.

- Self-Consistency: sample multiple reasoning paths and vote on the most consistent final answer.
- Chain-of-Verification: answer first, then generate verification questions and self-check.
- RLHF/DPO: train the model to prefer honest, cautious, and better-grounded responses.
- Inference-Time Intervention: intervene in internal activations during generation.
- Attributed Generation: require key factual claims to cite sources.

Their main differences are:

- Whether they rely on external knowledge bases
- Whether they require retraining
- How much inference cost they add
- Whether they are better for factual tasks or reasoning tasks

## 6. Designing a paper-reading assistant agent

### How would I choose the base model?

I would prioritize:

- Long-context capability, ideally starting at 32K and preferably 128K for strong full-paper handling
- Strong reasoning over methods, experiments, and argument chains
- Strong English paper understanding plus bilingual QA ability
- Strong instruction following and faithfulness, so the model does not invent claims not present in the paper

For individuals or small teams, a strong closed-source model is often the lowest-friction choice. For institutions or privacy-sensitive products, strong open-source self-hosted models are more attractive.

### How should prompts be designed, and how should long papers be handled?

Prompt design should exploit the natural structure of papers, such as:

- What research problem is being solved?
- What are the core contributions?
- How does the method work?
- What are the key empirical results?
- What limitations are discussed?

At the same time, the prompt should impose hard constraints:

- Answer only based on the paper.
- If the paper does not say something, explicitly say it is not mentioned.
- Cite the relevant section, paragraph, or table when giving conclusions.

For very long papers, three common solutions are:

- Map-Reduce: summarize by section, then summarize the summaries.
- Hierarchical QA: first read abstract, introduction, and conclusion, then dive into relevant sections on demand.
- RAG: segment the paper, index it, and retrieve the most relevant chunks for each question.

In a real product, I would most often recommend RAG because it uses context budget efficiently and supports source grounding.

### How do we ensure the output is accurate, objective, and faithful to the paper?

The system should include at least these design elements:

1. Source attribution: each key claim should point to a section, paragraph, or table.
2. Faithfulness prompt constraints: explicitly prohibit invention and overreach.
3. Double verification: key numerical results and experiment details should be rechecked against the source text.
4. Hallucination detector: automatically check whether factual claims are supported by the paper.
5. Multi-paper anti-confusion markers: when comparing papers, assign each paper a unique label to prevent cross-paper mix-ups.

From an engineering perspective, the most important property of a paper-reading agent is not that it sounds like an expert. It is that every important statement can be traced back to the source paper.
