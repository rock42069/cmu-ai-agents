# Prompt compression for long context

Prompt compression selects or rewrites input so a smaller sequence preserves
the information needed for a task. It can reduce latency and cost, fit evidence
inside a context window, and sometimes improve performance by removing noise.
It is also a lossy transformation whose failures may be invisible to the target
model.

## Selection should match the task

Generic compression preserves broadly informative or surprising content.
[[wiki/sources/LongLLMLingua - Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression|LongLLMLingua]]
instead conditions document and token importance on the current question. This
raises relevant-information density but makes the compressed artifact specific
to that question.

Useful systems separate:

1. **coarse selection** of documents, passages, or memory records;
2. **fine compression** within selected material;
3. **ordering** of retained evidence for the target model; and
4. **traceability** back to original text for verification or recovery.

## Compression is not context extension

Compression reduces the material placed in the window. It does not increase
positional capacity, preserve every dependency, or allow access to discarded
tokens. It therefore complements—but does not replace—techniques in
[[wiki/concepts/Long-context language models|Long-context language models]].

## Relationship to caching

Question-conditioned compression and
[[wiki/concepts/Reusable prompt-state caching|prompt-state caching]] pull in
opposite directions. Tailoring content to each question improves relevance but
reduces reuse across questions. Task-level cached summaries may be cheaper to
reuse but less precise. The balance depends on query repetition, compression
cost, target-model prefill cost, and quality risk.

## What to measure

Do not evaluate compression by token ratio alone. Measure downstream quality,
recall of necessary evidence, faithfulness, compression overhead, target-model
latency and cost, and sensitivity to different queries. High averages can hide
catastrophic failures on negation, numbers, names, code syntax, or multi-hop
dependencies.

## Durable takeaway

Compression is an information-selection policy with a budget, not a free speed
optimization. Preserve source traceability, make the task signal explicit, and
choose the compression ratio against the cost of omitting required evidence.

## Sources

- [[wiki/sources/LongLLMLingua - Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression|LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression]]
