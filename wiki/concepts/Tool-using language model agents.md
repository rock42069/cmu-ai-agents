# Tool-using language model agents

Language models become agents when generation participates in a feedback loop:
the model selects an action, an external system returns an observation, and the
new state influences what the model does next. Tools can supply information the
model does not reliably store, perform exact computation, or change an external
environment. The difficult part is the **policy around the tool**, not merely
exposing an API.

## Two complementary locations for learning

[[sources/ReAct - Synergizing Reasoning and Acting in Language Models|ReAct]]
and [[sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer]]
place that policy at different points:

| Dimension | ReAct | Toolformer |
|---|---|---|
| Primary contribution | Inference-time reasoning/action loop | Self-supervised tool-use training data |
| Supervision | A few human-written trajectories; optional fine-tuning experiment | A few API demonstrations plus loss-based filtering of model-generated calls |
| Runtime structure | Repeated thought, action, observation steps | Generation pauses to execute a learned textual API call |
| Multi-step interaction | Central to the method | Not supported by the paper's independent, one-call setup |
| Main feedback signal | Environment observations and task outcome | Reduction in future-token prediction loss |
| Main strength | Planning, grounding, recovery, and diagnosability | Scalable acquisition of when/which/how-to-call behaviour |
| Characteristic failure | Reasoning loops or derailment after poor retrieval | No chaining/refinement; noisy calls and call-trigger calibration |

These are not competing definitions of an agent. A system could learn tool-call
syntax and selection using Toolformer-like data, then execute those calls inside
a ReAct-like feedback loop.

## A useful decomposition

When examining any tool-using agent, ask separately:

1. **Need:** How does it decide that internal knowledge is insufficient?
2. **Selection:** How does it choose a tool and account for capability, cost,
   latency, permissions, and risk?
3. **Arguments:** How does it construct a valid, specific call?
4. **Execution:** What system validates and performs the action?
5. **Observation:** How is the result represented, checked, and incorporated?
6. **Iteration:** Can it recover, reformulate, chain tools, or stop looping?
7. **Termination:** How does it know it has enough evidence or has completed the
   user's goal?

ReAct emphasizes observation, iteration, and plan revision. Toolformer provides
a concrete learning signal for need, selection, and arguments, but its
future-token-loss criterion does not directly optimize truth, task completion,
cost, or safety.

## Durable takeaway

“Give the model tools” hides most of the engineering and research problem. A
capable agent needs both a learned or designed tool-use policy and a controlled
feedback loop that validates actions, grounds subsequent reasoning, handles
failure, and terminates safely.

## Sources

- [[sources/ReAct - Synergizing Reasoning and Acting in Language Models|ReAct: Synergizing Reasoning and Acting in Language Models]]
- [[sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer: Language Models Can Teach Themselves to Use Tools]]
