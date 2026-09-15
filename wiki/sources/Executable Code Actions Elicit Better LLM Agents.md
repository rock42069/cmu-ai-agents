# Executable Code Actions Elicit Better LLM Agents

Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, and
Heng Ji (ICML 2024).

## Brief

CodeAct proposes executable Python as the action language between an LLM agent
and its environment. Compared with isolated text commands or JSON calls, code
provides variables, control flow, data flow, tool composition, existing software
libraries, and useful execution errors. The paper evaluates the representation
across 17 models, introduces an 82-task multi-tool benchmark, builds a 7,139-
trajectory instruction dataset, and fine-tunes two 7B CodeActAgent variants.

## Summary

### Code as an action space

In CodeAct, each environment-directed action is a Python program executed by an
interpreter. Its output, including exceptions and tracebacks, becomes the next
observation in a multi-turn agent loop. The model can then revise the action,
debug it, or issue new code. Natural language remains the interface for the user;
code unifies agent-environment interaction.

The claimed advantages are structural. Variables preserve intermediate values;
loops and conditionals express repeated or contingent actions; one tool's output
can become another tool's input; and Python packages expand the action space
without wrapping every capability in a bespoke schema. Models have also seen
large quantities of code during pretraining, while custom text or JSON action
formats may require specialized tuning.

### Comparing code, JSON, and text

The paper first repurposes API-Bank level-1 examples to compare atomic calls.
Code is comparable to or better than the other formats for many models, although
closed models frequently perform well with JSON—likely reflecting tool-call
fine-tuning. Atomic calls do not exercise code's control- and data-flow benefits.

For complex composition, the authors create M3 ToolEval: 82 human-curated tasks
covering browsing, finance, travel, science, and information processing, each
with manually crafted tools. Models receive no demonstrations, can interact for
up to ten turns, and are scored by exact answer match. Across 17 tested models,
CodeAct is the best success-rate format for 12 and the fewest-turn format for 12.
For GPT-4-1106-preview it reaches 74.4% success, versus 53.7% for text and 52.4%
for JSON, with 5.5 average turns versus 7.7 and 7.6. Results are uneven for weak
open models: the best tested open model reaches only 13.4%.

### Training CodeActAgent

CodeActInstruct contains 7,139 multi-turn trajectories across information
seeking, math and code with software packages, SQL/Pandas table reasoning, and
ALFWorld robot planning. Strong proprietary models generate the trajectories.
The pipeline preferentially retains examples where an initial error is later
corrected, explicitly training recovery and self-debugging rather than only
first-attempt success.

The authors mix CodeActInstruct with 69,230 general reasoning and conversation
examples, then fully fine-tune Llama-2-7B and Mistral-7B. On out-of-domain MINT,
the Mistral CodeActAgent scores 32.4 versus 9.7 for Mistral-Instruct; it also
generalizes to text-action benchmarks and mostly preserves or improves the
reported generic capabilities. The Llama-2 variant improves many agent tasks
but scores 0 on M3 ToolEval, showing that the action representation and dataset
cannot compensate for every base-model limitation.

## Key ideas to keep in mind

- An agent action format is a programming-language design choice, not merely a
  serialization detail.
- Code compresses multiple calls and their control/data dependencies into one
  executable action; JSON usually describes one call at a time.
- Execution errors are observations. Multi-turn self-debugging turns failure
  into a feedback channel instead of making every first attempt terminal.
- A general-purpose interpreter exposes an enormous existing tool ecosystem,
  but also an enormous authority and security surface.
- Training trajectories should include recovery from mistakes, not only polished
  successful solutions.
- Representation advantages grow with task and model capability; weak models
  may not plan, code, or follow complex instructions well enough to benefit.

## Connections

- [[wiki/concepts/Tool-using language model agents|Tool-using language model agents]]
- [[wiki/concepts/Structured generation for agents|Structured generation for agents]]
- [[wiki/sources/ReAct - Synergizing Reasoning and Acting in Language Models|ReAct]]
  supplies the thought-action-observation loop; CodeAct makes the action itself
  an executable program with compositional control and data flow.
- [[wiki/sources/XGrammar - Flexible and Efficient Structured Generation Engine for Large Language Models|XGrammar]]
  can constrain generated action syntax, but sandboxing and semantic validation
  remain necessary before execution.

## Questions and caveats

- M3 ToolEval contains only 82 tasks and uses exact-match outcomes; it is useful
  evidence for composition, not a comprehensive real-world benchmark.
- Comparisons mix models with different prior exposure and vendor-specific
  tuning for code, JSON, or function calling.
- Training data is synthetically generated by proprietary teacher models and
  selectively filtered, which can transfer their biases and failure patterns.
- Executing model-generated Python is materially more dangerous than validating
  a narrow function call. A sandbox, resource limits, capability boundaries,
  package/network controls, and human approval for consequential actions are
  part of the actual system design.
- Tracebacks enable correction but do not guarantee it; the agent can hallucinate
  program state or repeatedly make unsafe or ineffective changes.

## Source

- [[raw/2402.01030-codeact.pdf|Local PDF]] ([file](../../raw/2402.01030-codeact.pdf); SHA-256 `749a45e36cf89fc7e1b701a2a72f6609ce8d3697577ceb0fff5ce9339ce8539e`)
- [[raw/2402.01030-codeact.LICENSE|Attribution and CC BY 4.0 license record]]
- [Canonical arXiv record](https://arxiv.org/abs/2402.01030)
- arXiv:2402.01030v4, revised June 7, 2024; ICML 2024
