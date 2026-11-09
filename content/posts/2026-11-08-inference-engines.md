---
title: "Inference engines"
date:   2026-11-08 19:11:00 +0100
---

## Overview

An inference engine is software that runs a model to produce output based on input from incoming request. On high level, it provides runtime environment to execute the model. There are multiple implementations and they differ in hardware support, model capabilities, performance priorities, deployment etc.

### Responsibilities

No matter the implementation, all inference engines have several responsibilities:

| Responsibility | Description |
| --- | --- |
| Load model weights | Load the trained parameters into the memory used for inference. |
| Manage request state | Maintain intermediate state, including key and value tensors for active requests. |
| Schedule requests | Select which requests to process and how to group concurrent work. |
| Compute outputs | Execute the model operations that produce tokens. |
| Serve requests | Expose an API through which applications submit requests and receive responses. |

Here weights are the trained model parameters, while request state is created and maintained during inference. The distinction is important from operational perspective: weights impose a relatively fixed memory cost, whereas request state varies with active workloads.

### Inference stack

An inference engine operates within a serving system that separates hardware computation, request execution, deployment management, and traffic routing.

| Layer | Responsibility |
| --- | --- |
| Kernels | Execute arithmetic operations on the hardware. |
| Inference engine | Load weights, manage memory, schedule requests, compute outputs, and expose an API. |
| Control plane | Deploy and manage inference engines and their lifecycles. |
| Gateway | Route requests across models and replicas. |

In this table, kernels represent computational code, not operating-system kernels. They are often benchmarked independently, although deployments commonly use the implementations provided by their inference engine. The engine-level mechanisms discussed below (batching, paging, quantization and prefix caching), address different constraints within this layered system.

### Prefill and decode

Request processing has two phases with different computational characteristics.

| Phase | Work performed | Typical bottleneck |
| --- | --- | --- |
| Prefill | Processes the prompt, including instructions, tool definitions, conversation history, and the latest message. Much of this work can run in parallel. | For substantial inputs, often compute capacity. |
| Decode | Generates output tokens sequentially; each token depends on preceding context. | For a single request, often memory bandwidth, as each step reads substantial model data from GPU memory. |

Long inputs and long outputs can therefore stress different resources on the same GPU. The distinction is useful when input and output lengths produce different performance constraints. Decode is often bandwidth-bound for a single request, but this is a common characteristic rather than a guarantee for every workload.

### GPU memory

Model weights and the key-value (KV) cache are two major consumers of GPU memory, but their usage patterns differ.

| Resource | Memory behavior | Operational implication |
| --- | --- | --- |
| Weights | Loaded once and generally fixed for a running model. | The selected model and precision determine a baseline memory requirement. |
| KV cache | Stores reusable intermediate state for active requests and grows with context length and concurrency. | Available cache capacity constrains the number and length of concurrent requests. |

For a model with 8 billion parameters stored at two bytes per parameter, the nominal weight estimate is approximately 8B × 2 bytes ≈ 16 GB. This estimate excludes the runtime buffers and cache required during serving.

Weights and the KV cache occupy the same GPU memory and compete for available capacity.

For requests of similar length, ten simultaneous conversations can require roughly ten times the KV cache of one conversation, before accounting for shared prefixes. Longer contexts and greater concurrency both increase cache demand. Accordingly, weight fit is an initial deployment constraint, while cache capacity becomes more significant as context lengths and concurrency rise.

## Optimizations

Inference optimizations aim to reduce memory use, improve throughput, or lower response latency. They address different constraints by changing how weights are stored, cache is managed, requests are scheduled, or computation is reused. Their benefits depend on the model, hardware, and workload.

### Quantization

Quantization stores weights using fewer bits than their original representation. Reducing precision can reduce weight memory: halving the bits per weight can roughly halve the memory those weights occupy. For weights estimated at 16 GB, this could free about 8 GB, depending on the quantization method and its overhead.

The resulting capacity can be used for additional KV cache, supporting more or longer concurrent requests. Quantization can therefore be useful even when the unquantized model already fits in memory.

The quality impact and overhead vary with the quantization method, model, and task. Some workloads are more sensitive to quantization than others, so the tradeoff requires evaluation for the intended use case.

Quantization can also be applied to the KV cache rather than only to model weights. Storing cached keys and values at lower precision reduces memory per cached token, potentially supporting longer contexts or more concurrent requests. As with weight quantization, the quality impact and performance benefit depend on the method, model, and hardware support.

### Paging and paged attention

The amount of KV cache required by a request is not known in advance. Reserving enough memory for the longest possible context can waste capacity when most requests use only a fraction of their reservation.

Paging allocates KV cache in small, fixed-size blocks as requests need them. Paged attention uses this block-based organization to access cached keys and values. This approach is useful when requests vary substantially in length, as fixed reservations can otherwise leave significant amounts of GPU memory reserved but unused.

### Optimized attention

Attention computation moves data between GPU memory and faster on-chip memory. Optimized implementations organize this computation to reduce those transfers and avoid storing the full attention-score matrix in GPU memory. This can reduce temporary memory use and improve performance, particularly when processing long prompts.

