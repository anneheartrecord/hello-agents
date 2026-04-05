# Chapter 1 Reference Answers

> These reference answers are for reference only. The real value of the exercises lies in training your ability to reason about agent definitions, task environments, system design, and engineering trade-offs rather than memorizing a single standard response.

## 1. Are the four cases agents?

### Case A: A supercomputer conforming to von Neumann architecture

Conclusion: strictly speaking, by itself it is not an agent.

Reasoning:

- High compute alone does not make something an agent.
- An agent needs a clear perception-action-goal loop, while a bare supercomputer is only a general-purpose compute platform.
- It can host agent systems, such as weather forecasting agents, autonomous driving training systems, or multi-agent simulations, but the platform itself is not the agent.

From first principles, the key property is not "can it compute fast," but "can it perceive, decide autonomously, and act on an environment toward a goal."

### Case B: Tesla autonomous driving system

Conclusion: yes, it is an agent, and a classic real-world control agent.

Possible classifications:

- From internal architecture, it is at least a model-based reflex agent because it must maintain an internal state of lanes, vehicles, obstacles, and motion predictions.
- From the goal perspective, it is also a goal-based agent, since it aims to reach destinations safely while obeying traffic rules.
- From the learning perspective, modern autonomous driving systems also include learning components such as perception, prediction, and policy models.

Why:

- It perceives the environment using cameras, radar, and other sensors.
- It acts on the environment through steering, braking, and acceleration.
- It makes real-time decisions in a partially observable, dynamic, stochastic, and continuous environment.

### Case C: AlphaGo

Conclusion: yes, it is an agent.

Possible classifications:

- It is a goal-based agent because its explicit goal is to win the game.
- It is also a utility-based agent because it optimizes long-term winning probability instead of choosing moves only by immediate gain.
- It also has learning-agent characteristics because its strength comes from supervised learning, reinforcement learning, and search.

Why:

- It perceives the board state.
- It selects and executes moves.
- It must plan many steps ahead based on the current situation.

Compared with autonomous driving, AlphaGo operates in a discrete, turn-based, rule-complete world. It is still an agent, but in a much more closed environment.

### Case D: ChatGPT acting as intelligent customer service

Conclusion: if it is only a plain text completion interface, the answer is debatable. If it can call tools such as order lookup, logistics, refund systems, and knowledge bases, and autonomously drive the problem toward resolution, then it qualifies as an agent.

More precisely, the system described in the exercise is an LLM-driven service agent.

Possible classifications:

- LLM-driven agent
- Tool-using agent
- Hybrid agent that combines quick response with multi-step reasoning and workflow coordination

Why:

- It perceives user language input and backend responses.
- It takes actions such as checking orders, querying logistics, generating solutions, and escalating to a human operator.
- Its goal is not just to answer a sentence, but to help complete the complaint-resolution loop.

## 2. PEAS analysis for an intelligent fitness coach

### P: Performance Measure

Performance measures should be quantifiable and trackable.

| Metric | Quantitative standard | Measurement method |
|--------|------------------------|--------------------|
| Goal achievement rate | Percentage of user goals achieved, such as fat loss, muscle gain, or endurance improvement | Periodic physical assessment and milestone reviews |
| Training plan completion rate | Weekly or monthly completion percentage | System logs of finished sessions |
| Health indicator improvement | Changes in weight, body fat, resting heart rate, and sleep quality | Wearables and smart scale data |
| User retention | Percentage of users continuously using the system for 30 or 90 days | Product analytics |
| Injury rate | Number of discomfort or injury events caused by poor training execution | User feedback and abnormal motion detection |
| User satisfaction | Survey score, NPS, repurchase or continued subscription intent | Feedback system |

### E: Environment

The environment is not just the training location. It includes all external factors that affect decision-making.

- Physical setting: home, gym, outdoors, office, each with different space, equipment, noise, and safety conditions.
- User physical state: fitness level, injuries, fatigue, sleep condition, menstrual cycle, recovery status.
- User goals and constraints: fat loss, muscle gain, rehab, endurance, time budget, money budget.
- External environment: weather, air quality, schedule, class booking availability, gym opening hours.
- Historical context: recent training load, dietary logs, body weight trends, adherence history.

### A: Actuators

Actuators are the ways the agent affects the external world.

- Real-time voice coaching for breathing rhythm, posture correction, and rest timing.
- Text or card reminders for daily workout plans, diet suggestions, and recovery guidance.
- Demonstration videos for unfamiliar exercises.
- Dynamic plan adjustment based on heart rate, fatigue, and phase goals.
- Risk alerts when heart rate is too high, movement is abnormal, or overtraining is detected.
- Weekly and monthly reports summarizing training trends and next-step recommendations.

