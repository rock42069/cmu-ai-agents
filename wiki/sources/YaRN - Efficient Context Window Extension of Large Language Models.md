# YaRN: Efficient Context Window Extension of Large Language Models

Bowen Peng, Jeffrey Quesnelle, Honglu Fan, and Enrico Shippole (arXiv v3,
2026; first submitted 2023).

## Brief

YaRN extends the usable context window of pretrained models that use Rotary
Position Embeddings (RoPE). Rather than compressing every RoPE frequency by the
same amount, it preserves high-frequency dimensions that encode nearby token
relationships, interpolates low-frequency dimensions that would otherwise move
out of their training range, blends between the two, and rescales attention.
On the tested LLaMA and Llama 2 models, this combination reaches long contexts
with substantially less fine-tuning than prior position interpolation methods.

## Summary

### The problem

A RoPE model generally deteriorates when asked to process positions beyond the
maximum length seen in pretraining. Position Interpolation (PI) avoids unseen
positions by compressing all positions back into the original range, but it
also compresses every RoPE frequency equally. The authors argue that this
destroys high-frequency positional information needed to distinguish nearby
tokens, especially at large extension factors.

RoPE dimensions have different wavelengths. Short-wavelength dimensions make
many rotations inside the original window and primarily represent local,
relative distance. Long-wavelength dimensions may not complete a rotation and
can therefore expose position patterns tied to the original absolute range.
These two groups should not be transformed identically.

### The method

YaRN combines two changes:

1. **NTK-by-parts interpolation.** Dimensions are treated according to how many
   rotations their wavelength completes inside the original context. Short
   wavelengths are left alone, long wavelengths are fully interpolated, and a
   ramp blends the treatment of intermediate dimensions. For the LLaMA family,
   the paper reports useful rotation-count boundaries of 1 and 32.
2. **Attention scaling.** The method applies a scale-dependent temperature to
   the pre-softmax attention logits. It implements this by scaling the rotary
   embeddings, so fixed-length RoPE caches absorb the change without extra
   per-token compute or an attention-kernel modification.

The paper also describes **Dynamic Scaling** for inference: recompute the
extension factor from the current sequence length instead of fixing it to the
maximum target length. This preserves the original behavior while the sequence
is short and degrades more gradually beyond the trained window. With KV
caching, the cache must precede the changing RoPE transformation; already
rotated keys cannot simply be reused under a new scale.

### Evidence

The main experiments extend Llama 2 7B and 13B from 4K to 64K and 128K. The 64K
models are fine-tuned for 400 steps on 64K-token PG19 chunks; the 128K models
start from those checkpoints and receive 200 additional steps on the same
64K-length data. On ten long Proof-pile documents, the 128K models maintain
finite sliding-window perplexity through 128K despite never training on 128K
sequences. The reported 128K perplexities are 2.37 for 7B and 2.24 for 13B.

In a controlled LLaMA 7B ablation with a 32K target and equal 400-step budgets,
YaRN has lower long-range perplexity than PI, NTK-aware interpolation, and
NTK-by-parts alone. For a smaller 4K-to-8K extension, YaRN reaches essentially
the same perplexity as a PI baseline using 400 rather than 1,000 steps. The
paper summarizes this as 2.5x fewer steps and, given the training setup, 10x
fewer tokens than the earlier method.

The 128K models average 99.4% on the paper's passkey-retrieval trials through
128K. Short-context benchmark scores remain close to the original Llama 2
baselines, although MMLU declines somewhat as the extension factor grows. The
appendix also shows that retrieval can remain strong even when perplexity has
started to worsen, so no single metric fully describes effective context.

## Key ideas to keep in mind

- RoPE frequencies serve different positional roles; uniform interpolation
  needlessly sacrifices local resolution.
- Context extension can be framed as keeping each frequency in the regime where
  the pretrained model learned to use it, rather than merely changing a single
  maximum-length number.
- YaRN is a composition: frequency-selective interpolation supplies the
  positional transformation, while attention scaling corrects a separate
  distributional effect.
- “Train short, test long” is possible here: the 128K models were fine-tuned on
  64K sequences, though this is an empirical result for the tested setup rather
  than a universal guarantee.
- A declared context window is not proof of useful context. Perplexity,
  retrieval, and downstream-task quality test different capabilities.
- Changing positional encodings does not reduce the attention computation or
  memory required for a longer sequence.

## Connections

- [[wiki/concepts/Long-context language models|Long-context language models]]
  separates nominal context size, language-modeling quality, retrieval, and
  downstream usefulness.

## Questions and caveats

- The principal model experiments use older LLaMA/Llama 2 7B and 13B models.
  The recommended constants and attention-scaling fit are empirical and may not
  transfer unchanged to other architectures, RoPE variants, or training data.
- Proof-pile perplexity is based on ten selected documents. GovReport uses 50
  documents, and the 128K passkey table uses ten trials per tested length. These
  are useful signals but limited samples.
- Passkey retrieval is deliberately simple and does not establish reliable
  reasoning, synthesis, or instruction following across a 128K prompt.
- The comparison of total training cost includes models and training runs from
  different sources; it is not a single fully controlled systems benchmark.
- “No additional computational or memory cost” means no overhead relative to
  other RoPE interpolation schemes at the same sequence length. Standard
  attention still becomes much more expensive as the sequence grows.
- The paper does not present a dedicated limitations section. The caveats above
  are synthesis from its methods, sample sizes, metrics, and experimental scope.

## Source

- [[raw/2309.00071-yarn.pdf|Local PDF]] ([file](../../raw/2309.00071-yarn.pdf); SHA-256 `e7c0268a796138460c6ba2f67a7cba5bd922c401aea94c1fcd09c4b76b883c85`)
- [[raw/2309.00071-yarn.LICENSE|Attribution and CC BY 4.0 license record]]
- [Canonical arXiv record](https://arxiv.org/abs/2309.00071)
- arXiv:2309.00071v3, revised February 6, 2026
