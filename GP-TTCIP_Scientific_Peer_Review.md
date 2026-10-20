# Scientific peer review: Beyond the Token Bottleneck

**Manuscript:** *Beyond the Token Bottleneck: A Two-Tier Cognitive Interconnect Architecture for High-Throughput Multi-Agent LLM Communication*  
**Reviewed file:** `paper.md`  
**Review date:** 19 October 2026  
**Review type:** Technical peer review, primary-source verification, analytical checks, and proposed experimental redesign  
**Manuscript fingerprint, SHA-256:** `c70501b5755236746e263e923584b7379c7ba66defa204a9300a5e56a198e363`

**Reading guide:** [Recommendation](#1-recommendation-and-principal-findings) · [Claim assessment](#3-claim-level-assessment) · [Reference audit](#4-audit-of-the-20-references) · [Performance analysis](#5-performance-and-information-theoretic-analysis) · [Architecture](#6-architecture-and-representation-findings) · [Reliability and safety](#7-uncertainty-safety-and-distributed-execution) · [Alternatives](#8-alternative-designs-serving-the-same-goal) · [Experimental plan](#9-falsifiable-evaluation-plan) · [Reproducible calculations](#appendix-a-reproducible-arithmetic-and-counterexamples)

## 1. Recommendation and principal findings

**Recommendation: substantial revision before this manuscript can support its scientific conclusions.** As an empirical systems paper, it is not ready for acceptance: it supplies neither an implemented system nor experiments establishing its headline results. As an architectural position paper, it contains a useful organizing idea, but several specifications are internally inconsistent and several supporting studies are interpreted too broadly.

The useful idea is to distinguish **communication through compatible shared state** from **communication across incompatible models and machines**. That distinction can guide a productive research program. Avoiding repeated prefill, unnecessary message generation, and redundant state movement are worthwhile goals. Explicit handling of provenance, precision, and speculative dependencies also deserves attention.

The manuscript does **not** establish that these ideas deliver 25–100× end-to-end acceleration, improved reasoning accuracy, universal compatibility, or complete auditability. These are proposed outcomes. The manuscript repeatedly moves from evidence that a component can help in one setting to numerical guarantees for a substantially different system.

The most consequential findings are:

1. **The performance argument lacks a measured baseline and critical-path model.** An 8-byte cache handle does not make the referenced cache free to read or eliminate the receiver's reasoning. A communication-stage speedup cannot be labeled an end-to-end speedup without accounting for all remaining work.
2. **The transceiver size conflicts with its equation.** With the manuscript's million-feature dictionary and example hidden width of 8,192, its dense encoder alone has **8,589,934,592 weights**, before bias, decoder, graph construction, or confidence estimation. The stated approximately 50-million-parameter transceiver requires a different, explicitly specified architecture.
3. **The wire format is incomplete.** The fields sum to exactly 1,500 bytes, leaving no room for IP/transport headers within a 1,500-byte IP MTU. Its 512-byte adjacency block describes a binary 64×64 directed graph, with no specified representation of multiple relation types.
4. **Representation similarity is being mistaken for a functioning semantic interface.** Alignment of document embeddings does not demonstrate interchangeable middle-layer reasoning states, preservation of logical scope, or successful independent training against a universal concept dictionary.
5. **Some cited evidence directly weakens the framing.** DiffMAS explicitly freezes the backbone while training adapters. HyLaT's main table gives approximately 8.95× acceleration against its natural-language baseline but 3.72× against its trained text baseline; its accuracy advantage is not uniform. PagedAttention already implements shared pages and copy-on-write. [R19] [R14] [R22]
6. **The confidence and security guarantees are unsupported.** Token entropy is not a calibrated probability that a proposition is true. Sparse feature IDs and FP16 magnitudes still admit substantial communication capacity. Orthogonal rotations do not authenticate sources or establish truth.
7. **Distributed rollback cannot complete globally in one hardware cycle.** A local pointer update can be fast; notification, dependency invalidation, cancellation, replay, and recovery of external effects are separate operations.

These findings do **not** prove that compact latent communication is ineffective. Existing work provides evidence that it can help. They show that TTCIP's particular quantitative and universal claims do not follow from the manuscript's evidence or design.

**Most promising revision:** retain the deployment distinction; establish exact, compatibility-checked cache sharing as the first implementation; build a compact structured-message baseline for heterogeneous agents; evaluate learned latent compression as an optional third mode. Develop hardware only after profiling identifies a stable bottleneck.

## 2. Review method, scope, and confidence

### 2.1 Research strategy

Before examining supporting results, I separated the manuscript's claims into five classes: bibliographic facts, reported experimental findings, mathematical specifications, systems-performance predictions, and security guarantees. Each requires different evidence. A numerical contradiction can be checked directly; a latency prediction requires an implementation and workload; a universal semantic guarantee requires considerably more than an embedding-retrieval experiment.

The review used the following procedure:

1. Read the complete supplied manuscript, including its confidence table and bibliography.
2. Resolve all 20 bibliography entries against primary publication records or primary full text. Inspect relevant methods, tables, and limitations for the claims carrying the main argument.
3. Check missing related work, particularly existing copy-on-write cache sharing, sparse cache retention, adapter training, and structured communication.
4. Recalculate parameter counts, memory sizes, packet budgets, information capacity, and performance bounds. Run a small Python audit and synthetic counterexamples; reproducible code appears in Appendix A.
5. Seek counterevidence to both the manuscript and this review. In particular, distinguish a false universal claim from a potentially valid result under narrower assumptions.
6. Design experiments that could falsify the revised proposal, with explicit baselines, quality constraints, and deployment boundaries.

Sources were prioritized as original papers, official project materials, and standards. Search snippets served as discovery aids; central judgments use primary text. The bibliography below records versioned links where available. This is a targeted technical review, **not a systematic meta-analysis of the entire field**. The paper's own “meta-analysis” label should likewise be changed unless it adds a documented selection process and comparable effect-size synthesis.

### 2.2 What was and was not tested

**Performed:** manuscript inspection, source verification, arithmetic checks, capacity arguments, and executable toy counterexamples demonstrating limitations of proposed metrics.

**Not performed:** training a transceiver, reproducing the cited benchmark results, running TTCIP, measuring GPU/CXL traffic, or conducting an adversarial evaluation of an implemented protocol. The attachment provides no code, checkpoints, dataset, trace, or hardware implementation enabling such a replication.

Published results are treated as **results reported by their authors**, not independently reproduced facts. A preprint's existence does not validate its experiments. Similarly, a toy counterexample establishes that a proposed guarantee does not follow mathematically; it does not estimate how often an actual model will fail.

### 2.3 Interpretation of confidence

“Demonstrated inconsistency” means a claim conflicts with the stated specification or elementary arithmetic. “Unsupported” means the manuscript has not established it; it need not be false. “Conditional” means it can hold under identified assumptions. “Open” means empirical investigation is needed.

For direct arithmetic and finite-format contradictions, my confidence is very high; a material error in those conclusions is unlikely, roughly below 1% as a subjective assessment. Source-interpretation judgments are high-confidence, with roughly 1–5% subjective risk of a materially different reading where the cited passage is explicit. Conclusions about the best eventual architecture are more uncertain, roughly 10–30% risk of being materially revised after experiments. These ranges are reviewer judgments, **not statistical confidence intervals**.

I cannot defensibly assign a probability that an unimplemented TTCIP will attain 50× speedup. The manuscript's 75–90% confidence in such outcomes has no stated calibration or evidentiary basis.

## 3. Claim-level assessment

| Manuscript claim | Assessment | Main reason | Corrective action |
|---|---|---|---|
| Latent exchange can avoid some text generation and repeated prefill | Supported in specific published systems | Several studies report benefits under defined configurations | Preserve, with model/task/metric qualifications |
| Existing methods face a general 2–4.3× ceiling | Unsupported | Selected empirical results are not an upper bound | Replace with workload-specific bottleneck analysis |
| The example GQA cache uses 327,680 bytes per token | Correct under stated assumptions | Direct dimensional calculation | Use KiB/MiB consistently; avoid universalizing the architecture |
| A 2,000-token example requires 640 MB | Numerically inaccurate | It requires 655.36 decimal MB, or 625 MiB | Correct the units and trace length |
| CoW handles yield 40–80× end-to-end speedup | Unsupported | Handle traffic omits cache reads, compute, scheduling, and coherence | Measure complete request latency and throughput separately |
| Retaining 3–5% of tokens/heads preserves novel reasoning | Unsupported | Attention magnitude is not a semantic sufficiency test | Sweep retention budgets with task-level non-inferiority tests |
| FP16 predictive entropy expresses exact epistemic confidence | Incorrect as stated | Token uncertainty and proposition correctness are different quantities | Define and calibrate claim-level uncertainty |
| A dense million-feature transceiver is approximately 50M parameters | Inconsistent with the equation | Encoder alone is about 8.6B weights at width 8,192 | Specify factorization, routing, or a smaller dictionary |
| Common embedding geometry proves a universal latent ABI | Unsupported extrapolation | Document retrieval and causal use of internal states are distinct | Test independently trained adapters on held-out model pairs |
| One parallel pass replaces arbitrary multi-step reasoning | Unsupported | Compression of an available result does not compute that result | Separate reasoning, encoding, transfer, and decoding |
| The frame fits a standard WAN MTU as specified | Incorrect for ordinary IP transport | Application fields already consume 1,500 bytes | Reserve transport/security overhead and permit larger messages |
| A 512-byte binary adjacency matrix encodes typed edges | Underspecified/inconsistent | One bit per ordered pair does not identify multiple edge types | Specify typed edge lists or typed matrices |
| Sparse monosemantic features guarantee complete auditability | Unsupported | Feature interpretation does not establish compositional or causal fidelity | Evaluate causal effects and monitor coverage |
| Sparsity eliminates steganographic channels | False as a general guarantee | IDs, magnitudes, literals, timing, and alternatives retain capacity | Specify and measure bounded leakage under a threat model |
| Orthogonal provenance is cryptographic verification | Incorrect | A geometric transform is not an authenticity or truth proof | Authenticate an external evidence/dependency record |
| Distributed abort restores all agents in one clock cycle | Physically untenable as an end-to-end claim | Signals and recovery take time; side effects may persist | Separate local invalidation from distributed recovery |
| A low reconstruction residual guarantees exact tail semantics | False in general | Small-norm coordinates can determine truth | Protect exact fields and test semantic counterfactuals |
| CXL/PIM can improve selected data paths | Plausible and partly supported | Existing systems show useful benefits in specific configurations | Profile and compare against optimized existing systems |

## 4. Audit of the 20 references

The bibliography is substantially real. I found primary records or full text corresponding to all 20 entries. The central problems are **interpretation, attribution, and missing qualifications**, rather than a basis for declaring the bibliography fabricated. First-posting years and later conference years may legitimately differ.

| Ref. | Work and primary-source finding | Assessment of its use in the manuscript |
|---|---|---|
| [R01] | **Large Concept Models**, 2024, v2. Operates autoregressively over sentence embeddings. §2.5.1 analyzes theoretical FLOPs, including SONAR encoding/decoding and 40 diffusion sampling steps. | Supports a higher-level representation research direction. It does not establish one inexpensive pass per logical proposition or a measured 200–400× total compute reduction. |
| [R02] | **Fast Byte Latent Transformer**, 2026, v1. BLT-D uses blockwise diffusion; the reported bandwidth reduction is an estimate. | Method description is broadly relevant. Preserve “estimated memory-bandwidth cost”; do not turn it into measured wall-clock acceleration. Use the actual title and named authors. |
| [R03] | **Learning to Communicate with Deep Multi-Agent Reinforcement Learning**, 2016. Introduces RIAL/DIAL. | Appropriate background for learned communication. Does not validate frozen-LLM semantic interoperability. |
| [R04] | **Cache-to-Cache**, v2. Reports 6.4–14.2% average accuracy improvement over individual models, approximately 3.1–5.4% over text communication, and average 2.5× latency speedup. | Separate the comparator and units. It uses learned projection/fusion; it is not simply raw cache copying. |
| [R05] | **Coconut**, originally 2024, reviewed v4. Studies continuous thought and search-like behavior. | Relevant, but a recurrent latent step need not correspond to exactly one word token; benefits depend on training and task. It is not proof of a universal communication standard. |
| [R06] | **The Platonic Representation Hypothesis**, 2024, v5. Presents evidence, a hypothesis, limitations, and counterexamples. | “Prove isomorphic structures” overstates the source. Geometric convergence is not a proof of interchangeable internal execution state. |
| [R07] | arXiv **2607.18141v1** resolves in primary HTML to **HyMCache**, using CXL-Hybrid Memory. §4.1 describes an RDMA deployment; §2.2 attributes the 600 ns example to another pooled-memory system. | Correct the title/system naming. Do not treat a cited device-access latency as a universal GPU-to-GPU or application latency. |
| [R08] | **vec2vec**, 2025, reviewed v4. Table 2 includes up to 0.92 cosine and 100% top-1 retrieval for document embeddings in particular pairings. | The numbers are real in their setting. They do not establish exact logical transfer between middle residual streams or independent universal dictionary training. |
| [R09] | **Searching for Structure**, 2024, v3. §5.2 starts near 70% communication success and rises to about 75%, without a significant increase; 25% is chance performance. | Describing this as learning from a measured 25% starting point to 75% is misleading. Structural emergence is supported, with qualifications. |
| [R10] | **Emergent Multi-Agent Communication in the Deep Learning Era**, 2020. Survey of learned communication. | Appropriate background; no direct evidence for TTCIP's quantitative predictions. |
| [R11] | **Deal or No Deal?**, 2017. The paper describes interleaving RL and supervised updates to preserve human language. | Broad account of language drift is reasonable. The specific interpretation of repetitive utterances as rigid numerical priority codes needs separate evidence; it is not established by merely showing repetition. |
| [R12] | **DroidSpeak**, initially 2024, reviewed v4 from 2025. Reports about 3.1× faster prefill and up to 4× throughput for same-architecture models. | These are different metrics. Neither constitutes a universal ceiling on end-to-end agent performance. |
| [R13] | **Secret Collusion**, initially 2024, reviewed v5 from 2025. Studies steganographic capability and limitations of mitigation, including paraphrasing. | Relevant security motivation. It does not prove universal spontaneous collusion or that a sparse concept channel eliminates the problem. |
| [R14] | **HyLaT**, 2026, v1. Table 1: NL 13.15 s, trained text 5.47 s, HyLaT 1.47 s. §4.3 also evaluates heterogeneous agents using adapters. | Supports meaningful efficiency gains. Does not support uniform 10–20% accuracy gains or categorically fatal heterogeneous fleet lock-in. |
| [R15] | **Sparse Autoencoders Learn Monosemantic Features in Vision-Language Models**, 2025/NeurIPS, reviewed v3. Limitations explicitly distinguish interpretable features from precise causal steering. | Directly relevant counterevidence to deterministic, exactly local semantic intervention and complete auditability. |
| [R16] | **CIPHER**, initially 2023, ICLR 2024. Abstract reports 0.5–5.0% improvement over language debate. | Correct the manuscript's 1.0–5.0% lower endpoint. The mechanism avoids sampling; an expected embedding does not retain every detail of the vocabulary distribution. |
| [R17] | **KVComm**, 2025, reviewed v3. Selectively shares layers, reporting useful performance with as few as 30% of layers' KV pairs; scope includes same models and fine-tuned variants of a common base. | The “cloning an entire RAM stick” analogy misrepresents selective sharing. Layer selection is not evidence for retaining only 3–5% of arbitrary tokens/heads. |
| [R18] | **DualPath**, 2026, v2. Studies storage-I/O imbalance in disaggregated agentic serving; reports throughput improvements from a better loading path. | Supports a real I/O problem in that architecture. It proves no universal sub-5× or sub-10× acceleration bound. |
| [R19] | **Learning to Communicate / DiffMAS**, 2026, v1. Figure 1 and Appendix C describe a frozen backbone with LoRA adaptation. | Its placement under unfrozen full-weight co-training is materially misleading. Joint optimization, adapter training, and full backbone training must be distinguished. |
| [R20] | **LatentMAS**, first posted 2025, reviewed v4 from 2026. Reports up to 14.6% accuracy improvement and 4–4.3× end-to-end acceleration; heterogeneous extension is discussed separately. | Relevant empirical evidence, but the selected maximum is not a theoretical barrier. Identify the precise comparison before combining its numbers with others. |

### 4.1 Missing related work and novelty

PagedAttention and its official implementation description already include reference-counted shared KV blocks and copy-on-write. The manuscript acknowledges PagedAttention, but must identify precisely what it adds. [R21] [R22]

Two particularly relevant 2026 papers are **ForkKV**, on CoW cache organization for multi-LoRA agents, and **Zipage**, combining token eviction with paged attention and prefix caching. Their existence does not establish equivalence to TTCIP or validate their own claims, but they are necessary comparators for novelty and implementation choices. [R23] [R24]

The historical taxonomy is useful but not exhaustive. Structured symbolic messages, tool results, artifact references, learned discrete codes, and task-specific schemas are important alternatives. GibberLink's developers explicitly describe a prompted switch to ggwave; that part of the manuscript is well founded. MCP and A2A standardize different application interactions and do not themselves eliminate internal model token generation. [R35] [R36] [R37] [R38]

## 5. Performance and information-theoretic analysis

### 5.1 A token count is not a semantic information measurement

For a vocabulary of size 128,000, the maximum information associated with choosing one vocabulary entry is approximately `log2(128000) = 16.97` bits. This is an alphabet-size bound, not a measurement of useful information about a task. Conditional token entropy is generally different. A sequence, shared context, and the receiver's learned knowledge jointly determine what a message conveys.

Conversely, 8,192 FP16 values occupy 131,072 bits, but their storage size is not their independent or task-relevant information content. Their coordinates can be correlated, redundant, irrelevant, or noisy. Comparing an entire model-state representation with one sampled token compares different objects.

The same qualification applies to CIPHER's expected embedding. For embeddings `(-1, 0, 1)`, the distributions `(0.5, 0, 0.5)` and `(0, 1, 0)` both have expected embedding zero but different entropies. A continuous channel can retain useful uncertainty that a sampled word discards, while still losing other information. The manuscript should avoid describing either representation as automatically preserving exact beliefs.

The VM/webcam/OCR analogy also exaggerates the baseline's costs. Text already travels as compact digital data. Its disadvantages may arise from generating verbose messages or reconstructing task state, rather than from an inherently inefficient physical serialization format. Binary encoding saves transport bytes; it does not automatically reduce the LLM's generation steps. Those improvements must be measured separately.

### 5.2 Correct cache arithmetic, with the right boundary

Under the manuscript's assumptions:

$$B_{\mathrm{KV/token}}=2\times80\times8\times128\times2=327{,}680\ \mathrm{bytes}=320\ \mathrm{KiB}.$$

For exactly 2,000 tokens:

$$B_{\mathrm{trace}}=655{,}360{,}000\ \mathrm{bytes}=655.36\ \mathrm{MB}=625\ \mathrm{MiB}.$$

The manuscript's 640 figure would be **640 MiB for 2,048 tokens**, not 640 MB for 2,000 tokens. Its approximate 80,000× ratio is reasonable against a 4-byte token ID: the exact ratio is 81,920×. A 128,000-entry vocabulary cannot represent every token with a fixed two-byte ID; it needs at least 17 bits, normally three packed bytes or four storage bytes.

However, the calculation describes a **full cache for one particular model layout**. It is not a lower bound for every architecture or protocol. Selected layers, quantization, retained prefixes, and alternative attention designs change it. KIVI, for example, is existing work on reducing cache precision; its relevance here is that raw FP16 is not the only baseline. [R41]

Ideal serialization times for that full 2,000-token cache are:

| Effective link rate assumed | Minimum serialization time |
|---|---:|
| 10 Gbit/s | 524.29 ms |
| 100 Gbit/s | 52.43 ms |
| 400 Gbit/s | 13.11 ms |

These are **calculated lower bounds**, excluding protocol overhead, contention, staging, and latency. They do not imply bandwidth saturation: saturation also depends on transfer frequency and concurrency. Nor do they establish a speedup ceiling relative to a text pipeline that may spend seconds generating messages.

### 5.3 Amdahl's law exposes the missing assumption

For an unchanged workload, let `f` be the fraction of baseline critical-path latency improved, `s` its acceleration, and `o` newly introduced overhead as a fraction of baseline latency:

$$S_{\mathrm{total}}=\frac{1}{(1-f)+f/s+o}.$$

Even with infinite component acceleration and zero overhead, the required removable fractions are:

| Claimed total speedup | Minimum fraction of baseline latency eliminated |
|---|---:|
| 10× | 90.00% |
| 20× | 95.00% |
| 25× | 96.00% |
| 40× | 97.50% |
| 50× | 98.00% |
| 80× | 98.75% |
| 100× | 99.00% |

If inter-agent generation, movement, and receiver prefill consume 80% of a baseline's latency, eliminating them completely gives at most 5× under this decomposition. Accelerating them 50× gives about 4.63× before new overhead.

**Counterargument in the manuscript's favor:** some workloads may indeed spend more than 98% of their time on verbose intermediate messages or repeated processing of huge shared prefixes. Very large improvements are possible there. Amdahl's law does not prohibit them. The missing evidence is that the manuscript's target workload has that structure and that the proposed mechanism removes the relevant work without losing quality.

A system that changes the reasoning algorithm or the number of agent rounds may also change other terms. In that case, demonstrate equivalent task quality and attribute the savings to the changed computation, not solely to networking.

### 5.4 Required latency and throughput accounting

For a single exchange, begin with:

$$T=T_{\mathrm{sender\ reasoning}}+T_{\mathrm{encode}}+T_{\mathrm{queue}}+T_{\mathrm{transport}}+T_{\mathrm{decode/inject}}+T_{\mathrm{receiver\ work}}+T_{\mathrm{verification}}.$$

For a parallel system, measure the actual dependency graph and critical path; adding all agent times would overcount overlap. Report GPU-seconds and energy separately from elapsed latency.

At minimum, distinguish:

- Time to first token, complete-task latency, and inter-message latency.
- Requests completed per second under a stated latency/quality objective.
- Application payload, actual link bytes, HBM reads/writes, and stored state.
- Warm cache and cold cache behavior, including dictionary/model initialization.
- Completed correct tasks per resource budget, including failures and retries.

Memory savings can raise batch size and improve throughput without proportionally improving an individual task's latency. The manuscript combines these effects too freely.

Avoiding vocabulary sampling also does not, by itself, remove receiver prefill. Prefill disappears or changes when the receiver can reuse suitable state or consume a different representation. An already available text message can be processed in a parallel prefill computation; it does not require one serial decoding pass per input token. Sender generation and receiver prefill therefore need separate timing and complexity models.

### 5.5 Repair the rate-distortion claim

Plotting accuracy against `kilobytes × milliseconds` is a possible engineering score, but it is not Shannon's rate-distortion function and is not equivalent to a multidimensional Pareto frontier. For example, 1,000 bytes at 100 ms and 10,000 bytes at 10 ms have the same product despite different deployment consequences.

Define a source distribution `X`, shared side information `C`, a coded message `Z`, and a task-relevant distortion measure. Under a formulation where the side information is available to both encoder and decoder, a conditional rate-distortion objective has the form:

$$R(D\mid C)=\inf I(X;Z\mid C),\qquad \mathbb{E}[d(X,\hat X(Z,C))]\le D.$$

The manuscript does not identify these quantities or establish an operational coding bound. For a practical paper, it is sufficient to report an **empirical quality–rate–latency tradeoff** without calling it a Shannon result. Keep bytes, latency, accuracy, and resource cost as separate axes or explicit constraints. Include the amortized cost of maintaining the shared codebook.

## 6. Architecture and representation findings

### 6.1 Tier 1: a cache handle is useful, but it is not the cached computation

An 8-byte value can be a handle to an existing cache object. The full metadata and storage can live elsewhere. That is a valid design choice. It does not mean the object is only eight bytes, that it is locally resident for every consumer, or that attention no longer reads it.

Exact prefix reuse requires compatible model weights, adapters, tokenization, position assignment, attention masks, cache layout, and precision conventions. A shared system prompt is useful; different preceding role prompts can make the same later text produce different contextual KV states. Matching semantic meaning is insufficient for exact cache identity.

The manuscript must distinguish **forking a branch from a common prefix** from **merging two independently evolved agent histories**. CoW solves storage sharing for the former. It does not define the semantics of the latter. Two branches may have incompatible positions, masks, or dependencies even when their tensors have identical shapes.

Similarly, immutable old KV states do not naturally become “deltas” in the sense of changed database records. New sequence positions append state. Selecting only some positions is a lossy attention approximation, requiring separate quality validation.

**Repair:** use a compatibility fingerprint and immutable branch object containing context identity, position/mask metadata, cache format, permissions, and lifetime. Count actual page reads and remote traffic. Require an explicit trained merge operation, or a structured message, when histories diverge. Treat CoW as a well-established substrate on which a new agent coordination method must justify its added value.

### 6.2 Attention sinks do not establish a 3–5% semantic budget

The manuscript conflates attention sinks, heavy hitters, novelty, and indispensable reasoning state. A token may attract attention without encoding a useful new proposition; a rarely attended token may contain the exact exception needed later. Sender attention also need not predict what the receiver will require.

StreamingLLM retains initial attention sinks alongside recent tokens and explicitly does not preserve arbitrary discarded long-range information. H2O retains a balance of recent tokens and heavy hitters. Neither establishes the manuscript's universal 90% sink figure or a general 3–5% retention budget. [R26] [R25]

**Repair:** test layer-, head-, and token-specific budgets separately, including recent-token protection and protected exact fields. Measure selection overhead and whether gathering attention statistics disrupts efficient attention kernels. Use held-out downstream task quality, not the sparsity percentage itself, as the acceptance criterion.

### 6.3 The 50M transceiver is not the written dense SAE

The equation implies:

$$W_{\mathrm{enc}}\in\mathbb{R}^{M\times d_A},\qquad M=2^{20}.$$

At `d_A = 8192`:

| Quantity | Calculated size |
|---|---:|
| Encoder matrix | 8,589,934,592 parameters |
| Encoder matrix stored in FP16 | 16 GiB |
| Separate encoder and equally sized decoder | 17,179,869,184 parameters; 32 GiB in FP16 |
| Dense encoder multiply per state | Approximately 17.18 billion FLOPs, counting a multiply-add as two |

Top-k sparsity is imposed **after** the dense projection. It does not make that projection cheap. With an untied matrix and no reuse, even reading 16 GiB at an assumed 3 TB/s would take approximately 5.7 ms in an idealized bandwidth-only calculation. Batching and weight residency alter this; the calculation is a conditional bound, not a measured GPU result.

A dense encoder with 50M weights and a million outputs could accept at most about 47 input dimensions before bias. A low-rank factorization could fit approximately rank 47 for the encoder alone, or about rank 23 for separately factorized encoder and decoder before other components. That is a substantially different model requiring quality tests.

If the dictionary is fixed and excluded from *trainable* parameter counts, disclose its **total resident storage and compute**. Calling only the adapter trainable does not remove the dictionary from the implementation budget.

**Repair options:** smaller task-specific vocabularies; hierarchical code selection; factorized or product codebooks; candidate routing followed by exact local scoring; or a compact learned message representation without a claim that every coordinate is a universal named concept. Compare equal total memory, training cost, and quality.

### 6.4 A common anchor corpus does not uniquely define a common language

Two independently trained sparse autoencoders can reconstruct their own model's states while assigning different meanings to the same feature index. Permuting dictionary coordinates and compensating in the decoder can preserve reconstruction loss. Cycle consistency also admits a wrong but reversible relabeling.

If the dictionary and canonical targets are fixed centrally, this symmetry can be constrained. But the manuscript then needs to explain who constructs those targets, what they mean, how every model maps to them, and which assumptions ensure alignment. A corpus alone is not that specification.

The anchor budget is also unsubstantiated. Under the restrictive assumption of one 64-active-feature training state per sample, 100,000 samples provide at most 6.4 million feature activations, averaging only about 6.1 activations per dictionary feature. Multiple states per document can change this calculation substantially; it is not an impossibility proof. It illustrates why corpus size, state sampling, coverage, and feature frequency must be reported.

The strongest available support is for **partial transferable structure**, not a universal ABI. In vec2vec's Table 7, the particular 100,000-source-example experiment has 0.85 top-1 retrieval, illustrating that favorable maxima should not be treated as guarantees. More fundamentally, retrieving the correct document does not prove faithful causal use of a transplanted state. [R08]

**Repair:** provide a canonical semantic schema or fixed teacher representation, independently train each adapter against it, and evaluate held-out model pairings, new concepts, and model upgrades. Measure task fidelity rather than only geometric agreement.

### 6.5 The frozen receiver still needs a defined way to use the message

The manuscript provides an encoder expression, but not a complete receiver computation. Questions left unanswered include:

- Which sender positions are pooled into the proposed state, and how is a variable-size reasoning state reduced to it?
- Is the receiver output one residual vector, several memory positions, or layer-specific KV tensors?
- How are norms, positions, causal masks, and pre-existing receiver context handled?
- What training target makes relation types and uncertainty change the receiver's behavior appropriately?
- How are the graph, literals, provenance, and uncertainty blocks generated, aligned, and decoded?

Injecting a vector at a middle layer does not by itself specify coherent lower-layer state for subsequent decoding. An external memory or adapter can make this well defined, but must be described.

Freezing the backbone is **not inherently incompatible with useful adaptation**. Prefix-tuning demonstrates a relevant precedent for training continuous conditioning while keeping base parameters fixed. The open problem is whether a particular message-conditioned adapter achieves sufficient cross-model fidelity. [R40]

The assertion that frozen attention heads will “learn” newly reserved provenance channels needs correction: either train an added mechanism, modify appropriate parameters, or demonstrate an existing behavior. A local transceiver also requires access to internal activations. Named proprietary services would require provider participation or suitable internal hooks; ordinary application interoperability does not imply that access.

### 6.6 One encoding pass is not one reasoning step

An encoder may output multiple message fields in parallel once the required information exists. That does not show that the sender can obtain a difficult proof, multi-step plan, or diagnostic result in one pass.

The proposal must distinguish:

1. The computation that discovers the relevant information.
2. The computation that compresses it.
3. Transport and interpretation of the compressed result.

There is no established reduction from arbitrary multi-step reasoning to constant-depth computation in this manuscript. Even “O(1)” needs a variable: fixed-size message generation can be constant in the number of *serialized output fields*, while still depending on model width, dictionary size, context length, and sequential layers.

Likewise, sentence-level LCM processing reduces sequence length but does not imply one cheap operation per proposition. The source's treatment includes decoding and iterative diffusion costs. A quadratic attention-size reduction describes one term, not total model computation. [R01]

### 6.7 The frame needs a real serialization specification

The listed fields sum correctly:

$$16+320+512+256+396=1500\ \mathrm{bytes}.$$

The networking conclusion does not. With an IP MTU of 1,500, ordinary UDP permits at most **1,472 application bytes over IPv4** with a minimal IP header, or **1,452 over IPv6**, before additional security or encapsulation. Raw Ethernet could carry a custom 1,500-byte payload on a local segment, but that is not the proposed interoperable WAN transport. RFC 8085 explicitly requires subtracting IP and UDP headers and accounting for path MTU. [R29]

The 64×64 graph has 4,096 bits, exactly 512 bytes. This supplies one binary edge flag per ordered pair. For four possible exclusive types plus “absent,” a straightforward three-bit-per-pair representation requires 1,536 bytes. Four independent binary relation matrices require 2,048 bytes. A sparse typed edge list can be much smaller, but it needs an explicit edge-count limit and overflow behavior.

There are further semantic and operational gaps:

- Feature IDs are not entity-instance IDs. Two occurrences of the same concept, nested negation, variable binding, quantifier scope, time, and n-ary relations need a representation.
- A 256-byte FP16 covariance block can store a 64×2 factor, if that is its defined format. It cannot represent an unrestricted 64×64 covariance; a symmetric FP16 matrix alone needs 4,160 bytes. Factor orientation, residual diagonal, PSD constraints, and interpretation are unspecified.
- The 16-byte header omits several later requirements, including a defined dependency pointer and dictionary/adapter identity. It provides no complete integrity, replay, ordering, or authentication design.
- A fixed 396-byte literal slot has no defined overflow, chunking, or external-object mechanism. Exact mathematics and code frequently require more space.

**Repair:** write a versioned, testable schema with capability negotiation, explicit bounds, typed entity instances, typed edges, literal lengths, and extension behavior. Aim for small common-case messages; permit multi-packet or referenced objects. A one-packet target is an optimization, not a semantic completeness theorem.

## 7. Uncertainty, safety, and distributed execution

### 7.1 Predictive entropy is not exact epistemic confidence

A model can confidently repeat a false statement. It can also be uncertain about wording while being certain about the underlying fact. Entropy over next-token predictions therefore does not identify the probability that an incoming proposition is correct. Epistemic uncertainty, ambiguity in the input, diversity of valid expressions, and sampling temperature are not interchangeable.

Storing a value in FP16 gives it a numeric representation; it does not calibrate it. Existing calibration research supplies methods and diagnostics, not a warrant that one scalar remains calibrated across tasks or model updates. [R28]

**Repair:** define confidence for a particular claim or decision and evaluate it against labeled outcomes on held-out data. Report Brier score or log loss, calibration plots, and accuracy versus abstention coverage. Record whether evidence comes from a tool observation, checked proof, model estimate, or another agent. Do not average correlated peer beliefs as if they were independent evidence.

### 7.2 Norm bounds are useful input checks, not truth detectors

Text is not a general security firewall: syntactically valid language can convey false premises or malicious instructions. Conversely, an unusual activation is not necessarily a hallucination. The phrase “instantaneous correlated psychosis” should be replaced with a precise hypothesis about correlated errors under untrusted state injection.

Mahalanobis screening requires a suitable distributional model and covariance estimate. If the covariance is taken over the full million-feature dictionary, it contains about `2^40` entries, requiring **4 TiB in FP32**. With only 100,000 independent vector samples, an ordinary sample covariance has rank at most 99,999 and is singular in that space. Multiple states per sample, regularization, dimensionality reduction, or active-subspace models can change the implementation, but must be specified.

Even a well-estimated in-distribution score cannot distinguish truth from plausible falsehood in general. An attacker can use ordinary-looking values; a novel correct result can be an outlier. A vector has a Euclidean norm; “spectral clipping” should identify which matrix or operator is being constrained.

**Repair:** treat norm, shape, NaN/Inf, and range checks as input validation. Add calibrated anomaly detection only for a defined operating distribution. Evaluate false positives on legitimate rare inputs and false negatives under adaptive attacks. Keep important tool actions behind independent authorization and validation.

### 7.3 Provenance needs evidence and authentication outside the hidden state

An orthogonal transform preserves geometry under a common rotation. It does not prove who sent a message, whether it was altered, or whether a claim is true. If different agents apply different rotations without a learned or explicit inverse, ordinary receiver attention can change unpredictably.

A 32-dimensional semantic-free subspace is also not guaranteed to exist in a frozen model. Reserve metadata in an external schema; do not assume all useful neural computation naturally avoids selected coordinates.

**Repair:** use authenticated messages or capabilities with origin identity, evidence/artifact hashes, verification method and version, timestamps/epochs, and dependency links. Separate “observed tool result” from “true proposition”: a tool can be wrong, stale, or misinterpreted. Deduplicate evidence by its origin and dependency graph, not the number of agents repeating it.

A trained attention bias may help the receiver use provenance, but it is an empirical modeling feature. Authentication and policy enforcement should not depend on whether the model pays attention to that feature.

### 7.4 Speculation requires bounded recovery, not instantaneous reversal

The transactional analogy is productive. The one-cycle global guarantee is not. At an illustrative 1 GHz, one cycle is 1 ns; even signal propagation over one meter of fiber is roughly 5 ns before switching or processing. A WAN abort necessarily takes longer still.

A controller may invalidate a branch with an epoch change or pointer swap in constant local work. That does not immediately notify remote agents, cancel in-flight kernels, find all dependencies, reclaim memory, or recompute valid results. Nor can it undo a tool call already committed outside the system.

A single parent pointer is insufficient for a computation depending on multiple incoming premises unless it references an external dependency set. Reordering, duplicate aborts, lost messages, process restarts, and competing commit decisions also require semantics. At one million transactions per second, an unqualified 32-bit counter wraps in about **71.6 minutes**; namespace and epoch rules matter.

**Repair:** immutable branch IDs, origin-plus-epoch transaction identifiers, explicit dependency sets, reliable idempotent aborts, and lazy invalidation by epochs. Use commit barriers for external effects, or document compensating actions where true rollback is impossible. Measure notification delay, invalidation completion, wasted compute, replay cost, and recovery latency separately.

### 7.5 Small geometric error does not bound logical error

Consider two states:

$$u=(1,0.01),\qquad v=(1,-0.01).$$

Their cosine similarity is approximately **0.99980002**. If the sign of the second component encodes a truth-relevant distinction, their interpretations are opposite. Reconstructing `u` as `(1,0)` gives normalized squared error approximately **0.00009999**, while losing that distinction entirely.

This is a mathematical counterexample, not a claim about a particular model's coordinates. It establishes that the proposed cosine and residual thresholds alone cannot guarantee entailment preservation. Independently, the cited SAE study warns that interpretable features do not always yield precise steering effects. [R15]

A perfect round trip also does not establish intermediate understanding. An encoder/decoder pair can consistently swap two concept labels and undo the swap on return. The cycle succeeds while an intermediate receiver interprets the message incorrectly.

**Repair:** test entity identity, argument order, negation scope, units, inequalities, quantifiers, and exact literals directly. Use task-specific semantic loss and executable checks. Combine residual monitoring with uncertainty calibration and mandatory exact encoding for designated fields. A model that never learned a rare distinction may reconstruct its own inadequate state accurately; low residual cannot detect that missing knowledge.

### 7.6 Sparse concepts do not eliminate covert communication

Choosing 64 distinct feature IDs from a dictionary of `2^20` permits:

$$\log_2 {2^{20}\choose64}\approx984\ \text{bits}$$

of raw selection capacity before magnitudes or graph structure. Sixty-four FP16 magnitudes occupy another **1,024 encoding bits**. The 396-byte literal field holds **3,168 bits**. Not all raw patterns are valid, meaningful, or covertly usable; these are representation-capacity calculations, not a measured steganographic rate. They nevertheless refute the argument that sparsity alone eliminates such a channel.

Quantizing a continuous channel reduces its raw precision, which can help constrain leakage. The remaining capacity must still be measured under the protocol's semantic and monitoring constraints; it cannot be inferred to be negligible from the proportion of zero coordinates.

Even modest choices among semantically equivalent messages, tiny magnitude changes, message ordering, or timing may carry hidden information. An interpretable alphabet can support a hidden code. The manuscript's own final confidence table acknowledges combination-based steganography, contradicting its earlier guarantee.

**Repair:** define sender capabilities, allowed information flow, monitor visibility, and tolerated leakage. Canonicalize ordering and serialization, restrict optional fields and literals, reject unused numeric encodings, quantize only as aggressively as quality permits, and test an adaptive adversary. Report estimated leakage or detection performance under that threat model. “100% human auditability” should be removed unless narrowly defined and demonstrated.

### 7.7 Hardware promises are premature

Existing CXL systems provide useful evidence, but their results depend on topology, access size, locality, and baseline. TraCT, for example, reports very different improvements for TTFT and peak throughput, demonstrating why those metrics must remain separate. CXL memory sharing also has explicit synchronization and consistency requirements. [R32] [R33] [R34]

The manuscript's 600 ns figure is not a general guarantee. HyMCache distinguishes different memory tiers and, in one evaluation, limits decode to one token to isolate reuse effects. That is a valid experiment for its purpose, but not evidence for arbitrary reasoning workflows. [R07]

The proposed optical HBM pool, PIM transceiver, tensor-MMU, and symbolic tiles need separate feasibility studies:

- End-to-end access latency, sustainable bandwidth, contention, and coherence traffic.
- Capacity, packaging, power, thermal constraints, and failure isolation.
- Cost of dense dictionary scoring, top-k selection, and irregular accesses.
- Supported formal language and bounded verification work for symbolic tiles.

Offloading work can save main-core cycles while still consuming elapsed time, memory bandwidth, energy, and hardware area. General logical verification cannot be assumed to complete at wire speed for arbitrary inputs. Begin with software profiling and a roofline/queueing model; consider dedicated hardware only after the codec and dominant workload are stable.

## 8. Alternative designs serving the same goal

These are proposed implementation paths, not measured replacements. Their purpose is to preserve the manuscript's goal—faster useful collaboration—while giving each claimed benefit a testable mechanism.

### 8.1 Start with a deployment-aware communication policy

| Situation | Preferred initial mechanism | Expected source of savings | Principal limitation |
|---|---|---|---|
| Identical model state and exact shared prefix in one memory domain | Immutable shared KV prefix plus private branches | Avoid duplicate storage, copying, and prefill | Continued attention and reasoning still cost time |
| Related models with verified cache compatibility | Selective or projected KV reuse | Reduce repeat processing | Quality and compatibility need explicit checks |
| Different model families or black-box services | Typed compact messages and artifact references | Reduce verbosity, repeated context, and unnecessary turns | Models may still need text serialization at their interface |
| Open models with trained adapters | Optional task-conditioned latent messages | Reduce intermediate generation and receiver processing | Training, distribution shift, and auditing costs |
| Large evidence objects or exact code/math | Content-addressed objects plus selected exact fields | Avoid resending identical content | Cache misses and retrieval latency remain |

The policy should select among measured paths based on compatibility, message size, quality requirements, and cache state. A latent route should have an explicit fallback when it cannot establish compatibility or confidence.

### 8.2 Alternative A: exact cache sharing before lossy state compression

Implement a branch service over an existing paged inference engine:

1. Compute a compatibility fingerprint covering weights, adapters, tokenizer, position scheme, mask rules, and cache format.
2. Identify exact shared prefixes and retain their KV blocks as immutable objects.
3. Fork branches using handles and reference counts; append private state without copying the full prefix.
4. Keep divergent histories distinct. Exchange a structured result when another agent needs the conclusion rather than the entire branch.
5. Measure both resident bytes and actual memory/link traffic, including reads of shared blocks.

Use existing PagedAttention/CoW behavior as the baseline. Evaluate ForkKV only where its multi-LoRA setting matches the experiment. Add cache quantization or retention policies one at a time, with a quality constraint. [R21] [R22] [R23] [R41]

**Why this addresses the paper's goal:** it removes redundant work with a clear exactness condition. It can establish the value of locality before introducing a new semantic code.

**Failure cases:** incompatible adapter changes, low prefix overlap, HBM pressure, many remote readers, or latency dominated by downstream tools. The design must report these cases, not discard them from the speedup average.

### 8.3 Alternative B: compact typed messages with references

Replace unrestricted agent monologues with a small application schema. For a software-debugging workflow, a useful message might contain:

```json
{
  "schema": "diagnostic-result/1",
  "task_id": "T42",
  "claim_id": "C17",
  "subject": {"artifact": "sha256:...", "symbol": "parse_header"},
  "relation": "fails_on",
  "object": {"test_case": "case-19"},
  "evidence": [{"artifact": "sha256:...", "kind": "test-log"}],
  "status": "observed",
  "depends_on": ["C3"],
  "literal_payload": "exact failing input, or a reference to it"
}
```

The JSON is for readability. A deterministic CBOR or similar binary codec can carry the same structure. CBOR is an existing standard, so this does not require inventing an application-level binary format from scratch. [R30]

Separate the effects of **shorter generated content**, **binary serialization**, and **reference reuse**. A black-box model may still need to generate or consume JSON/text; encoding that output as CBOR saves network bytes but does not retroactively save its generation tokens. Where tools already produce structured data, pass that data without asking an LLM to narrate it.

Require exact entity identifiers, relation types, units, scope where needed, and explicit evidence links. A typed schema supports arbitrary labels and new objects without assuming that every domain concept has already been learned by a universal SAE.

**Why this addresses the goal:** it directly tests whether the expensive part is verbose conversation, duplicated evidence, or the absence of shared application state.

**Failure cases:** tasks requiring rich context that the schema omits, excessive schema overhead for tiny messages, and frequent reference-cache misses. Permit bounded free text and object retrieval rather than silently discarding information.

### 8.4 Alternative C: learned messages with frozen bases and local adapters

After the structured baseline is established, test an optional latent codec for models whose internals are accessible.

Begin with a narrow domain and a compact code—perhaps 8–32 learned memory positions or a modest vector-quantized codebook. The numbers are experimental settings to sweep, not asserted optimal values. Keep exact identifiers, proof terms, and literals in a symbolic side channel.

**Training design:**

1. Define a fixed canonical teacher representation or typed task schema. Publish its version and training/validation split.
2. Train each sender adapter to produce a canonical message from selected states and task context. Train each receiver adapter to use that canonical message through a defined memory interface or prefix mechanism.
3. Freeze base weights if modularity is required. Allow gradients through the frozen computation to train the adapters; distinguish this from updating the base itself.
4. Include task loss and contrastive cases differing only in argument order, negation, or exact values. Reconstruction loss alone is insufficient.
5. Test sender and receiver adapters first in independently trained combinations, including model versions and pairings held out from training.

A candidate objective is:

$$\mathcal L=\mathcal L_{\mathrm{task}}+\lambda_a\mathcal L_{\mathrm{canonical}}+\lambda_r\mathcal L_{\mathrm{rate}}+\lambda_c\mathcal L_{\mathrm{calibration}}.$$

Every term needs an operational definition. `L_task` should measure downstream correctness; `L_canonical` anchors the common interface; `L_rate` charges actual coded bytes or message positions; calibration is fitted and evaluated on separate data. Adapter capacity, receiver compute, and teacher-generation costs must be reported.

Prefix-tuning is evidence that frozen bases can use learned continuous conditioning. It does not prove this cross-model codec will work. That must be established experimentally. [R40]

**Potential benefit:** fewer autoregressive intermediate tokens and smaller receiver inputs. **Principal risks:** poor transfer to unfamiliar receiver models, missing tail concepts, low-fidelity bindings, training-data leakage, and codec overhead exceeding the saved work.

### 8.5 Alternative D: reduce communication demand and accelerate remaining generation

Some of the largest savings may come from changing the workflow:

- Share immutable evidence once and transmit only changed claims or artifact references.
- Let a receiver request missing detail instead of receiving every intermediate state.
- Stop redundant debate when agents cite the same evidence and add no independent information.
- Route to a specialist only when expected quality improvement justifies added latency.
- Batch independent tasks and keep frequently interacting agents near their state.

These policies need quality-controlled evaluation: reducing messages can merely reduce useful deliberation. A strong single-agent or tool-driven baseline is mandatory.

For remaining autoregressive generation, exact speculative decoding is a separate acceleration route with established distribution-preserving constructions under its algorithmic assumptions. Its benefit depends on draft acceptance and hardware workload; it is not guaranteed to multiply independently with communication savings. [R39]

### 8.6 A more credible wire and provenance design

Use a negotiated maximum message size and a small-message target instead of promising all reasoning fits one Ethernet packet. An illustrative **1,200-byte application-message budget** could be:

| Component | Example allocation |
|---|---:|
| Version, capabilities, IDs, counts, and offsets | 64 bytes |
| Up to 32 node instances, 6 bytes each | 192 bytes |
| Up to 64 typed edges, 4 bytes each | 256 bytes |
| Claim-level confidence/verification fields | 64 bytes |
| Evidence and dependency references | 128 bytes |
| Exact literal bytes | 320 bytes |
| Extension/reserved space | 176 bytes |
| **Total** | **1,200 bytes** |

This is a **budget sketch**, not a validated codec or a guarantee of unfragmented delivery. Authentication and transport overhead are additional; actual sizing follows path-MTU negotiation. Reference lengths and counts impose explicit bounds. Larger objects use a reliable stream or referenced artifact, and overflow is visible to the receiver.

Unlike the manuscript's fixed universal frame, the design makes its tradeoff explicit: a compact common case with a defined large-message path. It can be tested before any learned encoder exists.

## 9. Falsifiable evaluation plan

### 9.1 State the hypotheses before running benchmarks

| Hypothesis | Experiment that could support it | Evidence that would count against it |
|---|---|---|
| H1: explicit branch sharing adds value beyond an optimized existing engine | End-to-end comparison at fixed quality and resources against existing prefix caching/CoW | Gains disappear once the baseline uses the same sharing and batching |
| H2: a compact codec conveys useful information beyond short structured messages | Compare messages with matched information access and quality constraints | Equal or better performance from compact text/schema at lower total cost |
| H3: independently trained adapters interoperate | Train adapters separately; test unseen model-pair combinations without pair tuning | Pair-specific tuning is necessary or role/negation errors increase |
| H4: the literal/fallback route protects exactness | Stress exact identifiers, units, code, proofs, and out-of-schema content | Undetected semantic errors despite low residual or high confidence |
| H5: speculation lowers completed-task latency | Fault-injected asynchronous workloads with recovery costs included | Aborts and replay erase gains, or stale results escape commit controls |
| H6: a restricted channel lowers covert leakage | Adaptive adversary under fixed task utility and monitor visibility | Substantial recoverable hidden information remains within valid messages |

“Success on the first packet” is not a statistical criterion. Replace it with a held-out independent-adapter test set. A hot swap should first preserve the interface contract; a 10% reasoning improvement is a separate hypothesis.

### 9.2 Baselines

Use at least:

1. A strong single agent with appropriate tools and the same relevant information.
2. The original natural-language multi-agent workflow.
3. Concise text/JSON with enforced length and task-specific schema.
4. Structured binary messages plus artifact references and cached shared context.
5. Existing prefix caching and CoW for compatible local models.
6. Relevant selective/projected KV or latent methods, using their stated supported configurations.
7. A centrally informed model, where feasible, as an information-access comparator—not a guaranteed quality upper bound.

If TTCIP adapters receive task-specific supervision, include a text baseline with comparable adaptation access or report that asymmetry prominently. Match model versions, tools, datasets, reasoning budgets, concurrency, hardware, and cache state. Report both quality-matched and resource-matched comparisons; equal token counts do not necessarily imply equal compute for different representations.

### 9.3 Workloads

Use a combination of controlled semantic tasks and realistic workflows:

- **Binding and precision:** reversed arguments, nested negation, repeated entity types, quantifiers, units, inequalities, exact strings, and rare identifiers.
- **Distributed information:** each agent initially has different evidence; vary overlap and the need to combine facts.
- **Software engineering:** issue resolution with hidden tests, reproducible repositories, and fixed tool access. Passing SWE-bench tests is useful empirical evidence, not a formal proof of program correctness.
- **Formal reasoning:** proof-checker-verified results in a specified language, including valid novel definitions outside the training schema.
- **Serving:** small/large contexts, 0–100% prefix overlap, short/long outputs, concurrency sweeps, and cold/warm caches.
- **Networks and failures:** bandwidth/RTT sweeps, message loss and duplication, reordering, process failure, stale epochs, and revoked dependencies.

Split by underlying problem, project, concept family, and model pairing where appropriate. Keep public anchor data, validation data, and final tests disjoint. Include entity renaming and synthetic held-out compositions to reduce reliance on memorized benchmark answers.

### 9.4 Metrics and statistics

Pre-register one primary quality metric and one primary performance metric per workload. Suggested reporting includes:

| Dimension | Measures |
|---|---|
| Task quality | Accuracy/pass rate, verified correctness, and exact-field error rate |
| Latency | Complete-task p50/p95/p99, TTFT separately, and failure/retry latency |
| Throughput | Correct completed tasks/s at stated latency and resource limits |
| Resource use | GPU-seconds, peak/resident memory, link bytes, memory traffic, and energy where measurable |
| Codec | Encoding/decoding time, actual bytes, fallback rate, and training/amortization cost |
| Uncertainty | Brier/log loss, calibration curves, selective error versus coverage |
| Reliability | Abort propagation, stale-result escape rate, replay and wasted compute |
| Security | Attack success, recoverable covert information, and monitor false positives |

Use paired task comparisons and confidence intervals that respect the sampling unit. Multiple prompts or seeds for the same underlying problem are not independent new problems. A cluster bootstrap by task/project can be appropriate; choose and justify the procedure before inspecting final results. Report training-seed variability for learned codecs and all materially different model pairings, not only the best pairing.

Run a pilot to estimate variance and choose sample size through a power calculation. For illustration, observing zero errors in 2,995 independent trials only supports a one-sided 95% binomial upper bound of roughly 0.1% on error probability. It does not establish 100% correctness, and the bound is inappropriate if trials are strongly dependent or selected after inspection.

Absolute and relative accuracy changes must be separated: 70% to 77% is **7 percentage points**, or **10% relative improvement**. Accuracy is bounded by 100%; “orders-of-magnitude accuracy improvement” is generally inappropriate without a carefully defined error metric.

### 9.5 Ablations that identify the mechanism

Start from the same optimized baseline and add one change at a time:

1. Shortened structured content.
2. Reference-based evidence reuse.
3. Exact cache sharing.
4. Cache quantization or selective retention.
5. Learned latent encoding/decoding.
6. Typed relationships and exact literals.
7. Confidence/provenance mechanisms.
8. Speculative overlap and rollback.

Include shuffled, zeroed, wrong-sender, and random-code controls. If a receiver performs equally well after the latent message is destroyed, the claimed semantic communication may not be responsible for its result. Likewise, a receiver that answers from benchmark memorization can pass an apparent communication test without using the sender's information.

### 9.6 Replace the proposed causal and cycle tests

The manuscript's bit-flip audit is valuable as an intervention idea but imposes the wrong guarantee. Changing a genuine causal premise may legitimately change several downstream branches. Conversely, no change may mean the model ignored the message.

Define the intended causal consequences first. Use matched interventions on entity identity, relation type, and scope; include no-op edits and unrelated-feature controls. Measure both the desired changes and unintended collateral effects. Keep stochastic generation controlled or compare distributions across repeated trials.

For multi-hop tests, measure exact symbolic invariants and downstream task outcomes at every hop, not just cosine similarity after the final round trip. Report error accumulation as hop count increases. Human or LLM entailment judgments alone are insufficient for a 100% logical-equivalence claim; use a formal checker where the language supports one.

### 9.7 Staged implementation and decision gates

| Stage | Deliverable | Decision gate |
|---|---|---|
| 0. Repair the proposal | Consistent terminology, source table, byte/parameter budget, and workload model | No numerical contradictions; hypotheses clearly labeled |
| 1. Profile the baseline | Reproducible task traces and critical-path breakdown | Claimed removable work is large enough to justify the target speedup |
| 2. Build exact/software alternatives | Cache-branch and structured-message prototypes | Demonstrable benefit over optimized baselines at matched quality |
| 3. Test a bounded latent codec | Independently trained adapters, full training details, and exact-field channel | Held-out pairings meet pre-registered quality and cost criteria |
| 4. Stress reliability/security | Fault injection, adaptive attacks, and recovery measurements | No unaccounted correctness/authorization failure in the declared scope; residual risks quantified |
| 5. Consider hardware | Profiling-driven model, prototype or simulation with validated assumptions | A stable codec bottleneck remains and justifies area/power/cost |

A reasonable initial engineering gate might be a pre-registered quality loss margin of at most **one percentage point** and at least **20% complete-task latency reduction** on a chosen workload. These are illustrative thresholds requiring application-specific justification, not universal scientific standards.

If the manuscript retains a claim of at least 25× acceleration, the experiment must support **that specific magnitude**, with uncertainty and scope stated. A smaller useful improvement should be published honestly rather than judged a failure merely because the original numerical aspiration was too large.

### 9.8 What would change this review's conclusion?

The strongest counterevidence would be a reproducible implementation showing:

- A complete transceiver architecture within its declared total parameter, storage, and compute budget.
- Typed, scoped semantics and exact values round-tripping correctly under held-out composition tests.
- Independently trained adapters working on unseen model-pair combinations without hidden pair-specific adaptation.
- Quality-controlled end-to-end gains over concise structured messages and optimized cache-sharing baselines, including all codec and recovery costs.
- Calibrated uncertainty and a narrowly specified, experimentally supported leakage/safety claim.

Such results would materially improve the assessment. They would not automatically validate the separate universal security or one-cycle distributed rollback claims.

## 10. Required manuscript revisions

### 10.1 Revisions needed before scientific submission

1. Recast the abstract and conclusion as a proposal unless experimental results are added. Label the 25–100× figures as targets, or remove them.
2. Replace the claimed universal speed ceiling with a measured or conditional critical-path model.
3. Correct source comparisons, especially DiffMAS training, HyLaT baselines, Kouwenhoven's chance level, and LCM/BLT cost metrics.
4. Specify the encoder, decoder, graph-generation mechanism, receiver integration, and training objectives completely.
5. Resolve the transceiver parameter budget and distinguish total, resident, and trainable parameters.
6. Publish a complete wire schema with typed relations, entity instances, scope, overflow, versioning, authentication, and transport accounting.
7. Define compatibility for cache sharing and state the semantics of branch merging.
8. Replace exact epistemic confidence, perfect auditability, and one-cycle global recovery with testable bounded claims.
9. Add relevant prior work and a clear novelty statement relative to existing CoW and compressed-cache systems.
10. Supply code, configurations, model/version identifiers, seeds, data splits, traces, and results adequate for reproduction.

### 10.2 Presentation corrections

- Use “literature review” instead of “meta-analysis” unless a systematic synthesis is actually performed.
- Use KiB/MiB for binary quantities and distinguish MB/s from Gbit/s.
- State whether accuracy differences are percentage points or relative percentages.
- Replace “fatal fleet lock-in” with the particular compatibility and retraining costs being asserted. Versioned adapters and hybrid channels are counterexamples to inevitability.
- Replace “syntactic firewall” and “correlated psychosis” with defined mechanisms and measurable outcomes.
- Do not equate human-readable feature names with complete human understanding of their joint causal behavior.
- Rename the “Shannon” plot unless it has an appropriate source, distortion measure, and coding formulation.
- Use `||z||_0 <= k` unless the implementation guarantees exactly k nonzero outputs after ReLU; ties and zero-valued selected entries need rules.
- Distinguish CXL, NVLink, shared addressing, and actual cache coherence; they are not interchangeable guarantees.
- The abstract's description of existing accuracy gains as single-digit conflicts with its later cited 14.6% maximum. Resolve the comparison and terminology.

## 11. Final assessment

The manuscript identifies a worthwhile systems problem and proposes a useful separation between local state reuse and heterogeneous communication. Its strongest near-term contribution could be a carefully evaluated communication policy that combines established sharing mechanisms with a new, bounded learned codec.

Its present central claims exceed the evidence. The most serious defects are the missing performance model, the dense transceiver's parameter mismatch, the incomplete wire semantics, and the unsupported inference from geometric similarity to reliable cross-model reasoning. The security and rollback claims require substantial narrowing.

The next scientific step is a **small reproducible software prototype with strong baselines and explicit failure conditions**. A successful result need not reach 50× to be valuable. It must show what was saved, which information was retained, what quality changed, and under which workload the result holds.

## Appendix A. Reproducible arithmetic and counterexamples

The following dependency-free Python program was executed during this review in equivalent form. It reproduces the central calculations and synthetic counterexamples. It does **not** run an LLM or validate a proposed alternative empirically.

```python
import math

M, d, k = 2**20, 8192, 64
kv_per_token = 2 * 80 * 8 * 128 * 2
trace_bytes = 2000 * kv_per_token
encoder_weights = M * d
frame_bytes = 16 + 320 + 512 + 256 + 396

assert kv_per_token == 327_680
assert trace_bytes == 655_360_000
assert trace_bytes / 2**20 == 625
assert encoder_weights == 8_589_934_592
assert encoder_weights * 2 / 2**30 == 16
assert frame_bytes == 1500

print("KV bytes/token:", kv_per_token)
print("2,000-token trace: MB, MiB:", trace_bytes / 1e6, trace_bytes / 2**20)
print("Encoder weights:", encoder_weights)
print("Encoder FP16 GiB:", encoder_weights * 2 / 2**30)
print("Encoder multiply-add FLOPs:", 2 * encoder_weights)
print("Application frame bytes:", frame_bytes)
print("UDP payload limits at MTU 1500:", 1500 - 20 - 8, 1500 - 40 - 8)
print("Typed dense graph, 3 bits/pair:", k * k * 3 // 8)
print("Four binary relation matrices:", k * k * 4 // 8)
print("Symmetric FP16 covariance bytes:", k * (k + 1) // 2 * 2)
print("64x2 FP16 covariance factor bytes:", k * 2 * 2)
print("Full dictionary covariance, FP32 TiB:", M * M * 4 / 2**40)

selection_bits = sum(math.log2(M - i) - math.log2(i + 1) for i in range(k))
print("64-feature subset capacity, bits:", selection_bits)
print("Magnitude encoding bits:", 16 * k)
print("Literal field bits:", 396 * 8)

for target in (10, 20, 25, 40, 50, 80, 100):
    print("Speedup target; minimum removable fraction:", target, 1 - 1 / target)

for gbps in (10, 100, 400):
    print("Link Gbit/s; ideal serialization ms:", gbps, trace_bytes * 8 / (gbps * 1e9) * 1000)

print("HyLaT relative to NL:", 13.15 / 1.47)
print("HyLaT relative to trained text:", 5.47 / 1.47)
print("32-bit counter wrap minutes at 1M/s:", 2**32 / 1e6 / 60)

# High similarity can coexist with reversal of a decisive coordinate.
u, v, reconstruction = (1.0, 0.01), (1.0, -0.01), (1.0, 0.0)
cosine = sum(a * b for a, b in zip(u, v)) / (math.hypot(*u) * math.hypot(*v))
residual = sum((a - b)**2 for a, b in zip(u, reconstruction)) / sum(a*a for a in u)
assert cosine > 0.999
assert residual < 0.0001
assert (u[1] > 0) != (v[1] > 0)
print("Opposite decisive signs; cosine:", cosine)
print("Lost decisive coordinate; residual:", residual)

# Cycle consistency alone cannot establish correct intermediate labels.
def swap_label(label):
    return {"CAUSES": "INHIBITS", "INHIBITS": "CAUSES"}.get(label, label)

assert swap_label(swap_label("CAUSES")) == "CAUSES"
assert swap_label("CAUSES") != "CAUSES"

# Byte-time products can hide materially different tradeoffs.
assert 1000 * 100 == 10000 * 10

# No observed failures does not imply a zero population error rate.
n = math.ceil(math.log(0.05) / math.log(0.999))
upper_95 = 1 - 0.05 ** (1 / n)
print("Independent zero-failure trials; one-sided 95% upper bound:", n, upper_95)
assert n == 2995
assert upper_95 < 0.001
```

## Appendix B. Scope and session planning

The submitted file contains approximately 4,944 whitespace-separated words and 37,474 bytes. The initial planning estimate was 9,000–11,000 tokens for the paper and 45,000–75,000 additional tokens for retrieval, analysis, and the review. These were estimates, not measurements of account quota or an exact remaining context window.

This documentary and analytical review can be completed in one session with compact evidence records. Actual experimental replication should be a separate project: first reproduce baseline behavior, then test a codec, then evaluate systems and adversarial properties. Splitting that empirical work is justified by dependencies and access to models/hardware, rather than by the length of this manuscript alone.

## References and evidence links

Primary sources accessed for this review on 19 October 2026. Versions identify the text inspected; some sources have multiple posting and publication dates. Bibliographic verification establishes the identity of a work, not independent reproduction of its results.

### Manuscript bibliography

- **R01.** LCM team / Barrault et al. *Large Concept Models: Language Modeling in a Sentence Representation Space*. 2024, arXiv:2412.08821v2. See particularly §§2.5.1–2.5.2. [Primary text][R01]
- **R02.** Kallini et al. *Fast Byte Latent Transformer*. 2026, arXiv:2605.08044v1. [Primary text][R02]
- **R03.** Foerster et al. *Learning to Communicate with Deep Multi-Agent Reinforcement Learning*. 2016, arXiv:1605.06676. [Primary record][R03]
- **R04.** Fu et al. *Cache-to-Cache: Direct Semantic Communication Between Large Language Models*. arXiv:2510.03215v2. [Primary text][R04]
- **R05.** Hao et al. *Training Large Language Models to Reason in a Continuous Latent Space*. arXiv:2412.06769v4. [Primary record][R05]
- **R06.** Huh et al. *The Platonic Representation Hypothesis*. 2024, arXiv:2405.07987v5. [Primary record][R06]
- **R07.** Jang et al. *HyMCache: A KV Cache Framework for Multi-Turn LLM Serving with CXL-Hybrid Memory*. arXiv:2607.18141v1, title as rendered in the primary HTML. See §§2.2, 4.1, 5. [Primary text][R07]
- **R08.** Jha et al. *Harnessing the Universal Geometry of Embeddings*. arXiv:2505.12540v4. See Tables 2 and 7. [Primary text][R08]
- **R09.** Kouwenhoven, Peeperkorn, and Verhoef. *Searching for Structure: Investigating Emergent Communication with Large Language Models*. 2024, arXiv:2412.07646v3. See §5.2. [Primary text][R09]
- **R10.** Lazaridou and Baroni. *Emergent Multi-Agent Communication in the Deep Learning Era*. 2020, arXiv:2006.02419. [Primary record][R10]
- **R11.** Lewis et al. *Deal or No Deal? End-to-End Learning for Negotiation Dialogues*. 2017, arXiv:1706.05125. [Primary paper][R11]
- **R12.** Liu et al. *DroidSpeak: KV Cache Sharing for Cross-LLM Communication and Multi-LLM Serving*. arXiv:2411.02820v4. [Primary text][R12]
- **R13.** Motwani et al. *Secret Collusion among AI Agents: Multi-Agent Deception via Steganography*. arXiv:2402.07510v5. See mitigation discussion and paraphrasing analysis. [Primary text][R13]
- **R14.** Mou et al. *HyLaT: Efficient Multi-Agent Communication via Hybrid Latent-Text Protocol*. 2026, arXiv:2605.25421v1. See Table 1, §4.3, Appendix B.1. [Primary text][R14]
- **R15.** Pach et al. *Sparse Autoencoders Learn Monosemantic Features in Vision-Language Models*. arXiv:2504.02821v3. See the Limitations paragraph. [Primary text][R15]
- **R16.** Pham et al. *Let Models Speak Ciphers: Multiagent Debate through Embeddings*. arXiv:2310.06272v2. [Primary record][R16]
- **R17.** Shi et al. *KVComm: Enabling Efficient LLM Communication through Selective KV Sharing*. arXiv:2510.03346v3. See problem scope and selective-layer evaluation. [Primary text][R17]
- **R18.** Wu et al. *DualPath: Breaking the Storage Bandwidth Bottleneck in Agentic LLM Inference*. 2026, arXiv:2602.21548v2. [Primary text][R18]
- **R19.** Yu et al. *Learning to Communicate: Toward End-to-End Optimization of Multi-Agent Language Systems*. 2026, arXiv:2604.21794v1. See Figure 1 and Appendix C for frozen-backbone/LoRA training. [Primary text][R19]
- **R20.** Zou et al. *Latent Collaboration in Multi-Agent Systems*. arXiv:2511.20639v4. See headline results and heterogeneous-agent discussion. [Primary text][R20]

### Additional primary sources

- **R21.** Kwon et al. *Efficient Memory Management for Large Language Model Serving with PagedAttention*. 2023, arXiv:2309.06180. [Primary record][R21]
- **R22.** vLLM project. Official PagedAttention introduction, 20 June 2023; explicitly describes shared blocks, reference counting, and CoW. [Project explanation][R22]
- **R23.** Wang, Ren, and Gui. *ForkKV: Scaling Multi-LoRA Agent Serving via Copy-on-Write Disaggregated KV Cache*. 2026, arXiv:2604.06370. [Primary record][R23]
- **R24.** Liao et al. *Zipage: Maintain High Request Concurrency for LLM Reasoning through Compressed PagedAttention*. 2026, arXiv:2603.08743. [Primary record][R24]
- **R25.** Zhang et al. *H2O: Heavy-Hitter Oracle for Efficient Generative Inference of Large Language Models*. 2023, arXiv:2306.14048. [Primary record][R25]
- **R26.** Xiao et al. *Efficient Streaming Language Models with Attention Sinks*. Official implementation and scope limitations. [Project repository][R26]
- **R27.** Gao et al. *Scaling and evaluating sparse autoencoders*. 2024, arXiv:2406.04093. Additional context on reconstruction, sparsity, feature evaluation, and training scale. [Primary record][R27]
- **R28.** Guo et al. *On Calibration of Modern Neural Networks*. ICML 2017. [Primary publication][R28]
- **R29.** IETF. RFC 8085, *UDP Usage Guidelines*, 2017. See §3.2 on message size and §3.3 on reliability. [Standard][R29]
- **R30.** IETF. RFC 8949, *Concise Binary Object Representation (CBOR)*, 2020. [Standard][R30]
- **R31.** Yang et al. *Beluga: A CXL-Based Memory Architecture for Scalable and Efficient LLM KVCache Management*. 2025, arXiv:2511.20172. Additional relevant hardware prior work. [Primary record][R31]
- **R32.** Yoon et al. *TraCT: Disaggregated LLM Serving with CXL Shared Memory KV Cache at Rack-Scale*. 2025, arXiv:2512.18194. [Primary record][R32]
- **R33.** Xu et al. *CXL-CCL: Inter-Node Collective GPU-Communication Using a CXL Shared Memory Pool*. 2026, arXiv:2602.22457v3. This is the identifiable related work corresponding to the manuscript's incomplete “CCCL” mention. [Primary text][R33]
- **R34.** CXL Consortium. *Introducing the CXL 3.1 Specification*, official webinar presentation, February 2024. [Official specification overview][R34]
- **R35.** GibberLink authors. Official repository and “How it works” explanation. [Repository][R35]
- **R36.** Gerganov. ggwave official repository. [Repository][R36]
- **R37.** Model Context Protocol. Specification dated 25 November 2025. [Specification][R37]
- **R38.** Agent2Agent Protocol. Version 0.3.0 specification. [Specification][R38]
- **R39.** Leviathan, Kalman, and Matias. *Fast Inference from Transformers via Speculative Decoding*. arXiv:2211.17192v2 / ICML 2023. [Primary record][R39]
- **R40.** Li and Liang. *Prefix-Tuning: Optimizing Continuous Prompts for Generation*. ACL 2021. [Primary publication][R40]
- **R41.** Liu et al. *KIVI: A Tuning-Free Asymmetric 2bit Quantization for KV Cache*. arXiv:2402.02750v2 / ICML 2024. [Primary record][R41]

[R01]: https://arxiv.org/html/2412.08821v2
[R02]: https://arxiv.org/html/2605.08044v1
[R03]: https://arxiv.org/abs/1605.06676
[R04]: https://arxiv.org/html/2510.03215v2
[R05]: https://arxiv.org/abs/2412.06769v4
[R06]: https://arxiv.org/abs/2405.07987v5
[R07]: https://arxiv.org/html/2607.18141v1
[R08]: https://arxiv.org/html/2505.12540v4
[R09]: https://arxiv.org/html/2412.07646v3
[R10]: https://arxiv.org/abs/2006.02419
[R11]: https://arxiv.org/pdf/1706.05125
[R12]: https://arxiv.org/html/2411.02820v4
[R13]: https://arxiv.org/html/2402.07510v5
[R14]: https://arxiv.org/html/2605.25421v1
[R15]: https://arxiv.org/html/2504.02821v3
[R16]: https://arxiv.org/abs/2310.06272v2
[R17]: https://arxiv.org/html/2510.03346v3
[R18]: https://arxiv.org/html/2602.21548v2
[R19]: https://arxiv.org/html/2604.21794v1
[R20]: https://arxiv.org/html/2511.20639v4
[R21]: https://arxiv.org/abs/2309.06180
[R22]: https://vllm.ai/blog/2023-06-20-vllm
[R23]: https://arxiv.org/abs/2604.06370
[R24]: https://arxiv.org/abs/2603.08743
[R25]: https://arxiv.org/abs/2306.14048
[R26]: https://github.com/mit-han-lab/streaming-llm
[R27]: https://arxiv.org/abs/2406.04093
[R28]: https://proceedings.mlr.press/v70/guo17a.html
[R29]: https://datatracker.ietf.org/doc/html/rfc8085
[R30]: https://www.rfc-editor.org/rfc/rfc8949.html
[R31]: https://arxiv.org/abs/2511.20172
[R32]: https://arxiv.org/abs/2512.18194
[R33]: https://arxiv.org/html/2602.22457v3
[R34]: https://computeexpresslink.org/wp-content/uploads/2024/03/CXL_3.1-Webinar-Presentation_Feb_2024.pdf
[R35]: https://github.com/PennyroyalTea/gibberlink
[R36]: https://github.com/ggerganov/ggwave
[R37]: https://modelcontextprotocol.io/specification/2025-11-25
[R38]: https://a2a-protocol.org/v0.3.0/specification/
[R39]: https://arxiv.org/abs/2211.17192v2
[R40]: https://aclanthology.org/2021.acl-long.353/
[R41]: https://arxiv.org/abs/2402.02750v2
