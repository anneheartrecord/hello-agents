# Chapter 10 Reference Answers

> These reference answers are for reference only. The real point of this chapter is not to memorize protocol names, but to understand which problem each protocol solves: agent-to-tool access, agent-to-agent collaboration, or large-scale agent networking.

## 1. Positioning differences among MCP, A2A, and ANP

### Why do the three protocols emphasize different things?

They are solving different problem boundaries:

- `MCP` emphasizes context sharing because it standardizes how one agent accesses tools, resources, and prompt templates.
- `A2A` emphasizes conversational collaboration because it standardizes how multiple agents coordinate, negotiate, and hand off work.
- `ANP` emphasizes network topology because it focuses on discovery, routing, scalability, and resilience in large agent networks.

A useful mental model is:

- MCP: capability access layer
- A2A: collaboration layer
- ANP: network layer

### Protocol choices for the intelligent customer-service system

For the three requirements in the exercise:

1. access to customer database and order systems: choose `MCP`
2. collaboration among specialist service agents: choose `A2A`
3. large-scale concurrent request handling: choose `ANP`

Why:

- tool and resource access is an MCP problem
- multi-role coordination is an A2A problem
- high-scale routing and node discovery is an ANP problem

### How can the three protocols be combined?

A practical layered design is:

```text
User request
  -> ANP ingress and routing layer
  -> A2A collaboration layer
  -> MCP tool/resource layer
  -> databases / order systems / knowledge bases
```

Responsibility split:

- `ANP` routes requests into the right service cluster
- `A2A` coordinates specialist agents within that cluster
- `MCP` exposes standardized access to tools, knowledge, and backend systems

## 2. MCP: tools, resources, prompts, and remote transport

### Designing a data-analysis MCP server

A strong server can expose three tools:

- `db_query`: controlled read-only SQL querying
- `chart_render`: convert structured results into chart output
- `report_generate`: turn data plus chart context into an analysis report

Workflow:

```text
User question
  -> db_query
  -> chart_render
  -> report_generate
  -> final analysis
```

Important engineering controls:

- permission levels per tool
- schema allowlists and row limits
- persistence of intermediate outputs for auditability

### What are Resources and Prompts for?

- `Resources` expose static or semi-static context such as documentation, policies, or manuals.
- `Prompts` expose reusable task templates.
- `Tools` perform actions.

Example: a financial-analysis assistant

- `Resources`: accounting rules, reporting definitions
- `Prompts`: quarterly analysis template
- `Tools`: query execution and chart generation

This gives the agent not only actions, but also reusable context and task scaffolding.

### Strengths and limits of JSON-RPC over stdio

Strengths:

- simple, language-agnostic protocol
- low-latency local process communication
- clearer security boundary because services are not exposed to the network by default

Limitations:

- best for local or near-local integration rather than distributed systems
- process lifecycle and health management require extra work
- less convenient for browser, SaaS, and cross-network scenarios

To support remote MCP servers, add:

- `HTTP/WebSocket` transports
- authentication such as tokens or mTLS
- session and timeout management
- retry, reconnect, and service-discovery logic

## 3. A2A: collaboration flow, conflict resolution, and framework bridges

### Adding a Reviewer to the research team case

A clean three-role split is:

- `Researcher`: gathers evidence and source material
- `Writer`: produces the draft
- `Reviewer`: critiques logic, structure, and clarity

Flow:

1. Researcher submits the research summary.
2. Writer creates a draft.
3. Reviewer returns revision requests.
4. Writer revises.
5. Reviewer accepts or requests another round.

This upgrades A2A from simple collaboration into a production-review loop.

### How should conflict resolution work?

Disagreement should not be settled by turn order. Extend the protocol with:

- `negotiation` messages for evidence-based disagreement handling
- `voting` messages for multi-agent or arbiter-backed resolution

Basic flow:

1. enter negotiation when conflict is detected
2. after limited rounds, move to voting if needed
3. if still unresolved, escalate to a human or rule engine

### How does A2A relate to AutoGen or CAMEL?

`A2A` is closer to a protocol standard. `AutoGen` and `CAMEL` are frameworks.

They are not direct substitutes:

- frameworks manage runtime, agent roles, and local orchestration
- protocols define how different systems exchange messages

Interoperability can be implemented with an `A2ABridge` that maps A2A messages to framework-native message objects and wraps framework outputs back into A2A-compliant envelopes.

## 4. ANP: topology, routing, and fault tolerance

### When should star, mesh, or hierarchical topology be used?

| Topology | Strengths | Weaknesses | Best fit |
|----------|-----------|------------|----------|
| Star | simple control and central scheduling | bottleneck and single point of failure risk | small systems with unified coordination |
| Mesh | flexible and decentralized | harder routing and governance | highly autonomous collaborative networks |
| Hierarchical | good balance of scale and manageability | more design complexity | medium and large enterprise systems |

As the network grows from 10 agents to 1000 agents, a common evolution path is from star or small mesh toward hierarchical topology with local mesh subnets.

### How should an intelligent routing algorithm work?

A routing score can combine:

```text
route_score = 0.35 * capability_match + 0.25 * current_load_inverse + 0.20 * latency_inverse + 0.20 * trust_score
```

Execution flow:

1. filter agents by capability
2. rank by route score
3. choose a primary and backup path
4. fail over automatically when the primary path breaks

### How should the system handle critical-agent failure?

Use a four-part fault-tolerance strategy:

- failure detection via heartbeats and health checks
- backup switching via active-standby or pooled peers
- state recovery via shared persistent storage
- traffic redistribution through the routing layer

For critical roles such as traffic control, also add a degraded fallback mode that reverts to rule-based control when the intelligent node is unavailable.

## 5. Security, privacy, and trust governance

### Security risks in MCP tool invocation and permission design

Main risks:

- accidental invocation of dangerous tools
- prompt injection causing high-risk calls
- malicious or malformed parameters leading to privilege escalation

Safer controls include:

- tool classification by risk level
- role-based authorization per agent
- parameter validation and policy checks
- explicit approval for dangerous operations

### End-to-end encryption for A2A and ANP

A workable design uses:

- agent certificates or public-private key pairs
- session key negotiation
- symmetric encryption for payloads and digital signatures for integrity
- routing nodes that can read only necessary metadata, not plaintext content

This supports confidentiality, integrity, and identity verification together.

### Designing a trust evaluation system

Maintain a dynamic `trust_score` per agent based on:

- historical task success rate
- consistency and truthfulness of messages
- peer or community feedback
- malicious-behavior indicators such as spam, abusive traffic, or repeated false outputs

Then adapt policy accordingly:

- high-trust agents get priority in critical collaboration
- low-trust agents lose weight, permissions, or are isolated entirely

In large agent networks, protocol is only the foundation. Governance is equally important.
