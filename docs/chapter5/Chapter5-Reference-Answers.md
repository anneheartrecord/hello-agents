# Chapter 5 Reference Answers

> These reference answers are for reference only. The real point of this chapter is to understand the trade-offs between low-code platforms and full-code development across speed, control, integration depth, and compliance.

## 1. Coze, Dify, n8n, and the three development modes

### How the three platforms differ in positioning

| Platform | Core positioning | Main strengths | Main weaknesses |
|----------|------------------|----------------|-----------------|
| Coze | Fast AI app and bot creation platform | Very low barrier to entry, friendly plugin ecosystem | More limited deep customization and private deployment |
| Dify | Open-source platform for LLM and agent applications | Good balance of workflow, knowledge base, plugins, and self-hosting | Still requires engineering ability in more complex settings |
| n8n | General workflow automation platform with AI as one capability | Excellent system-to-system orchestration | Less native support for sophisticated agent reasoning than dedicated agent platforms |

From first principles:

- Coze solves "build it quickly."
- Dify solves "build it quickly but keep control."
- n8n solves "connect many systems and automate the flow."

### When each development mode fits best

1. Low-code first:
   best for fast validation, quick launch, and teams with limited engineering resources.
   Example: marketing mini-programs, briefing bots, event assistants.

2. Full-code first:
   best for high customization, high performance, high compliance, and deep infrastructure integration.
   Example: financial risk review systems, internal developer platforms, complex multi-agent systems.

3. Hybrid development:
   best when standard workflows are handled by the platform, while specialized capabilities are handled by custom services.
   Example: Dify handles chat and orchestration, while an internal Python service calls proprietary company APIs.

In practice, many enterprise systems eventually move toward hybrid development because it balances speed and control better than either extreme.

## 2. Extending the Coze daily AI briefing case

### How to push the briefing automatically every morning at 8:00

The key change is to move from passive query-response to scheduled triggering.

A workable design:

1. Use the platform's scheduler if available, or trigger it daily via an external scheduler.
2. Run a fixed chain: fetch news -> generate briefing -> send message.
3. Deliver the result into Feishu, WeCom, or a WeChat Official Account integration.

If the platform's built-in scheduling is weak, add an external trigger layer such as `n8n`, `cron`, or a cloud function.

### How to improve the briefing prompt

The right target is not just "make it longer." It should feel more like an editorial workflow. Improvements can include:

- Fixed sections: headline news, key interpretation, trend watch, notable companies/products.
- Output constraints: separate facts from analysis.
- Audience targeting: executives, engineers, or general users need different summary granularity.

Example useful additions:

- Require each item to include one sentence on "why this matters."
- Allow trend judgment only when grounded in the available facts.

### What is MCP, why does it matter, and what would Coze gain if it supports it?

`MCP` is essentially a standardized protocol for exposing tools, resources, and prompt templates to models and agents.

Why it matters:

- It reduces repeated integration work.
- It gives different platforms and tool providers a shared interface contract.
- It lets agent systems move beyond closed plugin stores toward a more open tool ecosystem.

If Coze supports MCP in the future, it could:

- integrate enterprise internal systems more easily
- reuse MCP tools created by other teams or platforms
- move from a plugin-market model toward a broader standardized tool network

## 3. Dify multi-agent architecture, long DDL context, and deployment choices

### Why use a classifier for routing?

The classifier narrows the problem space before handing the request to a specialized sub-agent.

Benefits:

- Clearer responsibility per sub-agent
- Shorter prompts and more focused reasoning
- Better maintainability and debugging

If one universal agent handles everything, likely problems include:

- very long context
- blurred roles
- lower accuracy because one prompt must carry too many tasks

### How to avoid pasting all DDL for 50 tables into the prompt

Use `metadata retrieval + staged SQL generation`:

1. Build an index over table and field metadata.
2. Retrieve only the most relevant 2-5 tables for the current user query.
3. Provide only those DDL statements and examples to the model.
4. Perform secondary retrieval for foreign-key or field explanations if needed.

The underlying idea is to make database schema information retrievable rather than fully inlined.

### Local deployment vs cloud deployment

| Dimension | Local deployment | Cloud deployment |
|-----------|------------------|------------------|
| Data security | Stronger | Depends on provider |
| Initial cost | Higher | Lower |
| Maintenance difficulty | Higher | Lower |
| Performance controllability | Stronger | Constrained by the managed environment |
| Launch speed | Slower | Faster |

