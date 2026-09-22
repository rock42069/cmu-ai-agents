# Ring Attention with Blockwise Transformers for Near-Infinite Context

Hao Liu, Matei Zaharia, and Pieter Abbeel (arXiv v4, 2023).

## Brief

Ring Attention is a distributed implementation of exact self-attention for very
long sequences. It shards a sequence across devices, keeps each query block
local, and circulates key-value blocks around a device ring. Each device attends
its queries to one key-value block while the next block is transferred, so
communication can be hidden behind sufficiently large block computations. The
method reduces per-device activation memory and lets supported sequence length
grow roughly with the number of participating devices, but it does not reduce
the total quadratic arithmetic of dense attention.

## Summary

### The remaining memory bottleneck

Blockwise attention avoids materializing the full quadratic attention matrix,
and blockwise feedforward computation reduces large intermediate activations.
Even then, a conventional implementation retains every layer's sequence-wide
outputs because the next layer must attend over all positions. That remaining
activation storage grows linearly with sequence length on each device and
eventually exhausts its high-bandwidth memory.

Ring Attention partitions the sequence dimension so each device owns one query,
key, and value block. For a layer, the query block stays on its owner while key
and value blocks move to the next device in a logical ring. After one circuit,
every query block has interacted with every key-value block, so the result is
the same dense attention operation rather than an approximation.

Blockwise softmax can process key-value blocks in any order if it maintains the
appropriate running maximum, normalization denominator, and weighted-value
numerator. This permits each device to compute on its current block while
sending it onward and receiving the next. The paper gives the condition for
hiding communication as block size `c >= F/B`, where `F` is device FLOP rate
and `B` is interconnect bandwidth. Fast links need blocks of roughly 1K tokens
in the paper's hardware model; A100s connected by the much slower InfiniBand
configuration need about 24.5K.

The resulting per-device activation bound depends on the fixed block size and
hidden width rather than the global sequence length. In the paper's accounting,
Ring Attention uses `6bch` bytes per layer, compared with `2bsh` for blockwise
attention and feedforward without sequence sharding, where `s` is global
sequence length and `c` is local block size.

### Systems evidence

The maximum-length tests combine Ring Attention with fully sharded data
parallelism across A100 GPU and TPU configurations. On 8 A100s, the reported
maximum training context is 512K for a 3B model, 256K for 7B, and 128K for 13B:
eight times the blockwise-attention-and-feedforward baseline. On 32 A100s the
7B model reaches 4.096M and the 13B model 2.048M, a 32x increase. The largest
listed run is a 3B model at 16.384M tokens on TPUv4-1024; other configurations
range from 1.024M to 8.192M.

Across five configurations from 7B to 65B and contexts from 128K to 2.048M,
the reported model-FLOPs utilization is close to the same blockwise transformer
implementation at shorter context, roughly 31-37%. This supports the conditional
claim that communication is mostly hidden when blocks are large enough and the
interconnect is fast enough.

### Capability experiments

The paper applies the larger capacity in two task studies. An in-context
reinforcement-learning model conditions on 128 rather than 32 trajectories and
improves average ExoRL return from 111.13 to 113.66 across six tasks. This is a
small but consistent gain in the reported table.

For language modeling, the authors fine-tune LLaMA-13B at 512K context on 32
A100 80GB GPUs using cleaned ShareGPT conversations. On a synthetic line-number
retrieval test, the resulting model remains around 90% accurate near 500K
tokens. This demonstrates that the trained model can access a simple distant
fact; it is not a broad evaluation of comprehension or reasoning at that length.

## Key ideas to keep in mind

- Ring Attention is exact sequence parallelism: distribute the sequence and
  stream key-value blocks rather than approximating or sparsifying attention.
- Communication can be made nearly invisible only when local block computation
  lasts at least as long as the transfer. Hardware balance and interconnect
  topology are part of the algorithm's practical contract.
- The method scales **capacity per sequence** with device count. It exchanges
  the data-parallel option of processing more independent sequences for one
  much longer sequence while holding the total token batch comparable.
- Reducing per-device memory does not reduce dense attention's total arithmetic.
  Longer exact attention remains expensive even when it fits and parallelizes.
- A system that can execute a million-token sequence is not automatically a
  model that learned to use a million tokens effectively.

## Connections

- [[wiki/concepts/Long-context language models|Long-context language models]]
  distinguishes Ring Attention's systems capacity from positional adaptation
  and downstream long-context capability.
- [[wiki/sources/YaRN - Efficient Context Window Extension of Large Language Models|YaRN]]
  modifies RoPE so a pretrained model behaves sensibly at new positions. Ring
  Attention instead makes exact attention over those positions fit across
  devices; the two approaches address different constraints.

## Questions and caveats

- The headline “near-infinite” description means capacity can continue scaling
  with more devices; it is not an unbounded resource or constant-cost method.
- The paper's “no overhead” claim is conditional on communication being fully
  overlapped and is relative to performing the same dense attention work. Slow
  interconnects, small blocks, synchronization, and topology can expose costs.
- Exact dense attention still performs all query-key interactions. The appendix
  estimates that moving a 7B model from 4K to 1M context costs about 37.4x more
  training FLOPs per fixed-size dataset, and 10M costs about 439.7x more.
- The measured maximum in Table 3 is 16.384M tokens. The paper's broader claim
  of sequences exceeding 100M is not accompanied by a corresponding measured
  configuration in that table and should not be treated as equally evidenced.
- The 512K language-model experiment uses one model family, one fine-tuning
  dataset, and a simple synthetic retrieval test; it does not establish general
  quality throughout the full window.
- The paper has no dedicated limitations section. These caveats are synthesis
  from its resource assumptions, reported configurations, and evaluations.

## Source

- [[raw/2310.01889-ring-attention|Source record and local-reading-copy details]]
- [Canonical arXiv record](https://arxiv.org/abs/2310.01889)
- arXiv:2310.01889v4, revised November 27, 2023
