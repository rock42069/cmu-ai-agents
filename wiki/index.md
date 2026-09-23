# Wiki index

## Sources

- [[wiki/sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer: Language Models Can Teach Themselves to Use Tools]] — Learns when and how to call textual APIs by generating candidate calls, executing them, and retaining calls that improve future-token prediction.
- [[wiki/sources/ReAct - Synergizing Reasoning and Acting in Language Models|ReAct: Synergizing Reasoning and Acting in Language Models]] — Interleaves explicit reasoning, external actions, and observations so each can guide the others during task execution.
- [[wiki/sources/What Are Tools Anyway - A Survey from the Language Model Perspective|What Are Tools Anyway? A Survey from the Language Model Perspective]] — Defines LM tools as external program interfaces and surveys tool functions, learning methods, creation, evaluation, and cost trade-offs.
- [[wiki/sources/Executable Code Actions Elicit Better LLM Agents|Executable Code Actions Elicit Better LLM Agents]] — Proposes executable Python as a unified, composable action space and trains agents on multi-turn code-execution trajectories.
- [[wiki/sources/XGrammar - Flexible and Efficient Structured Generation Engine for Large Language Models|XGrammar: Flexible and Efficient Structured Generation Engine for Large Language Models]] — Makes grammar-constrained generation fast through cached token masks, context expansion, persistent stacks, and inference-engine overlap.
- [[wiki/sources/YaRN - Efficient Context Window Extension of Large Language Models|YaRN: Efficient Context Window Extension of Large Language Models]] — Extends RoPE context windows by selectively interpolating positional frequencies and rescaling attention, with efficient fine-tuning and limited train-short/test-long extrapolation.
- [[wiki/sources/Ring Attention with Blockwise Transformers for Near-Infinite Context|Ring Attention with Blockwise Transformers for Near-Infinite Context]] — Distributes exact blockwise attention over a device ring so per-device memory depends on local blocks and context capacity scales with hardware.
- [[wiki/sources/Prompt Cache - Modular Attention Reuse for Low-Latency Inference|Prompt Cache: Modular Attention Reuse for Low-Latency Inference]] — Reuses position-aware KV states for recurring prompt modules across requests to reduce prefill and time-to-first-token latency.
- [[wiki/sources/MemGPT - Towards LLMs as Operating Systems|MemGPT: Towards LLMs as Operating Systems]] — Gives a fixed-context agent a hierarchy of working, recall, and archival memory managed through tool calls, pressure events, and iterative retrieval.
- [[wiki/sources/LongLLMLingua - Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression|LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression]] — Uses question-aware document ranking, token compression, reordering, and subsequence recovery to shorten long prompts while retaining task-relevant evidence.

## Concepts

- [[wiki/concepts/Tool-using language model agents|Tool-using language model agents]] — How ReAct and Toolformer locate tool use at different stages of the agent pipeline.
- [[wiki/concepts/Structured generation for agents|Structured generation for agents]] — Why agent outputs need machine-checkable structure and how constrained decoding guarantees syntax efficiently.
- [[wiki/concepts/Long-context language models|Long-context language models]] — Separates nominal context capacity, positional adaptation, distributed execution, retrieval, downstream use, and total inference cost.
- [[wiki/concepts/Reusable prompt-state caching|Reusable prompt-state caching]] — How recurring prompt components become versioned inference artifacts, and why reuse boundaries affect correctness, latency, memory, and isolation.
- [[wiki/concepts/Agent memory management|Agent memory management]] — Memory as an auditable policy for writing, retrieving, compressing, updating, and fitting durable information into working context.
- [[wiki/concepts/Prompt compression for long context|Prompt compression for long context]] — The relevance, fidelity, cost, ordering, traceability, and caching trade-offs of lossy context reduction.
