# XGrammar: Flexible and Efficient Structured Generation Engine for Large Language Models

Yixin Dong, Charlie F. Ruan, Yaxing Cai, Ruihang Lai, Ziyi Xu, Yilong Zhao,
and Tianqi Chen (MLSys 2025).

## Brief

XGrammar is a constrained-decoding engine that guarantees an LLM's output
matches a context-free grammar while making grammar enforcement cheap enough
for production serving. Its key systems insight is that the validity of most
vocabulary tokens depends only on the parser's local state and can be cached;
only a small minority need the full recursive parse stack at runtime. Combined
with context expansion, persistent stacks, automaton optimizations, and overlap
with GPU inference, this largely removes the CPU bottleneck of structured output.

## Summary

### The problem

Agent outputs such as JSON function calls, SQL, XML, and domain-specific code
must be syntactically consumable by downstream software. Constrained decoding
enforces this by masking every token that would violate a grammar before the
sampler chooses the next token. Context-free grammars are more expressive than
regular expressions because they represent recursive structures, but a naive
engine may interpret the grammar against an entire vocabulary of more than
100,000 tokens at every decoding step. LLM tokens also contain multiple bytes
and may cross grammar-rule boundaries, so checking them can require traversing
and rolling back recursive parser stacks.

### Architecture

XGrammar converts a CFG into a byte-level pushdown automaton (PDA). The byte
level handles tokens that do not align with characters or UTF-8 boundaries. A
generated prefix may correspond to several valid parse stacks when the grammar
is ambiguous; XGrammar unions the tokens accepted by those stacks.

The main optimization divides vocabulary tokens at each automaton node into:

- **context-independent tokens**, whose validity can be decided from the top
  automaton state and precomputed; and
- **context-dependent tokens**, which cross a rule boundary and require the
  complete stack at runtime.

For Llama 3.1 with a JSON grammar, fewer than 1% of the 128K vocabulary tokens
initially require runtime checking. The adaptive token-mask cache stores the
smaller of the accepted or rejected sets for each state; on that example it
reduces cache memory from 160 MB to 0.46 MB.

Context expansion analyzes what can legally follow when a child rule returns to
its parent. It rejects many apparently context-dependent tokens in advance,
reducing the cited JSON case from 1,134 runtime-checked tokens to 120. A
persistent execution stack shares prefixes among alternate parser states and
makes branching and rollback cheap. Node merging and bounded rule inlining
simplify the PDA, while lexicographically ordered vocabulary checks reuse common
token prefixes during preprocessing.

Finally, mask generation runs on the CPU concurrently with GPU model inference.
Because the next mask depends on the previously sampled token, its computation
can overlap with the GPU work that produces the next logits.

### Results

The engine is evaluated on JSON Schema, recursive unconstrained JSON, XML, and
a Python-like DSL with Llama-3.1-8B-Instruct. Against the tested versions of
Outlines, llama.cpp grammar, and lm-format-enforcer, XGrammar reports under 40
microseconds per token for the JSON tasks and under 200 microseconds for XML and
the Python DSL—up to roughly 3x faster for JSON Schema and more than 100x for
CFG tasks than the best applicable baseline.

The ablation identifies the adaptive token-mask cache as the dominant gain:
latency falls from 38.280 ms after node merging to 0.154 ms after adding the
cache, then to 0.018 ms after inlining and context expansion. In SGLang serving
on an H100, XGrammar produces up to an 80x end-to-end improvement over the
compared grammar integrations under high load. MLC-LLM experiments report only
about 0.0-0.2 ms additional time per output token relative to unconstrained
generation. The tested outputs reach 100% syntactic correctness for function
calling and XML, versus 62% and 80% without grammar constraints.

## Key ideas to keep in mind

- Structured generation is a token-masking problem coupled to a parser state;
  it need not require retraining the language model.
- Precompute the common case and interpret only the exceptional tokens at
  runtime. This changes the problem from scanning the vocabulary to resolving a
  small state-dependent remainder.
- Token boundaries are not grammar boundaries. Correct implementations must
  handle multi-character, partial-UTF-8, and cross-rule tokens.
- Recursive grammars require stack state; persistent data structures make
  speculative checks, ambiguity, branching, and rollback practical.
- CPU grammar work can be hidden behind GPU inference when the engine pipeline
  is designed jointly rather than as two independent components.
- A syntactically valid tool call may still be semantically wrong, unsafe, or
  unsupported. Grammar enforcement solves only the structural layer.

## Connections

- [[wiki/concepts/Structured generation for agents|Structured generation for agents]]
- [[wiki/concepts/Tool-using language model agents|Tool-using language model agents]]
- [[wiki/sources/Executable Code Actions Elicit Better LLM Agents|CodeAct]] benefits
  from executable code as a flexible action representation; XGrammar addresses
  how code or structured calls can be constrained efficiently.
- [[wiki/sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer]]
  learns when and how to emit textual tool calls, whereas XGrammar can guarantee
  that the emitted representation belongs to a prescribed language.

## Questions and caveats

- The quality experiment measures syntactic validity, not whether a function
  name, argument value, SQL query, or agent action is correct.
- Speedups depend on the chosen grammar, vocabulary, baseline versions,
  batching, serving engine, and CPU/GPU hardware; “near-zero overhead” is not a
  universal latency guarantee.
- XML and the Python DSL use synthetic evaluation data, and the Python grammar
  intentionally omits indentation.
- Preprocessing and cache construction still have costs even when per-token
  runtime checks are cheap.
- A grammar can enforce only properties encoded in it; permissions, business
  rules, state-dependent validity, and side-effect safety require other layers.

## Source

- [[raw/2411.15100-xgrammar.pdf|Local PDF]] ([file](../../raw/2411.15100-xgrammar.pdf); SHA-256 `61e94a1a28bb6307250c4f82a9ff0dcd07273fed80011d6ba7535f388a8fa5f0`)
- [[raw/2411.15100-xgrammar.LICENSE|Attribution and CC BY-SA 4.0 license record]]
- [Canonical arXiv record](https://arxiv.org/abs/2411.15100)
- arXiv:2411.15100v3, revised May 12, 2025; MLSys 2025
