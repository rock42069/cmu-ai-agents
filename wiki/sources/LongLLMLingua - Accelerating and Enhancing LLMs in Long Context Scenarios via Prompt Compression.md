# LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression

Huiqiang Jiang, Qianhui Wu, Xufang Luo, Dongsheng Li, Chin-Yew Lin, Yuqing
Yang, and Lili Qiu (ACL 2024).

## Brief

LongLLMLingua compresses long prompts using a smaller language model before
sending them to a target LLM. It ranks documents by how well they help predict
the question, allocates more token budget to relevant documents, keeps tokens
whose predictability changes most when conditioned on the question, reorders
documents to mitigate “lost in the middle,” and repairs copied entities after
generation. It often improves accuracy while reducing target-model tokens, but
compression is lossy, question-specific, and adds model-computation overhead.

## Summary

### Question-aware coarse-to-fine compression

The method assumes a prompt containing instructions, documents, and a question.
At the coarse level, a small model scores each document by the negative
log-likelihood of the question conditioned on that document plus a restrictive
instruction. Documents that make the question more predictable receive higher
importance. This differs from ordinary retrieval because it asks how the
context explains the question rather than comparing embeddings.

At the fine level, **contrastive perplexity** measures the difference between a
token's perplexity with and without the question as conditioning context. The
paper relates this to conditional pointwise mutual information. Tokens whose
distribution changes most in light of the question are retained preferentially.

Document importance also controls each document's token budget, so likely
relevant documents are compressed less aggressively. Retained documents are
reordered by relevance to place useful evidence near prompt edges, compensating
for the target model's observed position bias.

Token pruning can corrupt names and numbers copied into the answer. A
post-generation subsequence-recovery step matches response fragments back to
the compressed prompt and replaces them with the corresponding full substring
from the original prompt. This is lexical repair, not factual verification.

### Evidence

Experiments use LLaMA-2-7B-Chat as compressor and GPT-3.5-Turbo-0613 or
LongChat-13B-16K as targets. Evaluation spans NaturalQuestions, LongBench,
ZeroSCROLLS, MuSiQue, and LooGLE.

On NaturalQuestions, approximately 4x compression improves GPT-3.5 accuracy
over the original prompt; the paper highlights a 21.4% gain when the gold
document is tenth. On LongBench, the original GPT-3.5 prompt averages 44.0,
while LongLLMLingua averages 48.8 at about 3x compression and 48.3 at about 6x.
On MuSiQue it scores 51.2 versus 45.8 for the original at 2.3x compression. On
LooGLE it averages 32.1 versus 22.6 while reducing roughly 30.5K tokens to 3.1K.

For prompts around 10K tokens, end-to-end latency—including compression and
the target request—improves about 1.4-2.6x at 2-6x compression. Historical
GPT-3.5 pricing yields estimated cost savings from 52.6% on MuSiQue to 94.0% on
LooGLE; these dollar figures are workload- and time-specific.

## Key ideas to keep in mind

- Long prompts are not automatically better: irrelevant context dilutes useful
  information and middle-position evidence may be harder to use.
- Compression should be conditioned on the task. Generic surprising-token
  scores preserve information, not necessarily relevance.
- Coarse document selection and fine token pruning solve different problems;
  relevance should guide how much budget each document receives.
- Reordering can improve use of retained evidence without changing its content.
- Lossy compression can improve quality by increasing relevant-information
  density, but every removed token creates a failure mode.
- Subsequence recovery repairs damaged surface forms; it does not restore
  omitted reasoning evidence or detect hallucinations.

## Connections

- [[wiki/concepts/Prompt compression for long context|Prompt compression for long context]]
  frames compression as a relevance, fidelity, cost, and caching trade-off.
- [[wiki/concepts/Long-context language models|Long-context language models]]:
  this reduces input inside a fixed window rather than extending the window.
- [[wiki/concepts/Reusable prompt-state caching|Reusable prompt-state caching]]:
  question-aware compression must be recomputed for each question, trading
  semantic targeting against reuse of cached context states.
- [[wiki/concepts/Agent memory management|Agent memory management]]:
  compression can fit selected memory into working context, but removed evidence
  may be unrecoverable during that turn.

## Questions and caveats

- The compressor is a 7B model, so preprocessing has meaningful cost; the paper
  reports roughly twice LLMLingua's compression compute.
- Question-dependent scores require recompression for new questions and prevent
  straightforward reuse of one compressed context.
- Token pruning can destroy syntax, names, numbers, negation, and long-range
  dependencies. Subsequence recovery covers only matchable output fragments.
- Target models and API pricing are dated; absolute quality, latency, and dollar
  savings are not guarantees for current systems.
- Coarse scoring may fail when relevance is subtle, global, adversarial, or only
  emerges through multi-step reasoning, a limitation acknowledged by the paper.
- Gains partly exploit target-model position bias; stronger models or tasks with
  dense relevant evidence may show a different quality-compression curve.

## Source

- [[raw/2024.acl-long.91-longllmlingua.pdf|Local PDF]] ([file](../../raw/2024.acl-long.91-longllmlingua.pdf); SHA-256 `baddd67ba8fb10b68ab1251de9961cc1e4868ead89b97bf99a5838ea80fb4c1c`)
- [[raw/2024.acl-long.91-longllmlingua.LICENSE|Attribution and CC BY 4.0 license record]]
- [Canonical ACL Anthology record](https://aclanthology.org/2024.acl-long.91/)
- ACL 2024, pages 1658-1677; DOI 10.18653/v1/2024.acl-long.91
