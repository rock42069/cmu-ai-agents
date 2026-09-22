# Long-context language models

A model's context window is the maximum sequence length it can accept under a
particular configuration. Its **effective context** is the portion it can use
reliably for the task at hand. These are not the same quantity.

## Four claims that should be kept separate

1. **The input fits.** The model and inference system can process the advertised
   number of tokens without a positional or systems failure.
2. **Language modeling remains stable.** Predictive quality, often measured by
   perplexity, does not collapse as the sequence grows.
3. **Information remains accessible.** The model can retrieve evidence from
   different positions, including the middle of a long prompt.
4. **The evidence is used well.** Retrieval, synthesis, reasoning, and final
   task performance remain dependable across the full context.

Passing one layer does not establish the next. A synthetic passkey can remain
retrievable even after perplexity worsens, while perfect retrieval of a simple
value says little about integrating many conflicting pieces of evidence.

## What YaRN contributes

[[wiki/sources/YaRN - Efficient Context Window Extension of Large Language Models|YaRN]]
addresses the first two layers and tests a narrow form of the third. It adapts
pretrained RoPE frequencies according to their wavelengths: local-frequency
dimensions are preserved, dimensions tied to the old range are interpolated,
and intermediate dimensions are blended. Attention scaling further improves
the distribution at long positions.

Its results demonstrate that positional behavior can be adapted much more
cheaply than repeating pretraining, and that fine-tuning examples need not be
as long as the final evaluated window. They do not show that arbitrary tasks
benefit from placing all available material in the prompt.

## Systems consequence

Positional extension and efficient attention are separate problems. YaRN adds
no material overhead relative to another RoPE configuration at the same length,
but it does not change ordinary attention's growth in compute and KV-cache
memory.

[[wiki/sources/Ring Attention with Blockwise Transformers for Near-Infinite Context|Ring Attention]]
addresses the complementary systems constraint. It shards a sequence across
devices and circulates key-value blocks while exact blockwise attention runs,
making per-device activation memory depend on local block size rather than the
global sequence length. If computation hides communication, supported sequence
length scales roughly with device count.

This makes dense long-context attention fit; it does not make all pairwise
attention free. The total arithmetic still grows with sequence length for a
fixed dataset, and the hardware requirement grows with the desired capacity. A
usable long-context system therefore needs both a model that remains capable at
distant positions and an inference stack that can afford them.

For an agent, long context is likewise not identical to memory. Context is a
bounded working set supplied to one inference; durable memory additionally
requires deciding what to retain, retrieve, update, and discard.

Long prompts also impose prefill latency even when they fit. If substantial
segments recur across requests,
[[wiki/concepts/Reusable prompt-state caching|reusable prompt-state caching]]
can skip some repeated computation. This is a serving optimization, not an
increase in context capacity or evidence that the model uses the context well.

## Durable takeaway

Treat a context-window number as a capacity claim, not a capability guarantee.
Evaluate long-context systems with multiple position-sensitive tests and the
actual downstream task, while accounting separately for model quality and
inference cost.

## Sources

- [[wiki/sources/YaRN - Efficient Context Window Extension of Large Language Models|YaRN: Efficient Context Window Extension of Large Language Models]]
- [[wiki/sources/Ring Attention with Blockwise Transformers for Near-Infinite Context|Ring Attention with Blockwise Transformers for Near-Infinite Context]]
- [[wiki/sources/Prompt Cache - Modular Attention Reuse for Low-Latency Inference|Prompt Cache: Modular Attention Reuse for Low-Latency Inference]]