Optimized attention and paged attention address different concerns: the former improves how attention is computed, while the latter enables attention to access a KV cache stored in separately allocated blocks. Engines can combine these techniques.

### Continuous batching

Batching groups requests so the GPU can process multiple requests in parallel. In static (traditional) batching, the engine forms a batch, processes it, and waits for all requests in it to finish before starting another batch. Requests with short outputs can finish before longer requests, leaving their batch slots unused until the other requests complete.

Continuous batching replaces completed requests with waiting requests as capacity becomes available. The batch composition can therefore change as requests finish, helping keep available processing capacity occupied. Batching is relevant once requests are concurrent. Continuous batching can reduce the time short requests remain grouped behind unrelated longer requests.

### Chunked prefill

Processing a long prompt in one uninterrupted prefill pass can delay token generation for requests already being served. Chunked prefill divides prompt processing into smaller pieces so the scheduler can interleave them with decode work from other requests.

Continuous batching changes which requests participate in ongoing execution, while chunked prefill controls how much prompt-processing work is scheduled at a time. Combining them can improve responsiveness under mixed workloads, although chunk size and scheduling policy affect the tradeoff between prompt-processing time, generation latency, and throughput.

### Prefix caching

Before generating an answer, an engine processes the input, which can include system instructions, tool definitions, and conversation history in addition to the latest user message. Repeated requests may share a substantial initial sequence of tokens.

Prefix caching reuses cached state when another request begins with the same initial tokens. It avoids reprocessing the shared portion and processes the remaining input for the new request. Reuse is limited when requests do not share an identical initial token sequence. It can be especially useful for workloads with long, repeated prefixes, including agent requests.

### Speculative decoding

Speculative decoding uses a smaller draft model to propose several upcoming tokens, which a larger model then checks in a single pass. When enough proposed tokens are accepted, multiple tokens can be produced with less work than generating and checking each token independently. The benefit depends on how accurately the draft model predicts tokens the larger model will accept.

This technique targets generation latency for an individual request, rather than primarily increasing the efficiency of serving concurrent requests.

## Operations

Inference operations cover how models are deployed, distributed across hardware, and made available to incoming requests. This includes managing deployments, routing traffic, and adjusting capacity as demand changes. The appropriate design depends on model size, workload, and reliability requirements.

### Parallelism across GPUs

Models that exceed the memory capacity of one GPU may be divided across multiple GPUs. Two common forms of model parallelism have different communication patterns.

| Approach | Partitioning strategy | Operational consideration |
| --- | --- | --- |
| Tensor parallelism | Splits computations within model layers across GPUs. | Requires communication between GPUs for partitioned layer computations. |
| Pipeline parallelism | Assigns different layers or groups of layers to different GPUs. | Uses communication between stages as requests progress through layer groups. |

The resulting communication patterns affect hardware requirements and performance. Sharding is relevant when a model does not fit on one GPU, so the memory constraint should be established before deployment.

### Deployment and traffic management

Deployment and traffic management cover where inference engines run, how requests reach them, and how serving capacity changes with demand. Control planes manage deployments, gateways route requests, and autoscaling adjusts the number of serving replicas.

#### Control planes

A deployment may begin with one engine serving one model on one GPU. As the number of models, replicas, or pre-production versions grows, deployment operations must determine where workloads run, how updates are applied, and how failures are handled. A control plane manages deployments and their lifecycles, and can replace manually maintained deployment configuration.

#### Gateways and routing

A gateway selects the model or replica that receives each request. Simple round-robin routing may not account for differences in cached context across replicas. If one replica holds a conversation's prefix while another does not, routing to the latter can require reprocessing that prefix and affect latency and cost. Cache-aware routing accounts for this difference when selecting a destination.

Deployment management and routing require greater attention as a system grows beyond a single model and replica. Their detailed designs depend on the deployment and workload.

#### Autoscaling

Autoscaling adjusts serving capacity as demand changes. For inference workloads, relevant signals can include queue depth and KV cache utilization; CPU utilization alone often provides little information about GPU inference pressure.

Newly provisioned GPU capacity is not immediately ready to serve requests. A node may need to download a container image, load model weights, and initialize the engine. This cold-start delay can outlast the traffic increase that triggered scaling and makes scaling to zero more difficult. Autoscaling can reduce the cost of idle GPU capacity, but its benefits depend on how quickly new capacity becomes usable.

### Prefill–decode disaggregation

Prefill and decode place different demands on hardware. Prefill–decode disaggregation runs the two phases on separate machines so each can use resources suited to its needs. Its design depends on the distinction between compute capacity, memory capacity, memory bandwidth, and request scheduling. The additional deployment complexity is relevant only when workload characteristics and scale justify separating the phases; many deployments may not need it.

## Conclusion

Inference engines turn trained models into usable services by managing computation, memory and request execution. The most useful optimizations depend on the constraint: model size, context length, concurrency or response latency. Deployment, routing and scaling decisions determine how those engine capabilities translate into a reliable service. Start with the workload's requirements, measure its bottlenecks and add complexity only where it provides a clear benefit.
