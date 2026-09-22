# Wiki index

## Sources

- [[wiki/sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer: Language Models Can Teach Themselves to Use Tools]] — Learns when and how to call textual APIs by generating candidate calls, executing them, and retaining calls that improve future-token prediction.
- [[wiki/sources/ReAct - Synergizing Reasoning and Acting in Language Models|ReAct: Synergizing Reasoning and Acting in Language Models]] — Interleaves explicit reasoning, external actions, and observations so each can guide the others during task execution.
- [[wiki/sources/What Are Tools Anyway - A Survey from the Language Model Perspective|What Are Tools Anyway? A Survey from the Language Model Perspective]] — Defines LM tools as external program interfaces and surveys tool functions, learning methods, creation, evaluation, and cost trade-offs.
- [[wiki/sources/Executable Code Actions Elicit Better LLM Agents|Executable Code Actions Elicit Better LLM Agents]] — Proposes executable Python as a unified, composable action space and trains agents on multi-turn code-execution trajectories.
- [[wiki/sources/XGrammar - Flexible and Efficient Structured Generation Engine for Large Language Models|XGrammar: Flexible and Efficient Structured Generation Engine for Large Language Models]] — Makes grammar-constrained generation fast through cached token masks, context expansion, persistent stacks, and inference-engine overlap.
- [[wiki/sources/YaRN - Efficient Context Window Extension of Large Language Models|YaRN: Efficient Context Window Extension of Large Language Models]] — Extends RoPE context windows by selectively interpolating positional frequencies and rescaling attention, with efficient fine-tuning and limited train-short/test-long extrapolation.

## Concepts

- [[wiki/concepts/Tool-using language model agents|Tool-using language model agents]] — How ReAct and Toolformer locate tool use at different stages of the agent pipeline.
- [[wiki/concepts/Structured generation for agents|Structured generation for agents]] — Why agent outputs need machine-checkable structure and how constrained decoding guarantees syntax efficiently.
- [[wiki/concepts/Long-context language models|Long-context language models]] — Separates nominal context capacity from stable language modeling, retrieval, downstream use, and inference cost.
