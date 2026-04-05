# Chapter 8 Reference Answers

> These reference answers are for reference only. The real lesson of this chapter is that memory systems and RAG are not isolated modules. They are the infrastructure behind long-term capability, external knowledge use, and personalization.

## 1. Coordinating four memory types and consolidating working memory

### Why does episodic memory emphasize recency while semantic memory emphasizes graph retrieval?

Episodic memory stores concrete events: what happened, when it happened, and in what task context. That makes temporal recency naturally important, because recent events are often more relevant to the current situation.

Semantic memory stores stable knowledge structures, such as facts, concepts, and relationships. Its value depends less on when the memory was created and more on how well it connects to other knowledge. That is why graph retrieval has higher weight.

A simple view:

- episodic memory answers “what happened recently?”
- semantic memory answers “how is this concept connected to other concepts?”

### How would a personal health assistant use the four memory types?

| Memory type | What to store | Example use |
|-------------|---------------|-------------|
| Working memory | current session goals and temporary state | today's breakfast, current step count, current symptoms |
| Episodic memory | concrete health events | elevated heart rate after last Wednesday's run |
| Semantic memory | stable knowledge and user profile | lactose intolerance, doctor-advised low-salt diet |
| Perceptual memory | raw sensor observations | wearable heart-rate curves, sleep data, scale readings |

A practical stack is: perceptual memory captures raw data, working memory supports the live interaction, episodic memory records meaningful events, and semantic memory stores long-lived health knowledge and user profile facts.

### When should working memory be consolidated into long-term memory?

Not every short-lived fact deserves promotion. Good triggers include:

- high repetition across sessions
- high importance, especially risks, allergies, chronic conditions, or prohibitions
- cross-task usefulness
- explicit user confirmation such as “remember this preference”

One practical scoring rule is:

```text
consolidate_score = 0.35 * importance + 0.25 * repeat_count + 0.20 * cross_task_value + 0.20 * user_confirmation
```

Promote the item when the score crosses a threshold.

## 2. RAG chunking, retrieval strategies, and embedding choices

### How should chunking work when documents have no heading structure?

For novels, legal text, or meeting transcripts, heading-based splitting is not enough. A better approach is sentence-level segmentation plus semantic-boundary detection.

Reference algorithm:

1. split into sentences or natural paragraphs
2. compute embeddings per segment
3. compare adjacent similarity
4. create a new chunk when similarity drops sharply or a topic shift is detected
5. merge with token-budget and overlap constraints

Pseudocode:

```text
for segment in segments:
    if similarity(prev, current) < threshold or token_limit_reached:
        start_new_chunk()
    else:
        append_to_chunk()
```

The point is to align chunk boundaries with topic boundaries instead of arbitrary length.

### Comparing baseline retrieval, MQE, and HyDE

In technical documentation QA:

- baseline retrieval is fast and cheap but sensitive to wording mismatch
- MQE expands one query into several retrieval angles, which helps when terminology varies
- HyDE generates a hypothetical answer first, then retrieves against that richer semantic representation, which helps when the user query is abstract

Recommendation:

- use baseline retrieval for direct FAQ-like questions
- use MQE when terminology is inconsistent
- use HyDE when users describe goals without knowing the right technical terms

### Comparing three embedding approaches

| Approach | Accuracy | Speed | Cost | Offline deployment | Best fit |
|----------|----------|-------|------|--------------------|----------|
| Bailian API | Usually high | Network-dependent | Pay-per-call | No | fastest cloud integration |
| Local Transformer | High and controllable | Hardware-dependent | Higher setup cost | Yes | private deployment, long-term control |
| TF-IDF | Weak semantic understanding | Fast | Very low | Yes | lightweight keyword retrieval or fallback |

Conclusion:

- use API embeddings for fastest initial delivery
- use local transformers for privacy and long-term control
- use TF-IDF as a light recall layer or fallback, not as the only semantic solution

## 3. Intelligent forgetting, memory archiving, and secure deletion of sensitive data

### Designing an intelligent forgetting strategy