Practical suggestion:

- Use local deployment for sensitive enterprise and compliance-heavy customers.
- Use cloud deployment for validation-stage products and small teams.

## 4. n8n email assistant: persistence, attachments, and complex automation

### How to replace in-memory storage with persistent storage

The core idea is to move state out of memory into external services:

- Use `Pinecone`, `Qdrant`, or similar for vector search.
- Use `Redis` or a database for memory persistence.

Typical steps:

1. Provision the target storage service.
2. Configure credentials and connection settings in n8n.
3. Replace memory-backed nodes with persistent ones.
4. Verify that state still exists after a restart.

### How to process PDF or image attachments

Extend the flow like this:

1. Detect attachment type when the email arrives.
2. Send PDF files through a document parsing path.
3. Send images through OCR or a vision model path.
4. Merge extracted attachment content with the email body.
5. Generate a reply based on both sources.

The real goal is not merely to read attachments, but to convert them into a unified representation that later nodes can reason over.

### A more complex e-commerce automation scenario

After an order is created, automatically:

1. send the confirmation email
2. update the inventory database
3. notify the logistics system to create a shipment
4. record the customer and event into CRM

This is exactly the kind of cross-system orchestration n8n is good at.

## 5. Prompt engineering in low-code platforms

### How the prompts differ across the three platforms

- Coze prompts usually lean toward end-user interaction and polished app behavior.
- Dify prompts more often emphasize routing, task boundaries, and knowledge injection.
- n8n prompts often focus on input-output constraints for a specific automation node.

This is not accidental. Platform form factor shapes prompt design. The more front-end oriented a platform is, the more its prompts behave like product interaction design. The more workflow-oriented it is, the more its prompts behave like backend task contracts.

### Should output length be hard constrained?

A rule like "must exceed 500 words" is only justified when the business requirement truly needs a minimum information volume, such as a publishable article draft or a weekly report.

It is harmful when:

- the question only needs a short answer
- the model inflates length with filler to satisfy the rule

Usually it is better to constrain structure and coverage instead of raw length.

## 6. Tool and plugin extension, MCP comparison, and custom plugin development

### What if the platform lacks the exact tool you need?

Common options:

1. Wrap the internal company API as an HTTP service.
2. Build a custom plugin or node if the platform allows it.
3. Push complex capabilities into an external code service and let the platform call it.

### How are MCP, RESTful APIs, and Tool Calling different?

One simple way to view them:

- `RESTful API` is a general integration mechanism for software systems.
- `Tool Calling` is a model-side mechanism for invoking tools.
- `MCP` aims to be a standardized interface layer for the agent era, unifying tools, resources, and prompt templates.

That is why MCP is often described as a new standard: it tries to define a common contract across models, platforms, and tool providers.

### High-level Dify custom plugin workflow

Typical steps:

1. Define the plugin capability boundary and I/O schema.
2. Implement the backend logic.
3. Design authentication, error handling, and parameter validation.
4. Register and test the plugin in Dify.
5. Write a clear tool description so routing and agent usage remain reliable.

## 7. Platform selection for three applications

### Application A: consumer-facing AI writing assistant mini-program

Recommendation: `Coze` or a similar rapid-delivery platform.

Why:

- The main goal is to validate demand quickly.
- The team is very small.
- Private deployment and deep custom orchestration are not the main constraints yet.

### Application B: enterprise intelligent contract review system

Recommendation: `Dify + private deployment + external code services`, possibly mixed with custom code.

Why:

- Strong data-security and integration requirements
- Need for knowledge base, workflows, auditability, and self-hosting
- Contract review is high risk and must remain controllable

### Application C: internal R&D efficiency tool

Recommendation: `n8n + custom code services`, or a more code-heavy internal architecture.

Why:

- This is fundamentally a multi-system process orchestration problem.
- The team has enough engineering strength for custom extensions.
- Long-term integration with Git, CI, testing systems, and bug trackers matters more than no-code convenience.

Summary:

- For fast validation, choose Coze.
- For enterprise compliance-heavy AI apps, choose self-hosted Dify.
- For internal systems orchestration, choose n8n or a code-led hybrid architecture.