### S: Sensors

Sensors are the information sources the agent uses.

- Smart bands or watches: heart rate, step count, sleep, blood oxygen, calorie burn.
- Smart body scale: weight, body fat, muscle mass, and other long-term indicators.
- Phone camera: motion capture for posture analysis and correction.
- User input: perceived fatigue, diet logs, pain feedback, changing goals.
- External APIs: weather, air quality, calendar, venue information.
- Smart gym equipment: load, repetitions, sets, movement speed.

### Task environment characteristics

- Partially observable: the system can never fully know the user's real condition, such as hidden pain, actual diet execution, or psychological state.
- Stochastic: the same plan can lead to different outcomes under different sleep, mood, or weather conditions.
- Dynamic: the user's condition and environment keep changing, so the plan cannot remain fixed.
- Sequential: today's training intensity affects tomorrow's recovery, so current decisions shape future decisions.
- Continuous: many state variables such as heart rate, speed, load, and joint angle change continuously.
- Human-in-the-loop: the user's willingness, honesty, and feedback are themselves important parts of the system.

## 3. Workflow vs Agent in the e-commerce refund scenario

### Advantages and disadvantages of the two approaches

| Dimension | Workflow | Agent |
|-----------|----------|-------|
| Compliance | Strong, because rules are fixed | Weaker, because outputs may vary |
| Consistency | Strong, same input leads to same output | Weaker, especially in edge cases |
| Cost | Low, suitable for high-frequency standardized flows | Higher, due to model inference and tool use |
| Flexibility | Low, updating behavior requires system changes | High, can handle ambiguous and non-standard descriptions |
| Auditability | Strong, decision chain is clear | Weaker unless extra logging and explanation are added |
| Edge-case handling | Weak, can get stuck outside predefined rules | Stronger at handling fuzzy or novel cases |

### When Workflow is more suitable

- The refund policy is clear and stable.
- Approval standards must be exactly consistent.
- There is high concurrency with low average order value and many standard products.
- Financial or regulatory audit requirements are strict.

### When Agent has more advantages

- Users provide highly unstructured descriptions, such as long complaint text, images, or mixed evidence.
- Decisions need to combine order history, product condition, and risk profile.
- Business policies change quickly and cannot be exhaustively encoded in rules.
- The front-end interaction needs natural language explanation, comfort, and follow-up questions.

### If I were the business owner

I would choose neither pure Workflow nor pure Agent. I would choose Approach C: Workflow as the skeleton, Agent as the analysis layer.

The reason is straightforward:

- Final approval and money movement should remain under deterministic rules or human control.
- The value of the agent is in understanding messy descriptions, summarizing evidence, and generating recommendations, not making unconstrained irreversible decisions.

### Approach C: a layered hybrid architecture

1. The first layer uses Workflow for standard refunds.
2. The second layer sends ambiguous or uncovered cases to the Agent for classification, summarization, risk analysis, and recommendation.
3. The third layer uses a rules engine or a human reviewer to make the final decision based on the Agent's output and business rules.

Benefits:

- Standard cases are automated at low cost.
- Complex cases get stronger understanding ability.
- Irreversible side effects stay inside an auditable and controlled system.

## 4. Adding memory, fallback options, and reflection to the travel assistant

### Feature 1: remember user preferences

Design idea: add a memory layer around the `Thought-Action-Observation` loop.

- Short-term memory stores the current session's budget, travel dates, companion count, and rejected options.
- Long-term memory stores stable user preferences, such as liking historical attractions, disliking queues, a budget ceiling, or hotel preferences.
- Before each `Thought` step, retrieve relevant memory and inject it into context.

Illustrative flow:

```text
User Input
  -> Retrieve Memory
  -> Thought
  -> Action
  -> Observation
  -> Update Memory
```

### Feature 2: automatically recommend alternatives when tickets are sold out

The key idea is to treat failure as a new observation rather than the end of execution.

- Add a `check_ticket_availability` tool.
- If the observation says `sold_out`, do not stop immediately. Enter a new `Thought` step.
- The new goal is not to repeat the same recommendation, but to find similar options by type, area, or budget.

A simple candidate strategy can be:

- Same category first: replace a historical attraction with another historical attraction.
- Same area first: reduce route disruption.
- Similar price range first: avoid jumping from low budget to high budget.

### Feature 3: reflect after three consecutive rejections

This requires explicit failure tracking and a reflection node.

- Maintain a `rejection_count` in session state.
- Increment it every time the user rejects a recommendation.
- When it reaches 3, trigger a `reflection step` to summarize why previous recommendations failed.

