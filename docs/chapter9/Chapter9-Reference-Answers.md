# Chapter 9 Reference Answers

> These reference answers are for reference only. The core lesson of this chapter is to treat context as a system resource that can decay, bloat, and require governance rather than as raw material to keep stuffing into the model.

## 1. The boundary between context engineering and prompt engineering

### What is context rot, and why do long windows not justify stuffing everything in?

`Context rot` means that as context grows, truly useful information gets diluted by noise, stale facts, repeated snippets, and low-relevance material. The model sees more tokens, but decision quality may get worse.

Even with 100K or 200K context windows, careful management still matters because:

- long context increases cost and latency
- long-range information use is not always stable
- redundant information distorts attention allocation
- longer context makes it easier to mix in stale or conflicting content

The real question is not “how much can fit?” but “what should be included?”

### Code review over 50 files: all-at-once vs JIT retrieval

| Strategy | Pros | Cons | Best fit |
|----------|------|------|----------|
| Load everything at once | complete global view | expensive, noisy, inflexible | small repos or one-shot whole-system analysis |
| JIT retrieval | lower cost, higher relevance, progressive exploration | weaker initial overview, depends on retrieval quality | medium or large repos, interactive review |

For a 50-file repository, I would prefer `JIT + a small global summary`: start with structure and module summaries, then fetch concrete files on demand.

### Examples of over-hardcoding and over-vagueness

Over-hardcoding example:

- “Reply in exactly 7 paragraphs, each 80-100 words, and the second paragraph must praise the user before giving 3 suggestions.”

Problem: the model loses room to adapt expression to the task.

Too-vague example:

- “You are a great AI assistant. Please help the user as much as possible.”

Problem: there is no operational boundary, quality standard, or execution constraint.

The right balance is to make goal, boundary, and essential formatting explicit without scripting every move.

## 2. Failure analysis and improvement of the GSSC pipeline

### What happens if one stage fails?

Each GSSC stage has a distinct role, so failure propagates differently:

- `Gather` failure: important sources never enter the pipeline
- `Select` failure: irrelevant material floods the context
- `Structure` failure: useful content exists but is badly organized
- `Compress` failure: key constraints disappear during summarization

The end result may be wrong answers, missed constraints, repeated searches, tool misuse, or long-horizon drift.

### How to add a context quality assessment function

After building context, output a small quality report with at least:

- information density: useful tokens / total tokens
- relevance: similarity between built context and current task
- completeness: whether mandatory slots such as goal, constraints, current state, and prior decisions are present

Example:

```json
{
  "density": 0.71,
  "relevance": 0.84,
  "completeness": 0.76,
  "suggestions": [
    "remove repeated log fragments",
    "add the most recent failure reason",
    "compress long terminal output"
  ]
}
```

### When are truncation or sliding windows better than LLM summarization?

When fidelity to original text matters more than abstraction, simpler methods are safer, for example:

- stack traces
- config diffs
- latest log tails
- structured tables

A good hybrid compression policy is:

- preserve structured raw text with truncation or sliding windows
- summarize long explanatory prose with an LLM
- extract critical constraints into explicit fields rather than free-form summary

## 3. NoteTool, TerminalTool, and an intelligent refactoring assistant

### How should automatic note organization work?

Treat temporary notes as a staging layer. Once count or volume crosses a threshold:

1. cluster similar notes
2. extract recurring themes, unfinished actions, and risks
3. promote durable task-relevant items into task notes or project notes
4. archive or delete redundant noise

Promotion rules:

- current-task actionable items become task notes
- cross-task constraints or architecture knowledge become project notes

### Are the current safety mechanisms enough, and how should approval work?

Path validation, command allowlists, and permission checks are necessary, but not sufficient. Risk depends not only on the command string but also on what target it affects and whether the effect is reversible.

A safer approval flow:

1. the agent produces an operation plan and impact statement
2. the system risk-classifies the action
3. high-risk actions require human approval
4. approval issues a one-time execution token
5. execution writes an audit log and result snapshot

### Workflow for an intelligent code refactoring assistant

```text
Read repository structure
  -> build refactoring plan
  -> NoteTool records goals and risks
  -> TerminalTool performs search/edit/test steps
  -> NoteTool records results and issues
  -> if failed, revise the plan
  -> if passed, continue to next subtask
```

The key is not “let the agent edit freely,” but “close the loop across planning, execution, verification, and record keeping.”

## 4. Layered context management in long-horizon tasks

### How should the three layers coordinate?

A clean responsibility split is:

- `TerminalTool`: instant fact layer for current files, commands, and live state
- `MemoryTool`: session-state layer for recent conclusions and intermediate decisions
- `NoteTool`: persistent knowledge layer for cross-turn and cross-interruption continuity

Putting information in the wrong layer causes problems:

- long-term notes polluted by transient terminal output
- important constraints lost because they only lived in session memory

Use promotion rules so information is only elevated when necessary.

### How should resume-from-breakpoint work?

Notes should capture at least:

- task id
- completed subtasks
- remaining subtasks
- dependency status
- latest failure reason
- key files and modification points
- explicit next action

Recovery flow:

1. read the task snapshot
2. validate current state with `TerminalTool`
3. repair drift if the world has changed
4. continue from the saved next action

### Designing a task dependency management system

Model tasks as a DAG:

- nodes are subtasks
- edges are dependencies
- states include `pending / running / blocked / done / failed`

When integrated with `NoteTool`, each node can map to a structured note record for persistence, auditability, and recovery.

## 5. Value, risks, and boundaries of progressive disclosure

### A concrete scenario: complex debugging

For a production incident, progressive disclosure can look like:

1. inspect the error summary
2. fetch relevant log fragments
3. read the implicated code module
4. inspect config and deployment diffs
5. form a repair hypothesis

Each step narrows the search space instead of front-loading all logs and code into the prompt.

### How to avoid inefficient exploration

Add exploration guidance:

- prioritize sources with highest expected information gain
- if two steps in a row yield no useful signal, force a direction change
- set path budgets so the agent cannot over-invest in side branches
- require metacognitive justification: “why is this the next best thing to inspect?”

### When is progressive disclosure better, and when is all-at-once better?

Progressive disclosure is better for:

- debugging and incident triage
- large codebase analysis
- multi-step research and evidence gathering

All-at-once loading is better for:

- short rewriting tasks
- small QA tasks where all material is already compact
- short-context tasks requiring strong whole-context consistency

The right choice depends on task scale and how information is distributed, not on ideology.
