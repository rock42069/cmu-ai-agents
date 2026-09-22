# Reusable prompt-state caching

LLM requests often contain a large stable prefix or recurring components:
system instructions, tool definitions, few-shot examples, reference documents,
application templates, or code files. Their transformer states can sometimes
be computed once and reused, turning part of prompt construction into a systems
artifact rather than repeated inference work.

## Three reuse levels

1. **Within one generation:** a conventional KV cache retains states for the
   prompt and generated tokens so decoding does not recompute the full prefix.
2. **Across requests with an identical prefix:** multiple requests can share a
   previously computed prefix cache when their tokens and positions match.
3. **Across modular prompts:**
   [[wiki/sources/Prompt Cache - Modular Attention Reuse for Low-Latency Inference|Prompt Cache]]
   assigns stable positional ranges to reusable components so requests can
   assemble selected cached modules with newly computed text.

The third level creates more reuse but weakens a key invariant. A cached prefix
has already attended to everything before it exactly as in the new request. An
independently encoded module has not attended to other modules that may later
appear beside it. Modular concatenation therefore changes the attention graph
unless related modules are precomputed together.

## Why this matters for agents

Agent loops frequently resend stable material while only observations and the
latest instruction change. Candidate cache units include the agent policy,
tool schemas, output-format rules, demonstrations, repository documentation,
and stable retrieved sources. Reuse can substantially reduce prefill latency,
especially when prompts are long and outputs are short.

This optimization should influence prompt architecture. A good cache module is:

- reused often enough to repay encoding and storage;
- stable under model, tokenizer, and prompt-version changes;
- positionally predictable; and
- semantically self-contained, or deliberately scaffolded with the modules it
  must attend to.

## Cache correctness is more than a hit

A byte-identical text match is insufficient if the model, tokenizer, numerical
format, adapter, positional layout, attention mask, or surrounding dependency
contract changed. Cache keys and invalidation rules need to cover these inputs.
In a multi-user system, authorization must also be checked before cached states
derived from private content are reused.

## Relationship to long context

[[wiki/concepts/Long-context language models|Long-context capability]] and
prompt-state reuse solve different problems. A long-context model determines
how much input can fit and be used; caching determines how much repeated prefill
work can be skipped. Caching neither extends the context window nor guarantees
better retrieval or reasoning, though it may make repeated long inputs much
cheaper to serve.

## Durable takeaway

Treat recurring prompt components like compiled assets: version them, define
their dependencies, measure their reuse, and invalidate them safely. Optimize
module boundaries for both semantic interaction and systems reuse rather than
assuming every repeated string is independently cacheable.

## Sources

- [[wiki/sources/Prompt Cache - Modular Attention Reuse for Low-Latency Inference|Prompt Cache: Modular Attention Reuse for Low-Latency Inference]]