Possible reflection questions:

- Was the mismatch about budget or attraction type?
- Was the itinerary too dense or the travel distance too large?
- Were the recommendations too generic or off from the user's actual interest?

Example pseudocode:

```python
state = {
    "preferences": {},
    "rejection_count": 0,
    "rejected_reasons": []
}

def loop(user_input, state):
    memory = retrieve_memory(state)
    thought = think(user_input, memory, state)
    action = choose_action(thought)
    observation = run_tool(action)

    if observation.get("ticket_status") == "sold_out":
        return recommend_alternatives(observation, state)

    if user_rejects(user_input):
        state["rejection_count"] += 1
        state["rejected_reasons"].append(extract_reason(user_input))

    if state["rejection_count"] >= 3:
        strategy = reflect_and_adjust(state)
        return new_recommendation(strategy, state)

    update_memory(state, user_input, observation)
    return respond(observation, state)
```

The core idea is not "loop more times," but "learn from failure inside the loop and change strategy."

## 5. Applying System 1 and System 2 to agent design

Here I use a medical diagnosis assistant as the scenario.

### Tasks suitable for System 1

System 1 is suitable for high-frequency, standardized, low-risk tasks that need quick judgment:

- Initial triage of common symptom patterns, such as fever, cough, and runny nose.
- Standardized questioning flow.
- Immediate drug allergy alerts.
- Quick matching of common drug dosage and common contraindications.

### Tasks suitable for System 2

System 2 is suitable for complex, high-risk tasks that need multi-step reasoning and evidence integration:

- Differential diagnosis for combined symptoms such as chest pain, breathing difficulty, and fever.
- Multi-drug interaction analysis.
- Rare disease screening and atypical case reasoning.
- Comparing treatment options and weighing risks against benefits.

### How the two systems work together

System 1 can serve as the fast front layer, and System 2 can serve as the escalation layer.

- If System 1 is confident and the risk is low, return a preliminary answer directly.
- If confidence is low, high-risk keywords are triggered, or the query involves medication, surgery, or urgent symptoms, escalate to System 2.
- In high-risk domains, System 2 output should still pass rule checks and human confirmation.

Simplified pseudocode:

```python
def process(query, context):
    fast_result = system1(query)

    if fast_result.confidence < 0.8:
        return system2(query, context)

    if contains_high_risk_signal(query):
        return system2(query, context)

    if involves_medication_or_surgery(query):
        return system2(query, context)

    return fast_result
```

## 6. Limitations and evaluation of LLM-based agents

### Why hallucinations happen

The root cause is that an LLM is a probabilistic generator, not an explicit fact database.

- It produces the token sequence that looks most likely under the context, not a verified truth retrieved from a structured source.
- Knowledge is compressed into distributed parameters instead of being stored in a directly queryable fact table.
- Once early generation drifts, later autoregressive generation can continue building on the error.

So hallucination is not a rare accident. It is a natural side effect of the model mechanism. It can be reduced, but not fully eliminated.

### What happens if there is no maximum loop count

At least three classes of problems can appear:

- Error propagation: if one step gets the wrong information, later steps reason from the wrong foundation.
- Resource exhaustion: token budget, time, API calls, and cost can keep increasing.
- Side-effect accumulation: repeated requests, repeated purchases, repeated messages, or even irreversible external actions.

In tool-using systems, it can also create dead loops, for example:

- A tool keeps returning malformed output and the agent keeps retrying.
- The goal is impossible, but the system has no explicit success or failure termination condition.
- The model keeps inventing subgoals without any convergence mechanism.

### How to evaluate an agent's intelligence

Accuracy alone is far from enough.

A fuller evaluation framework should at least include:

| Dimension | What it measures | How to measure |
|-----------|------------------|----------------|
| Accuracy | Whether the result is correct | Compare with gold answers or expert annotation |
| Robustness | Stability under abnormal input and edge cases | Adversarial tests and noisy input tests |
| Safety | Whether it leaks sensitive data or is vulnerable to prompt injection | Red-team evaluation |
| Tool-use success rate | Whether it calls tools correctly and handles outputs properly | Task completion rate and tool-call correctness |
| Latency and cost | Whether it is usable in real business settings | P50/P95 latency and per-task token cost |
| Consistency | Whether repeated runs on the same input stay stable | Multi-run comparison |
| Recoverability | Whether it can recover safely after failure | Fault injection tests |
| User satisfaction | Whether it actually solves user problems | Ratings, complaint rate, retention |

Accuracy only answers "is the answer correct." The real value of an agent also depends on whether it is safe, stable, economical, controllable, and recoverable.
