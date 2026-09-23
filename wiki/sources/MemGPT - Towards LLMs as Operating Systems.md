# MemGPT: Towards LLMs as Operating Systems

Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, Ion
Stoica, and Joseph E. Gonzalez (arXiv v2, 2024).

## Brief

MemGPT treats an LLM's context window as scarce working memory rather than its
entire memory. An agent uses tool calls to move information among a read-write
working context, recent-message queue, complete conversation store, and archival
database; memory-pressure events and function chaining let it save, retrieve,
and revise information before responding. The paper demonstrates stronger
long-term conversational recall and iterative document retrieval, but the
“virtual context” remains a fallible policy implemented by the underlying LLM,
not an actually infinite or automatically reliable context window.

## Summary

### Virtual context management

The operating-system analogy separates **main context**, visible during
inference, from **external context**, which is persistent but must be retrieved.
Main context contains static system instructions, a fixed-size read-write
working context, and a FIFO queue of recent messages. External context contains
recall storage for the full message history and archival storage for arbitrary
text objects and embeddings.

A queue manager appends messages and monitors token usage. Around a warning
threshold (the example uses 70%), it injects a memory-pressure event so the LLM
can preserve important facts. At the hard threshold, it evicts older messages,
stores them in recall storage, and updates a recursive summary. This gives the
model both a lossy synopsis and access to the original history.

The LLM edits working memory and queries external memory through validated
function calls. A `request_heartbeat` flag immediately returns control after a
tool result, allowing it to page through search results or perform multi-hop
lookups before yielding to the user. Retrieval policy, memory writing, and
control flow are therefore partly delegated to the model itself.

### Conversational-memory evidence

The authors augment Multi-Session Chat with a deep-memory retrieval task: after
five sessions, the agent must answer a narrow question about an old conversation.
Fixed-context baselines receive a lossy summary; MemGPT can query the complete
history. Accuracy rises from 38.7% to 66.9% with GPT-3.5 Turbo, from 32.1% to
92.5% with GPT-4, and from 35.3% to 93.4% with GPT-4 Turbo. Answers are graded
by an LLM judge and also compared with ROUGE-L.

In a conversation-opener task, models use stored persona and conversation
history to initiate a new session. Similarity scores show strong persona
alignment, but are indirect proxies for engagement rather than a user study.

### Document-analysis evidence

For NaturalQuestions-Open, archival memory contains embeddings from 20 million
Wikipedia passages and the evaluation samples 50 questions. Unlike a baseline
receiving one fixed top-k retrieval set, MemGPT can reformulate searches and
page through results. Its success still depends on deciding to continue: the
paper observes that it often stops before exhausting the ranking, and GPT-3.5
performs poorly because of weaker function calling.

The synthetic nested key-value task uses 140 UUID pairs (about 8K tokens),
varies lookup depth from zero to four, and tests 30 orderings. GPT-3.5 fails at
one nested lookup and GPT-4/GPT-4 Turbo reach zero by three; MemGPT with GPT-4
stays accurate across the tested depths. Other MemGPT backends improve over
their baselines but degrade, showing that orchestration does not remove
dependence on the underlying model's agent competence.

## Key ideas to keep in mind

- Long-term agent memory is an active data-management loop: decide what to
  write, retrieve, keep in working context, and when to stop.
- Durable storage is distinct from the model-visible working set. External
  information has no effect until retrieval places it back into the prompt.
- Memory-pressure signals turn context limits into observable events the agent
  can react to before eviction.
- Function chaining enables pagination, reformulation, and multi-hop retrieval.
- Recursive summaries are lossy indexes, not substitutes for source history.
- The memory system's ceiling is its policy quality: poor writing, searching,
  updating, or stopping decisions produce poor memory.

## Connections

- [[wiki/concepts/Agent memory management|Agent memory management]] separates
  durable storage, retrieval, working context, and memory-control policy.
- [[wiki/concepts/Tool-using language model agents|Tool-using language model agents]]:
  MemGPT makes memory operations tools inside an observation-action loop.
- [[wiki/concepts/Long-context language models|Long-context language models]]:
  virtual context works around a fixed window; it does not extend positional
  capacity or make all stored information simultaneously visible.
- [[wiki/sources/ReAct - Synergizing Reasoning and Acting in Language Models|ReAct]]
  interleaves reasoning, actions, and observations; MemGPT applies that pattern
  specifically to memory operations.

## Questions and caveats

- Experiments use proprietary 2023-era OpenAI models, small samples, synthetic
  tasks, and LLM judging. Results may not transfer to real deployments.
- The paper does not rigorously measure memory-write precision, forgotten
  important facts, contradictions, or recovery after incorrect updates.
- Iterative searches increase latency, tokens, tool calls, and opportunities to
  stop too early or retrieve misleading evidence.
- Persistent histories require retention, deletion, privacy, provenance, and
  access-control policies beyond the model-level mechanism.
- “Unbounded” describes expandable external storage. Each inference still sees
  a bounded prompt and only a selected subset of memory.
- The paper has no dedicated limitations section; these caveats follow from its
  architecture, evaluation design, and reported failures.

## Source

- [[raw/2310.08560-memgpt.pdf|Local PDF]] ([file](../../raw/2310.08560-memgpt.pdf); SHA-256 `9f674bcff69c86f11c813dcfad613d8841f5f8ed17979e3c4df06a91df7762e0`)
- [[raw/2310.08560-memgpt.LICENSE|Attribution and CC BY 4.0 license record]]
- [Canonical arXiv record](https://arxiv.org/abs/2310.08560)
- arXiv:2310.08560v2, revised February 12, 2024
