# Chapter 4 Reference Answers

> These reference answers are for reference only. The real goal of this chapter is not to memorize one paradigm, but to understand how different paradigms organize thinking, acting, observing, and reflecting, and what those choices imply in real systems.

## 1. ReAct, Plan-and-Solve, and Reflection: differences and selection

### Core differences

| Paradigm | Core loop | Key trait | Strengths | Limitations |
|----------|-----------|-----------|-----------|-------------|
| ReAct | Thought -> Action -> Observation | Think while acting | Reacts quickly to changing environments and tool outputs | Can become locally greedy and weak at global planning |
| Plan-and-Solve | Plan -> Execute -> Execute | Plan first, then execute | Better at satisfying global constraints in complex tasks | Static plans can break during execution |
| Reflection | Execute -> Reflect -> Refine | Produce a draft, then improve it | Strong for quality optimization and polishing | Higher latency and iteration cost |

### Which paradigm fits a smart-home assistant?

I would use `ReAct` as the base architecture.

Reasons:

- Smart home control is a strong tool-calling scenario: the system must directly operate light, AC, curtain, and speaker APIs.
- The environment is dynamic. After each action, the agent should observe whether the device actually changed state.
- User instructions are often contextual, for example, "turn that one off too," which requires ongoing interpretation while executing.

### Can the three paradigms be combined?

Yes, and in production systems they often should be.

A strong hybrid architecture is:

1. Use `Plan-and-Solve` to generate the high-level plan.
2. Use `ReAct` inside each step to execute with tools and adapt to observations.
3. Use `Reflection` before the final answer or risky action for quality or safety review.

This is well suited for refund processing, business travel booking, enterprise reporting, and complex customer-service workflows.

## 2. Fragility of regex parsing and better alternatives

### Why regex parsing is fragile

Regex parsing works only when output format stays fully stable. It breaks easily when the LLM output changes slightly, for example:

- Markdown code fences are added.
- Chinese and English colons are mixed.
- Field order changes.
- Multiple actions are emitted in one reply.
- The model adds explanatory prose before the structured part.

In other words, the failure is often not a reasoning failure but a protocol-design failure.

### Better alternatives

Recommended order:

1. `Function Calling / Tool Calling`: best choice, because tool invocation is returned in structured form.
2. `JSON Mode`: force the model to produce JSON and parse that.
3. `JSON + retry`: attempt resilient parsing, and ask the model to regenerate if needed.

### Comparison

| Method | Pros | Cons |
|--------|------|------|
| Regex | Simple and dependency-light | Extremely brittle under format drift |
| Function Calling | Stable structure and natural parameter validation | Requires model/API support |
| JSON Mode | Much more robust than regex | Can still produce invalid JSON |

Conclusion: regex is acceptable for teaching the idea, but real systems should prefer structured protocols.

## 3. Tool calling extensions: calculator, failure correction, and scaling to many tools

### Adding a calculator tool to ReAct

The core rule is: do not use `eval()`. Use AST-based whitelist parsing instead.

Implementation idea:

- Normalize mathematical symbols such as `× -> *`, `÷ -> /`, and Chinese parentheses.
- Parse the expression with `ast.parse(..., mode="eval")`.
- Only allow safe numeric nodes and mathematical operators.
- Return explicit errors for invalid syntax, division by zero, or disallowed nodes.

The essence is to shrink "execute arbitrary code" into "evaluate only a safe arithmetic subset."

### How should tool-selection failure be corrected?

Use a three-layer mechanism:

1. First failure: gentle correction, telling the model the tool or parameter is wrong.
2. Second failure: inject a more detailed schema, examples, and the previous error.
3. Third failure: circuit-break the tool-calling path and fall back to text response or human escalation.

The key is not infinite retry. The key is to feed structured failure information back into the next reasoning step so the model has a chance to self-correct.

### What if there are 50-100 tools?

Putting all tool descriptions into the prompt no longer scales. The main issues are:

- Too many tokens
- Confusion among similar tools
- Lower tool-selection accuracy

Engineering upgrades:

1. Organize tools by categories.
2. Use embeddings to retrieve the top 5-10 relevant tools before showing them to the model.
3. Use two-stage routing: select category first, then concrete tool.
4. For very large toolsets, train a dedicated tool router.

Once the toolset gets large, the problem stops being just prompt design and becomes a retrieval-and-routing systems problem.

## 4. Dynamic replanning, business-travel selection, and hierarchical planning

### How should dynamic replanning work?

Static plans always break in real environments, so failures should be handled by severity:

- Mild failure: parameter issues or temporary timeouts. Retry the current step.
- Moderate failure: the current path fails but the goal remains. Replan from the failure point onward.
- Severe failure: the goal itself is no longer achievable. Stop and explain to the user or ask for new constraints.

Key design principles:

