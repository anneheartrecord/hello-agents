# Chapter 12 Reference Answers

> These reference answers are for reference only. The real point of this chapter is not memorizing metrics, but building a clear judgment that agent evaluation is always shaped jointly by task goals, execution behavior, cost constraints, and business risk.

## 1. Benchmarks, customer-service evaluation design, and the three major challenges

### BFCL vs GAIA

`BFCL` focuses on tool-calling capability, especially:

- choosing the correct function
- supplying correct parameters
- handling multi-tool invocation properly

`GAIA` focuses more on broad assistant capability, including:

- multi-step task completion
- external information retrieval
- complex problem solving
- final-answer quality

BFCL uses AST matching because tool use is fundamentally a structured program-like object. GAIA uses quasi exact match because final answers often have multiple surface forms.

| Method | Strengths | Weaknesses |
|--------|-----------|------------|
| AST matching | structure-sensitive and good for call verification | may misjudge semantically equivalent but structurally different cases |
| Quasi exact match | practical for answer variation | limited for deep semantic equivalence |

### Designing metrics for an intelligent customer-service system

Break evaluation into four parts:

1. intent understanding: classification accuracy, recall, confusion matrix
2. API correctness: tool-call success rate, parameter correctness, execution-order correctness
3. friendliness and professionalism: LLM Judge plus human spot checks and terminology checks
4. robustness under exceptions: stress tests with noisy input, timeout, missing data, and adversarial prompts

The rule is simple: automate objective dimensions first, then use judge models and human review for subjective dimensions.

### Solutions to the three main challenges

1. output uncertainty:
   use repeated sampling, distribution-aware statistics, and stability evaluation rather than a single run.

2. diverse evaluation standards:
   build a layered metric system separating correctness, efficiency, safety, and user experience.

3. high evaluation cost:
   adopt tiered evaluation, with lightweight suites for daily development and heavy suites before release.

None of these remove cost entirely, but they make evaluation more sustainable.

## 2. BFCL: AST matching, boundary samples, and evaluator extensions

### Why is AST matching better than string matching?

String matching is too brittle because of:

- parameter reordering
- whitespace and quoting differences
- surface-form differences with similar structure

AST matching focuses on structural intent, which is closer to what we actually want to validate.

But AST matching can still fail:

- false positives when structure matches but semantic defaults differ
- false negatives when structure differs but the effective call is equivalent after expansion

Improvements include schema validation, parameter normalization, and pre-execution simulation.

### Designing boundary samples for the four categories

Useful additions:

- `simple`: missing optional args, ambiguous units, alias-based function names
- `multiple`: weakly dependent tools where order is easy to get wrong
- `parallel`: a mostly parallel task with one hidden sequential dependency
- `irrelevance`: prompts that mention tool-like keywords but do not actually need a tool call

These reveal a common failure mode: calling tools just because the prompt sounds tool-related.

### How should the BFCL evaluator be extended?

Add:

- order evaluation for dependency-sensitive calls
- efficiency evaluation for near-minimal call count
- detailed error analysis grouped by wrong tool, wrong params, wrong order, or over-calling

This turns evaluation from a single score into a diagnostic system.

## 3. GAIA: difficulty levels, answer matching, and custom suites

### Differences among Level 1, 2, and 3, and what Level 4 should be

A practical interpretation:

- `Level 1`: short tasks with limited capability requirements
- `Level 2`: more retrieval, integration, and moderate reasoning
- `Level 3`: longer multi-step tasks with more tool use and cross-source synthesis

A strong `Level 4` would add:

- longer-horizon task execution
- dynamic environment change
- deeper tool-dependency chains
- joint constraints across safety, cost, and planning

### Limits of quasi exact match and a smarter alternative

Quasi exact match usually handles normalization such as number formatting, case normalization, and textual cleanup, so it can often accept `42`, `42.0`, and `forty-two`.

But it struggles with:

- deeper paraphrases
- semantically equivalent but structurally distant answers
- multi-field composite outputs

A stronger stack is:

1. rule-based normalization
2. structured field comparison
3. LLM Judge for semantic equivalence with explanation

### How should a custom GAIA suite be designed?

In finance, for example, create 10 questions spanning:

- Level 1: direct extraction from one report
- Level 2: cross-report comparison and trend analysis
- Level 3: integrated reasoning over reports, news, and industry signals

Each item should define:

- canonical answer
- acceptable variants
- required tools
- scoring rules

That is what makes the evaluation repeatable and interpretable.

## 4. LLM Judge and multi-judge evaluation

### Strengths and limits of LLM Judge

Strengths:

- can score dimensions that are hard to formalize
- flexible for creativity, coherence, and completeness
- well suited to open-ended generation tasks

Limits:

- style bias
- length bias
- inter-judge inconsistency
- drift when rubrics are underspecified

### Scoring rubric ideas for three scenarios

1. code generation:
   correctness 40%, readability 20%, efficiency 20%, safety 20%.

2. creative writing:
   originality 30%, narrative coherence 25%, language quality 25%, emotional impact 20%.

3. technical documentation:
   accuracy 35%, clarity of structure 25%, completeness 25%, terminology consistency 15%.

Every rubric should include anchor examples so judges do not drift.

### How should a multi-judge system work?

Use 3-5 different judge models:

1. each judge scores independently against the same rubric
2. normalize scores across judges
3. aggregate with median or trimmed mean
4. trigger review or arbitration if disagreement is too large

Flag abnormal judges when:

- their scores deviate too far from the others
- justifications are shallow or formulaic
- behavior sharply deviates from historical scoring patterns

## 5. Tiered evaluation, continuous evaluation, and reporting

### A tiered evaluation strategy

- quick evaluation: small sample, critical path, low cost, for daily iteration
- standard evaluation: main flows plus important edge cases, for release gating
- comprehensive evaluation: stress tests, long-horizon tasks, adversarial inputs, and human review, for major launches

A common flow is: prepare dataset -> run evaluation -> aggregate metrics -> generate report -> apply gate or alert.

### Designing a continuous evaluation system

Core components:

- scheduler
- evaluation runner
- metric store
- trend dashboard
- alerting system

Example alert rules:

- core metric drops beyond threshold in two consecutive runs
- sudden rise in tool-call failure rate
- critical task flips from pass to fail

### Designing reports for different audiences

Developer reports should highlight:

- failing samples
- error-type distribution
- tool-call traces
- regressions between versions

Product-manager reports should highlight:

- task completion rate
- critical user-flow performance
- cost changes
- release risk summary

User-facing reports should be simplified into:

- whether core capabilities improved
- which scenarios became more reliable
- simple charts or grades instead of technical jargon

Evaluation is not only for the model team. It must support product decisions and external communication too.
