---
title:  "LLM compression"
date:   2026-06-15 14:34:00 +0100
---

LLM compression is the set of techniques used to make large language models cheaper, faster, smaller, and easier to deploy while preserving as much quality as possible. It can compress the model itself, the runtime state used during inference, or the input context sent to the model.

## What is LLM compression?

LLM compression means reducing one or more of the resources required to use a model:

| Resource | Why it matters |
|---|---|
| Disk size | Model distribution, storage cost, startup time |
| RAM/VRAM | Whether the model fits and how many requests can run concurrently |
| Memory bandwidth | Often the dominant bottleneck for token generation |
| Compute | Matrix multiply cost, attention cost, and server throughput |

Compression can target different parts of the system:

| Target | What is compressed | Typical benefit |
|---|---|---|
| Model weights | Parameters stored in matrices | Smaller model files and lower VRAM |
| Activations | Intermediate tensors during forward pass | Lower memory and faster compute |
| KV cache | Attention keys/values saved during generation | Lower VRAM for long context and concurrency |
| Input tokens | API cost, prefill latency, and context-window pressure |
| Trainable parameters | Fine-tuning cost and adapter storage |
| Architecture | Layers, heads, hidden size, routing | Better quality per compute unit |

## Why would you comperss LLMs?

### Fit models on available hardware

Compression can change a deployment from "does not fit" to "fits on one machine". Compression can enable:

- Running local models on laptops.
- Serving models on fewer GPUs.
- Deploying models to edge devices.
- Hosting larger models in a fixed memory budget.

For example:

| Model size | FP16/BF16 rough weight memory | 8-bit rough memory | 4-bit rough memory |
|---:|---:|---:|---:|
| 7B | 14 GB | 7 GB | 3.5 GB |
| 13B | 26 GB | 13 GB | 6.5 GB |
| 34B | 68 GB | 34 GB | 17 GB |
| 70B | 140 GB | 70 GB | 35 GB |
| 405B | 810 GB | 405 GB | 202.5 GB |

These are simplified weight-only estimates. Real deployments also need KV cache, runtime overhead, and sometimes duplicated weights across replicas or tensor-parallel shards.

### Reduce inference cost

Serving cost is often driven by GPU memory, memory bandwidth, and request concurrency. Smaller weights and caches can allow:

- More requests per GPU.
- Larger batch sizes.
- Fewer GPUs per replica.
- Lower latency under load.
- Cheaper instance types.

### Improve latency and throughput

Compression can improve performance by:

- Moving less data from memory.
- Increasing effective memory bandwidth.
- Allowing larger batches.
- Reducing CPU/GPU transfer.
- Enabling faster low-precision matrix kernels.

However, smaller does not automatically mean faster. Speed depends on whether the hardware and inference engine have optimized kernels for the compressed format.

### Support long context

For chat, RAG, coding agents, and document workflows, the KV cache can dominate memory. KV cache memory grows with layer count, KV heads, head dimension, sequence length, batch size, and precision. Compressing weights does not solve a KV cache bottleneck by itself.

Long-context workloads can become KV-cache-bound even when weights fit comfortably. KV cache compression is therefore critical for:

- Chatbots with long histories.
- Retrieval-augmented generation over large context.
- Coding agents with many files in context.
- Document analysis.
- Multi-turn customer support.
- High-concurrency API serving.

### Enable local, edge, or private deployment

Quantized and distilled models are easier to run on laptops, workstations, edge devices, and private infrastructure. This makes it more practical to run models on-device or on-premises, in disconnected environments or regulated environments where data cannot leave the network, in low-latency applications where cloud roundtrips are unacceptable.

### Lower energy and infrastructure footprint

At high request volume, smaller models and more efficient inference can reduce GPU-hours, power, cooling and hardware requirements, reducing environmental impact.

## How is LLM compression done?

The main options are:

| Option | What it compresses | Main benefit | Main risk |
|---|---|---|---|
| Quantization | Weights, activations, or KV cache | Lower memory, lower bandwidth, often faster inference | Quality loss if bit width, calibration, or kernels are wrong |
| Pruning and sparsity | Weights, neurons, heads, layers, or tokens | Smaller model or less compute | Speedups require sparse-aware hardware/runtime |
| Knowledge distillation | Train a smaller student model to imitate a larger teacher model | Real reduction in model size and latency | Requires data, training, and careful evaluation |
| Low-rank factorization | Weight matrices | Fewer parameters in selected layers | Can hurt quality; savings depend on rank and implementation |
| Weight sharing/codebooks | Weight storage | Smaller files and sometimes memory | Usually limited runtime speedup without custom kernels |
| Efficient architecture design | Model structure | Best quality per compute when training/building a model | Not a drop-in compression step for an existing checkpoint |
| KV cache compression | Runtime attention cache | More concurrency and longer context | Can damage long-context reasoning or retrieval |
| Prompt/context compression | Input tokens | Immediate cost and latency savings | Can remove information the model needed |

### Quantization

Quantization is the most widely used LLM compression method. It represents numeric values with fewer bits.