- Preserve useful outputs from already completed steps.
- Do not reset global step or token budgets during replanning.
- Record failure information in structured form so the planner can use it.

### Business travel: Plan-and-Solve or ReAct?

For "book a business trip from Beijing to Shanghai including flight, hotel, and car rental," the best answer is `Plan-and-Solve + dynamic replanning`.

Why:

- There are strong cross-step constraints. Flight time affects hotel check-in, and hotel location affects rental logistics.
- Users usually want to review a complete feasible plan before the system executes it.
- Budget, time, and location constraints must be handled globally.

Pure ReAct risks local optimization, such as booking the cheapest flight first and only later discovering it breaks the rest of the itinerary.

### Why hierarchical planning helps

Use a high-level plan plus per-step subplans.

Benefits:

- Lower planning complexity per call
- Better error isolation
- Better support for parallel execution across submodules
- More efficient token usage

In essence, hierarchical planning separates strategy from tactics.

## 5. Reflection: two-model design, stopping conditions, and an academic-writing assistant

### What happens if execution and reflection use different models?

A good pattern is:

- A fast model does execution and rewriting.
- A stronger model does review and reflection.

Benefits:

- Lower total cost
- Lower average latency
- Better review quality

Risks:

- The reviewer may produce suggestions too abstract or too advanced for the executor to implement well.
- The two models may create style inconsistency and oscillation.

The engineering fix is to force the reviewer to give concrete, actionable edits rather than abstract criticism.

### Smarter stopping conditions

Stopping only when the feedback contains a phrase like "no need to improve" or when a max iteration count is hit is too brittle. Better choices are:

- Quality score threshold: stop once the output reaches, for example, 8/10.
- Version-delta threshold: stop when changes between rounds become very small.
- Multi-dimensional scoring: stop only when correctness, completeness, readability, and efficiency all pass.

The main upgrade is moving from hard-coded phrase checks to explicit quality control.

### A multi-dimensional Reflection mechanism for an academic writing assistant

Use at least four reflection dimensions:

- Logical flow: whether the argument is coherent across paragraphs and sections
- Method novelty: whether the contribution is more than a trivial recombination
- Language quality: terminology consistency and academic tone
- Citation quality: whether claims are supported and referenced correctly

Different reviewers can score each dimension, and the next refinement round can focus on the lowest-scoring dimension first.

## 6. Prompt engineering across the three paradigms

### Why do ReAct and Plan-and-Solve prompts look different?

- A `ReAct` prompt emphasizes the interaction protocol: how to cycle through `Thought -> Action -> Observation`.
- A `Plan-and-Solve` prompt emphasizes generating a full plan first and then following it instead of improvising at every step.

The structural differences are not cosmetic. They directly support the logic of each paradigm.

### Why does role setting affect Reflection behavior?

If you change the reviewer role from "an extremely strict code review expert" to "an open-source maintainer who prioritizes readability," the model's feedback emphasis usually shifts:

- from bugs, edge cases, and defensive coding
- toward naming, clarity, maintainability, and interface readability

This means role prompting implicitly changes the weighting of the evaluation function.

### Why can few-shot help?

Few-shot examples are especially useful when output format needs to stay highly stable.

For example, adding a ReAct demonstration such as:

```text
Question: What is the weather in Beijing?
Thought: I should call the weather tool.
Action: weather
Action Input: {"city": "Beijing"}
```

can significantly reduce formatting drift and improve parsing success.

## 7. E-commerce customer-service agent: architecture and risk control

### Which paradigm should be chosen?

I would use a combination of `Plan-and-Solve + ReAct + Reflection`:

- `Plan-and-Solve` defines the high-level workflow: understand the refund request, query order data, inspect logistics, apply policy, and draft a reply.
- `ReAct` handles execution-time tool calls against order, logistics, and email systems.
- `Reflection` provides second-pass review for low-confidence or disputed cases.

### At least three tools

Useful tools include:

- `order_lookup`: retrieve order status, product details, and paid amount
- `logistics_query`: retrieve tracking and delivery status
- `policy_checker`: apply refund-policy logic
- `send_email`: deliver the final message

### How should prompts balance company interest and user friendliness?

The prompt should explicitly require that the agent:

- follows company refund policy without making unauthorized promises
- explains evidence gaps instead of inventing facts
- maintains a professional, calm, and friendly tone
- escalates to a human when confidence is below a threshold

### What risks arise after launch, and how can technology reduce them?

Main risks:

- Wrong refund decisions causing financial loss
- Tool failures leading to wrong order lookup or wrong email delivery
- Poor wording escalating already sensitive disputes
- Hallucinations or loops in long-tail edge cases

Mitigations:

- Put irreversible actions behind rules and approval gates
- Validate tool parameters and add circuit breakers
- Use Reflection and human escalation for high-risk cases
- Keep detailed logs and audit trails for replay and debugging
