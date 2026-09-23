# Agent memory management

An agent's memory is not everything stored about its past. It is the system that
decides what to retain, how to represent it, what to retrieve, what to expose to
the model now, and when to revise or forget it.

## Storage is not working memory

[[wiki/sources/MemGPT - Towards LLMs as Operating Systems|MemGPT]] provides a
useful hierarchy:

- **working context** contains the small set of facts deliberately kept visible;
- a **recent-message queue** preserves local conversational continuity;
- **recall storage** retains complete interaction history; and
- **archival storage** holds searchable documents and other long-lived data.

Only information placed into the current prompt affects an inference. External
storage may be vast, but forgotten, poorly indexed, or prematurely abandoned
searches make it functionally absent.

## Memory is a tool-use policy

MemGPT exposes writing, editing, searching, and pagination as tools. The model
receives observations such as memory pressure and search results, then chooses
another memory action or yields to the user. This places memory inside the same
feedback loop as other agent tools:

1. notice that current context is insufficient or nearly full;
2. decide what information matters;
3. write, evict, summarize, search, or page;
4. inspect the result and possibly repeat; and
5. stop when enough grounded evidence is in working context.

Each step can fail independently. A retrieval index cannot recover a fact that
was never stored; a perfect store is useless if the model asks the wrong query;
and relevant results still fail if the model stops too early or overwrites a
correct memory with an incorrect update.

## Fitting retrieved memory into context

Retrieval often returns more than the current window should hold.
[[wiki/concepts/Prompt compression for long context|Prompt compression]] can
increase relevant-information density, but it is lossy and may remove evidence
needed for later reasoning. Summaries have the same trade-off. Durable systems
should preserve links to original records so compressed working representations
remain inspectable and refreshable.

## Operational requirements

Persistent agent memory needs provenance, timestamps, versioning, conflict
handling, access control, deletion, and retention policies. Memory writes should
identify whether they are user statements, observations, model inferences, or
summaries; otherwise uncertain synthesis can silently become “fact.”

## Durable takeaway

Treat memory as a managed, auditable data path rather than a larger prompt.
Separate durable records from model-visible working context, and evaluate the
write, retrieval, compression, update, and stopping policies—not storage size
alone.

## Sources

- [[wiki/sources/MemGPT - Towards LLMs as Operating Systems|MemGPT: Towards LLMs as Operating Systems]]
- [[wiki/sources/LongLLMLingua - Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression|LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression]]