Combine importance, access frequency, recency, and cross-reference count:

```text
retain_score = 0.40 * importance + 0.25 * access_frequency + 0.20 * recency + 0.15 * cross_reference_count
forget_score = 1 - retain_score
```

When storage pressure rises, forget high-`forget_score` items first.

This is better than pure time-based deletion because old but important memories should not disappear just because they are old.

### How should a memory archive work?

Move long-unused but potentially valuable memories into a cold-storage layer such as object storage or an archive database.

Integration pattern:

- working memory usually expires instead of being archived
- episodic memory can move to cold storage after long inactivity
- semantic memory should retain high-value knowledge and archive low-frequency fringe facts
- perceptual memory can keep summaries hot while raw bulky data moves cold

Retrieval should check hot data first, then the archive index, and restore on demand.

### Why isn’t deleting one database row enough for sensitive forgetting?

Sensitive information may still survive in:

- vector embeddings
- graph nodes and edges
- caches
- backups and logs
- derived summaries or downstream memories

A safer deletion flow is:

1. locate the primary record plus all embedding, graph, and cache references
2. cascade-delete related indexes and edges
3. regenerate affected summaries or derived knowledge
4. enforce deletion or expiry policies in backup systems
5. write an audit record proving the deletion completed

## 4. Coordinating RAG and Memory in the learning assistant

### When should RAG be prioritized, and when should Memory be prioritized?

Prefer RAG when:

- the question depends on external documents or textbook content
- the task is fundamentally knowledge lookup
- source-grounded answers matter most

Prefer Memory when:

- the answer depends on the learner's own history
- the task needs past mistakes, preferences, progress, or weak points
- the question is fundamentally personalized tutoring

### Designing an intelligent router

Use a lightweight router to classify each request into:

- `knowledge_query`
- `personal_progress_query`
- `hybrid_query`

Useful routing features:

- personal-history phrases such as “last time” or “my progress”
- textbook or formula references
- whether both user history and external knowledge are required

Hybrid queries can use dual retrieval plus reranking.

### Designing a smarter learning report generator

A better report should include:

- learning trajectory over time
- knowledge blind spots and repeated failure patterns
- next-step recommendations tied to relevant materials

Required ingredients:

- episodic memory for study events
- semantic memory for concept relationships
- RAG for pulling supporting material from the corpus
- hybrid retrieval to map weak topics back to learning content

### Data isolation in a multi-user web service

In Qdrant, isolation can be done with `payload.user_id`, per-tenant collections, or shard routing. In Neo4j, use `tenant_id` labels, subgraph isolation, or separate databases.

Performance guidance:

- use hot caches for active users
- partition by tenant where possible
- apply user-level filtering before similarity search instead of after retrieval

The core principle is to isolate at the indexing and query layer, not only as a post-filter.

## 5. Knowledge-graph quality and the value of graph retrieval

### How reliable is automatic entity-relation extraction?

It is useful, but not consistently reliable. Common failure modes include:

- wrong entity boundaries
- inverted relation direction
- ambiguity from same-name concepts across domains
- hallucinated relations not supported by source text

### Designing a knowledge-graph quality assessment system

Evaluate at least four dimensions:

- extraction precision through human spot checks
- structural consistency of the graph
- source traceability for each edge
- downstream task gain, meaning whether graph retrieval actually improves answers

### A query scenario where pure vector retrieval is not enough

Example: “Find the prerequisite knowledge chain for a course and identify which missing nodes in the student's current mastery will block the next stage.”

This requires:

- multi-hop reasoning
- path finding
- comparing the student's mastered concepts against a prerequisite graph

Pure vector search may retrieve relevant text, but it does not naturally produce dependency paths.

### When does graph retrieval clearly outperform pure vector retrieval?

Graph retrieval is especially strong for:

- multi-hop relation questions
- dependency-path questions
- structured entity interaction questions, such as disease-drug-symptom-contraindication chains

Vector retrieval is better at semantic similarity. Graph retrieval is better at structural reasoning. In many practical systems, the hybrid strategy is stronger than either alone.