| Target | Common formats | Notes |
|---|---|---|
| Weights only | W8A16, W4A16, GPTQ, AWQ, GGUF quants | Most common for local and open-model inference |
| Weights and activations | W8A8, FP8, W4A8 | More speed potential, more kernel and calibration sensitivity |
| KV cache | FP8, INT4, INT2, NVFP4, custom vector quantization | Critical for long context and high concurrency |
| Fine-tuning state | 4-bit base weights with adapters | Often used for memory-efficient fine-tuning rather than final serving |

At a high level, quantization maps high-precision values to lower-precision buckets.

Example:

```text
Original FP16 values:
[-1.72, -0.41, 0.03, 0.88, 1.96]

Quantized 4-bit codes:
[1, 6, 8, 11, 15]

Metadata:
scale and zero-point or other reconstruction parameters
```

At inference time, the runtime either:

- Dequantizes values back to higher precision before computation.
- Uses low-precision kernels directly.
- Uses a hybrid approach where storage is low precision but accumulation is higher precision.

Pros:

- Usually the fastest path to lower memory use.
- Often no retraining required.
- Large ecosystem support.
- Can reduce cost dramatically.
- Makes larger models accessible on smaller hardware.

Cons:

- Can reduce quality.
- Can increase hallucinations or brittle behavior if too aggressive.
- Can hurt difficult tasks more than easy chat tasks.
- Requires runtime-specific compatibility.
- Speedups are not guaranteed without optimized kernels.
- Evaluation must be task-specific.

### Pruning and sparsity

Pruning removes parts of a model that appear less important. Sparsity keeps the original shape but sets some weights or structures to zero so they can be skipped by supported kernels.

| Pruning type | What is removed | Example |
|---|---|---|
| Unstructured pruning | Individual weights | Set 50% of low-importance weights to zero |
| Semi-structured pruning | Fixed sparse patterns | NVIDIA-style 2:4 sparsity |
| Structured pruning | Channels, heads, neurons, layers | Remove attention heads or MLP neurons |
| Layer dropping | Entire transformer blocks | Distill or fine-tune after removing layers |
| Token pruning | Less important tokens during inference | Reduce attention workload |

Pros:

- Can reduce both parameters and computation.
- Structured pruning can produce a genuinely smaller dense model.
- Can combine with quantization.
- Useful for research and specialized hardware.

Cons:

- Not always faster in practice.
- Sparse formats and kernels are runtime-dependent.
- May require fine-tuning or distillation to recover quality.
- Aggressive pruning can break reasoning and instruction following.
- Can remove rare but important capabilities.
- More difficult to deploy than quantization.

### Knowledge distillation

Distillation trains a smaller student model to imitate a larger teacher model. The teacher may provide labels, logits, chain-of-thought rationales, tool-call traces, critique data, preference data, or synthetic instruction data.

The output is not a compressed copy of the same model. It is a smaller model trained to reproduce selected behaviors of the larger model.

| Type | Description | Common use |
|---|---|---|
| Response distillation | Student imitates teacher answers | Instruction tuning |
| Logit distillation | Student matches teacher probability distribution | More information-rich but requires teacher logits |
| Step/rationale distillation | Student learns reasoning traces | Math, code, planning |
| Tool-use distillation | Student learns when/how to call tools | Agentic workflows |
| Preference distillation | Student learns teacher or human preferences | Alignment |
| Domain distillation | Student learns specialized domain behavior | Enterprise assistants |

Pros:

- Produces a genuinely smaller model.
- Can reduce latency and cost substantially.
- Can specialize a model for a domain, product, or workflow.
- Can be combined with quantization after training.
- Often better than simply pruning an existing model.

Cons:

- Requires training data and training infrastructure.
- Student inherits teacher errors and blind spots.
- Broad generality is hard to preserve.
- Distillation can violate model-provider terms if not permitted.
- Evaluation must cover reliability, safety, and edge cases, not just average accuracy.

### Low-rank factorization

Many neural network layers contain large dense matrices. Low-rank factorization approximates a large matrix as the product of smaller matrices.

Pros:

- Mathematically clean.
- Can combine with quantization.
- Can reduce parameter count in selected layers.
- Can be layered with fine-tuning.

Cons:

- Drop-in compression may require fine-tuning.
- Speedup depends on matrix sizes and kernels.
- Not always competitive with quantization for simple deployment.
- Choosing ranks layer-by-layer is nontrivial.

### Weight sharing, clustering, and entropy coding

These methods reduce storage by making many weights share values or by encoding weights more compactly.

Pros:

- Useful for distribution and storage.
- Can be combined with quantization.
- Good for many model variants with small differences.
- Reduces parameter count.
- Can be effective when designed into the model.
- May preserve depth while reducing storage.

Cons:

- Often helps disk size more than runtime latency.
- Needs custom kernels or decompression strategy for serving speed.
- Added indirection can hurt performance.
- Hard to apply after training without degradation.
- May reduce representational capacity.
- Usually requires training or fine-tuning.
- Less common as a practical post-training LLM compression method.

### Efficient architecture design

Sometimes the best compression is not compressing a finished model but designing or selecting a better-sized model. Important architecture-level efficiency techniques include:

