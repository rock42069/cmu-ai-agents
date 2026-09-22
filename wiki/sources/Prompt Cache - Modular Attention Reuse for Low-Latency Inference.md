# Prompt Cache: Modular Attention Reuse for Low-Latency Inference

In Gim, Guojun Chen, Seung-seob Lee, Nikhil Sarda, Anurag Khandelwal, and Lin
Zhong (MLSys 2024).

## Brief

Prompt Cache reduces time-to-first-token by reusing precomputed key-value
attention states across requests that share prompt segments. A schema exposes
reusable system messages, templates, documents, or code files as positioned
modules; inference copies their cached states and computes only new text. The
prototype reports 5-10x GPU TTFT improvements when modules reside in GPU memory
and up to 70x on the tested CPUs. This is not always exact reuse: independently
encoding modules prevents attention across their boundaries, so modularity,
quality, memory, and latency have to be designed together.

## Summary

### From per-request KV caching to inter-request reuse

Ordinary KV caching computes the input prompt's key and value states once and
reuses them while output tokens are generated. It avoids recomputing earlier
tokens during decoding, but each new request still pays the full prompt-prefill
cost. Prompt Cache observes that many requests share large pieces: system
prompts, few-shot examples, prompt templates, reference documents, user-profile
traits, or source files.

It precomputes these recurring segments as **prompt modules** and reuses their
KV states across requests. Cached inference concatenates the selected modules'
states, computes states for uncached instructions and parameter values, and
uses the result as the initial KV cache. The optimization therefore targets
prefill and time-to-first-token (TTFT); subsequent token generation has the
same cost as conventional KV-cached decoding.

### Why a schema is necessary

KV states depend on token positions, so the same text cannot be moved freely
and retain the same cached representation. Prompt Cache introduces Prompt
Markup Language (PML), which defines a schema of named modules and reserves
stable position-ID ranges for them. A request derived from the schema imports
selected modules and supplies uncached text.

PML supports:

- **parameters**, which reserve a fixed positional gap for a runtime value;
- **unions**, whose mutually exclusive alternatives share a starting position;
- **nested modules** for hierarchical prompt structures;
- **scaffolds**, which precompute a group jointly when cross-module attention
  must be preserved; and
- model-specific system, user, and assistant tags.

This produces discontinuous position IDs when a request selects only some
modules. The authors find that the tested models tolerate these gaps as long as
relative positions within a segment are preserved. Their prototype modifies
RoPE and ALiBi implementations to look up the positional transformation for
the supplied IDs; embedding-table models already accept arbitrary IDs.

### The approximation hidden inside modular reuse

A module encoded by itself cannot attend to modules that will later surround
it. Concatenating independently computed KV states is therefore equivalent to
masking attention across module boundaries during prefill. This can help when
modules are semantically independent, but it can hurt when meaning depends on
their interaction. Scaffolding jointly encodes a known group to restore its
shared attention span, at the cost of storing another, larger cached variant.

Parameters also replace reserved placeholder states with states computed for
the runtime value. These mechanisms make the layout reusable but mean Prompt
Cache is not generally bit-for-bit equivalent to precomputing the complete
assembled prompt.

### Implementation and results

The roughly 3K-line PyTorch/Hugging Face prototype stores modules in either GPU
HBM or CPU DRAM. GPU storage gives the lowest latency but limited capacity;
CPU storage is larger but requires host-to-device copies. A buffered
concatenation implementation avoids repeated tensor allocations.

Using Llama2 7B on eight LongBench datasets with prompts averaging about 5K
tokens, GPU TTFT improves 5-10x when modules are in GPU memory and 1.5-3x when
they are copied from CPU memory. CPU-only inference improves by as much as 70x
on an Intel i9-13900K and 20x on an AMD Ryzen 9 7950X. In a synthetic 3K-token
test on an RTX 4090, TTFT falls from 900 ms to 90 ms, while subsequent-token
latency remains about 32 ms for both approaches.

The LongBench quality comparison covers Llama2 7B/13B, MPT 7B, and Falcon 7B.
Most cached scores are close to the full-prefill baseline, but changes are not
uniform: for example, Llama2 passage-retrieval accuracy falls from 7.50 to 4.25
at 7B and from 9.08 to 6.50 at 13B, while several multi-hop QA scores improve.
The table therefore supports approximate quality preservation on average, not
strict output equivalence.

Cache storage is substantial and model-dependent. At 16-bit precision, the
paper reports about 0.50 MB per cached token for Llama 7B, 0.78 MB for Llama
13B, and 2.5 MB for Llama 70B. A 1K-token module thus needs about 500 MB for
Llama 7B or 2.5 GB for Llama 70B before replication and scaffold variants.

## Key ideas to keep in mind

- KV reuse can cross request boundaries when repeated prompt structure is made
  explicit and assigned stable positions.
- Prompt caching accelerates prefill and TTFT, not the cost of generating each
  later output token.
- The most reusable unit is not necessarily the largest repeated string. Module
  boundaries determine which tokens can attend to one another during prefill.
- Moving cached states is linear in prompt length, while recomputing dense
  prefill contains a quadratic attention term; longer repeated prompts create
  more opportunity for reuse.
- Cache location is a latency-capacity trade-off: GPU memory is fast and scarce;
  CPU memory is plentiful but introduces transfer latency.
- Reusable prompts become versioned runtime artifacts tied to the exact model,
  tokenizer, positional layout, and module contents.

## Connections

- [[wiki/concepts/Reusable prompt-state caching|Reusable prompt-state caching]]
  develops the implications for agent runtimes, repeated tool descriptions,
  templates, and retrieved context.
- [[wiki/concepts/Long-context language models|Long-context language models]]:
  longer prompts increase prefill cost; caching makes repeated long context
  cheaper but neither expands the model's window nor improves its ability to
  use distant evidence.
- [[wiki/concepts/Tool-using language model agents|Tool-using language model agents]]:
  agent requests often repeat system instructions, tool schemas, demonstrations,
  and stable environment documentation, creating natural cache candidates.

## Questions and caveats

- Independently encoded modules impose cross-module attention masking. The
  paper's average benchmark results should not be read as an exactness proof for
  arbitrary prompts; scaffolding restores interactions only at extra memory cost.
- The approach requires requests to follow a declared schema and reserves
  positional gaps, which can consume context positions and complicate dynamic
  prompts whose structure is not known in advance.
- Cached states are model-, tokenizer-, precision-, and position-specific.
  Model updates or prompt edits require invalidation and recomputation.
- Memory cost can dominate at scale: thousands of tokens across many documents,
  tenants, model replicas, or scaffold combinations quickly reach tens or
  hundreds of gigabytes.
- The prototype does not supply a production cache replacement, admission,
  eviction, prefetching, compression, or multi-tenant isolation policy.
- TTFT speedups depend on the repeated fraction, prompt length, hardware,
  memory placement, and model size. They do not imply equal improvements in
  end-to-end latency for responses with many generated tokens.
- Reusing user- or tenant-specific states introduces privacy and access-control
  concerns that the paper does not evaluate; this is an operational inference,
  not a reported experimental result.

## Source

- [[raw/2311.04934-prompt-cache|Source record and local-reading-copy details]]
- [Canonical arXiv record](https://arxiv.org/abs/2311.04934)
- arXiv:2311.04934v2, revised April 25, 2024; MLSys 2024
