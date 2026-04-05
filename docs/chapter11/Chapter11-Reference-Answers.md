# Chapter 11 Reference Answers

> These reference answers are for reference only. The main lesson of this chapter is not memorizing training acronyms, but understanding why once agents enter multi-step decision-making, tool use, and delayed feedback settings, training must move from one-step prediction toward sequence optimization.

## 1. The shift from LLM training to Agentic RL

### Why does Agentic RL include historical observations in state while PBRFT often starts from only the prompt?

In PBRFT-like settings, the model is usually treated as: given a prompt, produce an answer. Optimization is centered on the initial input and final output.

In Agentic RL, the agent keeps receiving new observations during execution, such as:

- tool outputs
- environment changes
- intermediate failure signals

That means decisions are trajectory-conditioned rather than one-shot. Without historical observations in state, the policy cannot adapt to what happened earlier.

Main consequences:

- training becomes harder because the state and trajectory spaces are larger
- the learned capability becomes stronger because the model learns adaptation during execution, not just static response generation

### Mapping an intelligent code debugging assistant to RL

A reasonable formulation is:

- state space `S`: user request, current code snippet, tool outputs so far, edits attempted, test results
- action space `A`: read file, search docs, run test, modify code, request more context, finish task
- reward `R`: large reward for passing tests, medium reward for partial repair, penalties for regressions, wasted steps, or harmful edits
- transition `T`: each action changes repository state, test state, and observations

This is an interactive control problem, not a one-shot QA task.

### Why is supervised learning weak at optimizing long-horizon objectives?

Consider mathematical proof generation:

- supervised learning sees reference proofs and learns to imitate them
- intermediate reasoning steps do not have one canonical form
- a locally plausible step may make the proof impossible several steps later

Reinforcement learning can delay reward until the final proof succeeds and then credit or blame the whole trajectory, making it better suited to long-horizon optimization.

## 2. SFT, LoRA, and GRPO

### What is the core idea of LoRA?

LoRA is not just “train fewer parameters.” Its real assumption is that useful downstream updates can often be approximated by low-rank changes.

The method freezes most base-model weights and trains only small low-rank matrices that approximate the needed update directions.

Why it works:

- the pretrained model already contains strong general representations
- many downstream tasks do not require rewriting the full parameter space
- low-rank updates capture the most important adjustment directions cheaply

Choose LoRA when compute is limited, data is modest, or many fast experiments are needed.

### What advantages does GRPO have over PPO?

`PPO` usually brings heavier training machinery, often including value estimation or more complex baselines.

`GRPO` simplifies optimization by comparing candidates within the same sampled group and using relative reward inside that group.

Benefits:

- simpler training pipeline
- better robustness to reward-scale issues
- natural fit for generation tasks where multiple candidate outputs are easy to sample

When adapting it to code generation or dialogue optimization, the main changes are:

- replace the reward function with task-specific reward
- design candidate diversity carefully
- control length and formatting bias

### How should the SFT pipeline be extended?

Add three capabilities:

1. multi-turn dialogue training using structured `messages`
2. data augmentation such as paraphrasing and difficulty-aware sampling
3. visualization of loss, gradients, length distribution, and sampled output quality

The goal is not decoration. It is better interpretability of both data and training dynamics.

## 3. Reward design and defenses against reward hacking

### A more fine-grained reward for math problems

One useful decomposition is:

```text
reward = 0.5 * final_correctness
       + 0.2 * step_validity
       + 0.2 * partial_progress
       - 0.1 * verbosity_penalty
```

Meaning:

- `final_correctness`: whether the final answer is right
- `step_validity`: whether the reasoning steps are coherent
- `partial_progress`: credit for correct intermediate progress
- `verbosity_penalty`: penalty for uselessly long solutions

### Reward functions for three task types

1. Code generation assistant:
   test pass rate, static analysis quality, runtime efficiency, readability.

2. Customer-service dialogue agent:
   resolution rate, user satisfaction, response time, compliance.

3. Game AI:
   win rate, strategy diversity, robustness against different opponents.

A reward function should not over-focus on one headline metric, or the policy will learn pathological shortcuts.

### Examples of reward hacking and defenses

Examples:

- a coding agent edits the tests instead of fixing the code
- a support agent ends conversations quickly with useless canned replies to optimize latency
- a math agent generates bloated template reasoning to game process-based reward

Defenses:

- decompose reward so no single metric dominates
- add hard constraints, such as forbidding test modification
- run adversarial evaluation looking for high-score cheating behaviors
- add human review and rule-based checks for suspicious cases

## 4. Extending the math-reasoning training case

### What is GSM8K good for, and how should it be extended?

GSM8K mainly contains elementary-to-middle-school word problems. It is good for:

- multi-step arithmetic reasoning
- decomposition plus calculation
- tasks with clean final answers and easy automatic grading

To train more advanced capability, extend:

- datasets toward advanced math, proofs, and olympiad-style questions
- tool use with symbolic calculators or proof checkers
- rewards beyond final answer, including proof structure and rigor

### How should generalization be evaluated?

To test whether the model learned reasoning rather than memorization, use:

- paraphrased variants of the same concept
- out-of-distribution harder chains
- distractor-heavy variants with irrelevant information

To improve generalization:

- regularization and early stopping
- data augmentation and template diversity
- curriculum-style training across difficulty bands

### What are the challenges of online learning?

Online learning can continuously absorb user feedback, but it must handle:

- noisy or unreliable feedback
- catastrophic forgetting
- safety risks from malicious or low-quality data
- rollout, canary, and rollback requirements

In practice, the safer pattern is often “collect online, filter offline, update in controlled batches.”

## 5. Teaching agents to use tools

### A tool-learning training plan

Given tools such as search, calculator, and code execution, the objective is:

- use a tool when needed
- avoid unnecessary tool use
- choose the correct tool
- supply correct parameters

Reward can combine:

- tool-selection correctness
- parameter correctness
- final task completion
- call efficiency

### A hierarchical RL plan for tool use

Split the policy into two levels:

- high-level policy chooses the subgoal, such as “search documentation” or “run tests”
- low-level policy chooses the concrete tool and parameters

Training-wise, the high level receives task-level rewards while the low level receives immediate tool-use feedback. The two levels are coordinated through explicit subgoal interfaces.

### Curriculum learning when there are many tools

A good curriculum is:

1. single-tool simple tasks
2. two-tool chained tasks
3. multi-tool low-dependency tasks
4. multi-tool long-chain high-dependency tasks

Before advancing stages, check:

- stable success rate above threshold
- decreasing tool-misuse rate
- converging average call count

Curriculum learning works because it controls the exploration space instead of forcing the policy to blindly search across 50+ tools from day one.