- Grouped-query attention and multi-query attention
- Mixture-of-experts
- Sliding-window and sparse attention
- Smaller dense models trained better

Pros:

- Can outperform post-hoc compression.
- Better control over latency, memory, and deployment shape.
- Can avoid quality loss from compressing the wrong model.

Cons:

- Requires training or selecting a new model.
- Not a direct transformation of an existing checkpoint.
- More evaluation and migration work.

### KV cache compression

During autoregressive generation, a transformer stores key and value tensors for previous tokens so it does not have to recompute them. This is the KV cache. The KV cache grows linearly with sequence length and batch size. For long-context and high-concurrency serving, KV cache memory can dominate. Weight compression helps fit the model. KV cache compression helps serve real workloads.

| Method | Description | Benefit | Risk |
|---|---|---|---|
| KV quantization | Store cache in INT8, INT4, FP8, etc. | Lower VRAM | Quality loss if too aggressive |
| Token eviction | Drop less important cached tokens | Lower memory | Irreversible forgetting |
| Token merging | Combine similar tokens or cache entries | Lower memory | Loss of detail |
| Attention sinks | Preserve special high-importance early tokens | Better stability | Heuristic-dependent |
| Sliding cache | Keep recent window plus selected global tokens | Long-context efficiency | Misses distant details |
| CPU offload | Move cache from GPU to CPU memory | Larger effective context | Transfer latency |
| Hierarchical cache | Hot cache on GPU, cold cache elsewhere | Better memory use | System complexity |
| Low-rank KV | Approximate cache tensors with lower-rank forms | Memory reduction | Reconstruction error |
| Prefix caching | Reuse cache for shared prompts | Throughput improvement | Less useful for unique prompts |

Pros

- Enables longer context.
- Improves concurrency.
- Reduces serving cost.
- Helps avoid GPU memory fragmentation and out-of-memory errors.
- Can be combined with quantized weights.

Cons

- Quality degradation can be subtle and task-dependent.
- Retrieval tests may not reveal reasoning damage.
- Some methods help memory but add latency.
- System implementations are more complex than weight-only quantization.

### Prompt and context compression

Prompt compression reduces the number of input tokens before the model sees them. This is not model compression, but it often has the fastest business impact because API and prefill costs scale with input tokens.

| Method | Description |
|---|---|
| Summarization | Replace long text with a shorter summary |
| Extractive selection | Keep only relevant passages |
| Reranking | Retrieve many chunks, send only top-ranked chunks |
| Deduplication | Remove repeated or near-duplicate content |
| Structured extraction | Convert documents into concise facts or fields |
| Conversation memory | Summarize old turns and keep recent turns verbatim |
| Query-focused compression | Compress context specifically for the current question |
| Embedding-based filtering | Drop chunks with low semantic relevance |
| Prompt template minimization | Remove unnecessary instruction verbosity |

Pros

- Works with closed and open models.
- No model modification required.
- Reduces cost and prefill latency immediately.
- Can improve answer quality by removing irrelevant context.

Cons

- Compression may remove details needed later.
- Summaries can introduce errors.
- Relevance filtering can miss subtle dependencies.
- Requires careful evaluation for legal, medical, financial, and code tasks.
- Over-compression can make the model confidently wrong.
- Compression logic becomes part of the product behavior.

### Combining compression methods

Compression methods are often combined.

| Combination | Why use it |
|---|---|
| Quantization + KV cache quantization | Reduce both model and runtime memory |
| Distillation + quantization | Build a small student, then compress further |
| Pruning + quantization | Reduce parameters and precision |
| Context compression + KV cache compression | Reduce both input length and long-context memory |
| Efficient architecture + quantization | Best practical deployment footprint |

## Evaluation

Compression should be evaluated across four dimensions:

- Quality
- Performance
- Cost
- Reliability

Quality metrics measure:

- Human preference ratings.
- LLM-as-judge with calibration.
- Domain expert review.
- Tool-call correctness.
- Safety and refusal behavior.
- Regression tests for known prompts.

Performance metrics measure:

- Time to first token.
- Tokens per second per request.
- Maximum context length.
- Maximum concurrent users.
- GPU utilization.
- Memory bandwidth utilization.

Cost metrics measure:

- Cost per 1M input tokens.
- Cost per 1M output tokens.
- Cost per successful task.
- GPU-hours.
- Power consumption.
- Required hardware tier.
- Engineer time and operational complexity.

Reliability metrics measure:

- Out-of-memory frequency.
- Error rates.
- Latency spikes.
- Behavior on unusual prompts.
- Long-running serving stability.

## Security, privacy, and governance

Compression changes model behavior and deployment patterns. It should be treated as a model change, not only an infrastructure optimization.

Consider:

- Whether compressed models still follow safety policies.
- Whether refusals and guardrails survive compression.
- Whether domain-specific factuality changes.
- Whether private training data was used for distillation.
- Whether teacher-model terms permit distillation.
- Whether quantized community checkpoints are trustworthy.
- Whether model files came from a verified source.
- Whether compression tools execute untrusted code.

