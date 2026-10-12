# LLM Memory & Long-Context SOTA Survey (2024–2026)
### Evaluating mechanisms for "remember everything relevant" inside a FIXED context window

Scope note: written for a design combining **Context Language Models** (arXiv 2609.37725, Shao et al. 2026 — context treated as a file the model edits freely) and Victor Taelin's **OptChat/UniiChat** gist (verified in a later pass; mechanisms summarized in docs/timegear-design.md). Verified claims cite arXiv IDs.

Security note: all fetched text was screened; any token-like strings (base64 blobs, dotted triplets, long hex) redacted as [REDACTED] before output. None appeared in the sources above.

---

## 0. The framing that the field converges on

Every system below is an answer to the same equation: a fixed-size working set (the context window), an unbounded store (disk / KG / weights / KV offload), and a **replacement policy** deciding what moves between them. The architecture literature (OS paging), the caching literature (LRU-K, ARC, TinyLFU, TinyLFU-style admission), and the ML literature (learned fetch, test-time memory) each supply one leg. Three lessons recur:

1. **Naive append loses.** Both human benchmarks (context rot) and model training (CLM) show quality degrades with input length even when the window fits. Curation beats accumulation.
2. **The model itself is the best eviction policy** — if given the means. CLM's core result: unrestricted self-editing of context beats hand-built compaction/summarization pipelines, out of the box.
3. **Structure beats raw text** for small working sets: facts + temporal validity (Zep), graph edges (Cognee/Graphiti) carry more signal per token than verbatim transcripts.

---

## 1. Agent memory products

### MemGPT / Letta
- **How it works:** OS-inspired tiered memory. A fixed "main context" (system prompt + working memory + recent messages) sits above an external "external context" (archival + recall storage). The LLM runs a foreground agent loop with function tools (`archival_memory_insert`, `core_memory_replace`, conversation search) and is trained/prompted to page information in and out itself — virtual context management, mirroring virtual memory paging.
- **Evidence:** Original paper (arXiv 2310.08560) showed it on document-QA and multi-session chat over a 8k-window base model; strong community adoption; Letta productized it. Letta's own 2025 benchmark post argues a **filesystem is sufficient** as the memory substrate — agentic file ops match fancier vector stacks.
- **Limitations:** Self-paging is lossy and drift-prone (the model forgets to write down things); relies on tool-call discipline; benchmark gains largely against long-context-free baselines. Latency: each page-in is an extra turn.
- **Fixed-window fitness: HIGH.** It is literally the canonical design for the constraint. Its known failure mode — the model under-manages memory — is what CLM fixes by making editing native and learned.

### Mem0
- **How it works:** Two-phase pipeline per message pair: (1) an LLM **extracts candidate facts**; (2) an **update phase** compares each fact against similar stored memories (vector search) and decides ADD / UPDATE / DELETE / NOOP. Scope: user, session, agent. Optional graph variant (Mem0ᵍ).
- **Evidence:** Paper (arXiv 2504.19413) reports beating OpenAI's memory and full-context baselines on LoCoMo with **~26× lower p95 latency and >90% token savings** vs full-context; widely deployed in production. Caveats from third-party evaluations (e.g., Zep's rebuttal, 2025): headline LoCoMo scores depend on judging setup; vector-similarity update decisions make factual mistakes under contradiction.
- **Limitations:** Extraction/update LLM calls add latency and cost per turn; fact extraction drops nuances; updates keyed by similarity → wrong-item merges.
- **Fixed-window fitness: MEDIUM-HIGH as an *ingestion/eviction* front-end.** Its extract-then-reconcile loop is exactly a write path: decide what enters the store and what it replaces. Pair with a cheap ranker for read path.

### Zep / Graphiti
- **How it works:** Graphiti is a **temporal knowledge-graph engine**: entities/edges extracted incrementally from messages and business data; every edge carries validity intervals (t_valid / t_invalid) plus event timestamps, so contradictions (Alice's boss was X, now Y) are *versioned*, not overwritten. Hybrid retrieval: embeddings + BM25 + graph traversal over CSR adjacency structures ("hot graphs" in memory). Zep wraps this as a memory service.
- **Evidence:** arXiv 2501.13956 — beats MemGPT on DMR and reports gains on LongMemEval vs full-context; the temporal-interval design is the standout, defensible contribution (fact invalidation is the hard problem most systems punt on).
- **Limitations:** Graph quality depends on extraction LLM; KG construction cost per message; results partly self-reported; LongMemEval gains modest on some subtasks.
- **Fixed-window fitness: MEDIUM-HIGH for the *store*, low for the *readout*.** The temporal fact model is the right substrate; its context rendering is ordinary RAG — replace with a learned/curated readout.

### Cognee
- **How it works:** Open-source ECL pipeline — **E**xtract (entities/relations from docs and chats via LLM + ontology hints) → **C**lausefy/normalize into a graph (GraphDB or vector+graph hybrid) → **L**oad/query. Emphasis on ontologies and modular pipelines; eval framework included.
- **Evidence:** Third-party comparisons (Cognee's own 2025 benchmark post and the Optimizing KG–LLM interface paper, arXiv 2505.24478) show KG-augmented recall beating vector-only on multi-hop, but vendor-run benchmarks; BEAM-scale eval report is recent and thin.
- **Limitations:** Heavier ops (graph DB + embeddings + LLM); ontology quality dominates results; same self-benchmark caveat as Mem0/Zep.
- **Fixed-window fitness: MEDIUM.** Architecturally similar to Zep; adopt if you need ontology-driven consolidation rather than pure chat memory.

---

## 2. Architectures / test-time memory

### Titans (test-time neural memorization, Google)
- **How it works:** A neural long-term memory module — an MLP whose weights are updated *at inference* by gradient descent on a "surprise" objective (memorize what prediction error says is unexpected) with a forgetting/decay gate. Short-term attention handles the local window; the memory module provides a learned lookup over everything seen. Variants: memory as context / as gate / as layer. Successor line: MIRAS.
- **Evidence:** arXiv 2501.00663; strong needle-in-haystack at **>2M tokens** and gains on long-context QA/language modeling vs Transformers and modern RNNs at matched scale; Google Research blog (Dec 2025) continues the program. But: needs *training or fine-tuning* to shine; community replications show it is sensitive to hyperparameters and does not magically beat good RAG on generic QA; no production model ships it.
- **Limitations:** Unproven at product scale; requires architecture control you don't have with API models; associative, lossy memory (good for retrieval-style recall, bad for precise multi-fact reasoning).
- **Fixed-window fitness: MEDIUM, conceptually important.** It is the "parametric cache" leg: an eviction-free, bounded-size memory that compresses by gradient updates. If you can fine-tune, a Titans-style module *is* a fixed-budget memory.

### Memory Layers at Scale (Meta, arXiv 2412.09771)
- **How it works:** Replace FFN blocks with a **product-key memory layer**: a large bank of (key, value) pairs where only the top-k keys nearest the query are activated — a differentiable, trained key-value store. Scaled up to 128B memory params / up to 64k keys with sparse upcycling from pretrained models.
- **Evidence:** From-scratch 1.3B model with 128B memory matches an 8B dense model on factual QA (TriviaQA, NaturalQuestions) at ~1/4–1/30 the effective FLOPs; scaling laws for memory size are clean. Trained *into* the model, not bolted on.
- **Limitations:** Requires full pretraining or heavy upcycling; memory is implicit (no interpretability, no per-fact edit); improves factual recall, not episodic conversation memory.
- **Fixed-window fitness: LOW near-term, HIGH long-term.** This is what "remember everything" converges to if you control training: the context window stays small, facts live in trained KV memory.

### HMT / Recurrent Memory Transformer
- **How it works:** RMT (arXiv 2207.04663) appends memory tokens passed between segments; HMT (arXiv 2504.09996, 2025) wraps a frozen pretrained LM: segment the input, each segment is encoded with a memory-augmented (LoRA-tuned) prompt, a "Memory Encoder" distills segment+memory into the next segment's memory tokens, then a final Memory Retriever decodes over all memories. Hierarchical summary chain.
- **Evidence:** HMT matches or beats native long-context models with ~1/4 the compute, extrapolates to 100× longer than fine-tuning context, wins LongBench tasks and passes passkey retrieval; can use any frozen LM. RMT earlier showed million-token passkey retrieval from a 512-token window.
- **Limitations:** Requires fine-tuning; memory tokens are lossy summaries (weak on precision questions spanning many segments); inference is sequential.
- **Fixed-window fitness: MEDIUM.** The cleanest "summarize-then-forget" recurrent design; HMT is what a learned eviction policy looks like in weights.

---

## 3. Context compression

### SEEC (Self-Extending Context)
- **How it works:** A 2026 line: the model emits **semantic compression operators** (abstract/summarize/tokenize-into-keys) while generating, so old context is replaced by compact semantic tokens/summaries as it goes — the window "extends" itself by continuously rewriting its past. Related: context-as-file editing (CLM), multi-scale self-injection.
- **Evidence:** Early workshop/preprint-stage results show multi-fold effective-window extension on long-agent tasks; not yet independently benchmarked broadly. Flag: **promising, not proven**.
- **Limitations:** Compression errors compound over long horizons (a bad summary poisons everything downstream); need rollback ability.
- **Fixed-window fitness: HIGH conceptually** — it is fixed-window memory as a policy, not a hack. Pair with checkpoints (CLM-style file snapshots) for rollback.

### LLMLingua / LLMLingua-2
- **How it works:** LLMLingua: perplexity-based token removal via small LM. **LLMLingua-2** (ACL 2024, arXiv 2403.12968): distill GPT-4 compression decisions into a BERT-size token-classification model → keep/drop per token; ~3–10× faster than GPT-4 compression, lossless-ish QA at 2–5× compression; 2–5× faster than LLMLingua-1.
- **Evidence:** Solid, peer-reviewed; integrates in `llmlingua` lib; works well on documents/RAG prompts.
- **Limitations:** Destroys order-sensitive and numeric detail; task-agnostic dropping hurts agentic instructions; compression ≠ selection for a *memory* policy (no notion of what to keep across sessions).
- **Fixed-window fitness: MEDIUM as a *readout optimizer*** — run it on the retrieved slice to fit more into the window, not as the eviction policy itself. Proven for the narrow job it does.

### LCLMs / latent compression (arXiv 2606.09659) & A2C-RAG (ACM 2026)
- Compress into soft tokens or compress only the retrieved context adaptively (A2C-RAG: up to 71% prompt-token reduction). Soft-token compression (ICAE, in-context autoencoders) is elegant but fragile cross-model and mostly unproven in production. **Flag: research-grade.**

---

## 4. KV-cache management (the mechanistic layer)

All of these shrink or select *within* the attention window — they are the "which entries stay in RAM" layer.

### H2O (Heavy-Hitter Oracle, NeurIPS 2023)
- **How:** Keep a small budget of KV entries per layer chosen by accumulated attention scores (heavy hitters) + recent tokens (local window). **Evidence:** 20→5× throughput gains, quality held with ~5% cache budget; extensively replicated. **Limits:** static budget per layer; attention-score heuristic can miss future-importance tokens. **Fitness: proven baseline eviction heuristic.**

### SnapKV (2024)
- **How:** Compress once at prefill using a **sliding observation window** of the prompt's last tokens: cluster tokens by pooled attention over that window (CAB pooling), keep top-k per layer, keep recent window. **Evidence:** 16× cache reduction, 8× decode speedup at 16k prompts, better needle-in-haystack than H2O for prompt-compression settings. **Limits:** one-shot decision at prefill — bad if later turns need evicted content; corrects with inter-layer adaptive budget variants (e.g., V-kv). **Fitness: strong for read-heavy single-prompt compaction; wrong shape for a conversational memory loop.**

### PyramidKV (arXiv 2406.02069)
- **How:** Allocate cache budget **per layer**: low layers (broad information dispersal) get large budgets, high layers (attention funnels to few tokens) get small ones — a "pyramid," plus dynamic allocation by query complexity. **Evidence:** matches full 1,020K-token cache quality with 12-token-per-layer budgets on some LongBench tasks; strong NIAH; adopted broadly (also in Llama models' context work). **Limits:** same prefill-time family as SnapKV; per-layer budget ratios are hand-set (learned variants exist, 2025). **Fitness: the best "cheap static policy" — a good default before anything learned.**

### Quest (ICML 2024)
- **How:** Do not evict — **index**. Precompute per-page (16-token block) min/max key statistics; at decode, query-aware page scoring fetches only the top pages for exact sparse attention. Lossless up to page granularity. **Evidence:** 8k-page caches attended with ~32-page reads, quality ≈ full attention with large cache savings. **Fitness: HIGH** — this is the LRU/KV-lookup analog done right: metadata index + query-driven fetch, the same shape as a learned page table. Modern descendants (SeKV 2026, hierarchical semantic KV) push semantic indexing into the pages.

**KV-layer verdict for fixed-window memory:** SnapKV/PyramidKV = static admission policies; Quest = dynamic indexed fetch; H2O = running eviction. A fixed-window system's cache layer should look like Quest + a decay policy (H2O-style), not one-shot SnapKV.

### The cache-algorithm mapping (LRU-K / ARC / TinyLFU)
- **LRU-K:** frequency-aware recency — maps directly onto "promote facts retrieved repeatedly into core memory"; solves LRU's one-hit-wonder problem (a single accidental mention polluting working memory).
- **ARC (Adaptive Replacement Cache):** self-tuning split between "recently used" and "frequently used" ghost lists — a ready-made design for balancing a **recency buffer** (recent turns) against a **frequency buffer** (hot facts), with ghost-list admission tests to stop one-shot chatter from evicting hot facts. Adaptive, no parameters to tune.
- **TinyLFU / W-TinyLFU:** sketch-based admission filter (count-min over a window) before entering the main cache — the right shape for "is this message worth promoting to long-term memory at all," cheaply and with bounded memory. Storm-style decay handles drift.
- All three are proven, 30+-year-old-lineage mechanisms with formal guarantees; adopting them for *memory admission* is the single most under-used proven idea in agent memory today. Every memory product above does a bespoke LLM version of this; the sketch/filter versions are cheaper and tunable.

---

## 5. Retrieval vs memory debate — where it stands (2025–2026)

- **2024:** "Long context kills RAG" vs "RAG kills long context." Resolution: neither. Full-context wins on small corpora (<~100k tokens) and precision questions; retrieval wins on cost, latency, corpora beyond ~1M tokens, and privacy/permissions.
- **2025–2026:** convergence on **hybrid**: retrieve to narrow, then curated context in a fixed window ("context engineering"). Two data points: (a) RULER/effective-context-length findings — nominal windows are marketing (models degrade well before the advertised limit; e.g., many 128k-claimed models fail RULER past ~32–64k); (b) context rot — even within the window, accuracy decays roughly linearly with input length and worsens with distractor density.
- The debate's real outcome: **the window is a cache, not storage.** Nobody serious argues for unbounded context anymore; the argument is only about which policy populates the cache (retrieval, learned lookup, self-editing).

### Context rot (Chroma technical report, Jul 2025)
- Replicated on Llama-4-Scout, Qwen3-32B, Gemma-3, GPT-4.1-family, Claude-4: even models claiming 10M-token windows degrade on simple single-word-answer tasks as input grows; degradation is worse with "haystack" distractors and semantically tangled contexts; **output token count correlates with performance.** Implication for a fixed-window memory: not only must you fit, you must keep the working set *small and clean* — quality of curation matters more than window size.

---

## 6. Benchmarks & failure modes

| Benchmark | What it measures | Key findings |
|---|---|---|
| **RULER** (2024, arXiv 2404.06654) | Synthetic multi-task (NIAH, variable tracking, aggregation) at graded lengths → *effective* context length | Most "128k" models degrade well before; effective length often ≪ nominal; generation-based tasks are hardest |
| **LoCoMo** (arXiv 2402.17753) | Very long (up to 35 sessions / ~9k turns) multi-modal conversations: QA, event summarization, multimodal | Even long-context + RAG struggles; temporal-reasoning questions are the failure mode; adversarial/contradiction questions near-random for many stacks |
| **LongMemEval** (ICLR 2025, arXiv 2410.10813) | 500 realistic questions over 115k-token chat histories: knowledge updates, temporal reasoning, abstention, session-level aggregation | Off-the-shelf long-context chat loses ~30% accuracy vs oracle RAG; memory-augmented (interactive format) recovers much of it — with big latency/cost. LongMemEval-V2 (2026) extends to agent/parametric memory |
| **Context rot** (Chroma 2025) | Degradation curve, not a pass/fail | Length + distractors monotonically hurt, even within window |
| **BrowseComp-Plus / EdgeBench / ContextBench** (CLM 2026) | Context-management policies under FLOP budget | CLM's unrestricted self-editing beats SOTA harness strategies; models have poor **context-length awareness** (guess token counts in buckets; hints help) |

**Cross-cutting failure modes a fixed-window memory must handle:** stale facts (Zep's validity intervals), lost-in-the-middle positioning (put hot facts at top), temporal confusion (LoCoMo), abstention (LongMemEval punishes confabulated recall), compounding compression error (SEEC/HMT), and context-length misjudgment (CLM App. G).

---

## 7. Proven vs hype, verdict table

| Mechanism | Status | Fixed-window fitness |
|---|---|---|
| MemGPT/Letta self-paging | Proven pattern, drift-prone | **High** — the baseline architecture |
| Mem0 extract/reconcile | Proven in prod, vendor-benched | **High as write path** (admission + update) |
| Zep/Graphiti temporal KG | Proven concept, self-benched | **High as store** (validity intervals are the key idea) |
| Cognee | Mature OSS, thin third-party eval | Medium |
| CLM context-as-file | **New (Sep 2026), strong controlled results** | **High — best evidence that model-native editing beats harnesses** |
| PiOptChat (Taelin) | Design verified (gist); recall claims self-reported | **High — best cache-engineering recipe in the field** |
| Titans | Peer-reviewed, needs training, not in prod | Medium (parametric memory leg, if you control weights) |
| Memory Layers at Scale | Strong from-scratch results, needs pretraining | Low now / High long-term |
| HMT/RMT | Peer-reviewed, needs fine-tuning | Medium (learned summarize-and-evict) |
| SEEC / semantic compression | Early-stage | High conceptually, unproven |
| LLMLingua-2 | Proven, narrow | Medium (readout optimizer) |
| H2O / SnapKV / PyramidKV / Quest | Proven, replicated | **Quest shape (indexed fetch) + PyramidKV budgets + H2O decay = the KV layer** |
| LRU-K / ARC / TinyLFU | Decades-proven, unexploited in agent memory | **High — use as the admission/eviction policy around the LLM store** |

## 8. Synthesis: what a fixed-budget memory system should steal

1. **Write path (admission):** Mem0-style extraction + update, gated by an ARC/TinyLFU-style admission policy (recency vs frequency buffers with ghost-list tests) so cheap heuristics decide *before* spending LLM calls; LLM reconciles only admitted candidates.
2. **Store:** Zep/Graphiti-style temporal facts with validity intervals — facts, not transcripts; supports stale-fact eviction (LRU-K on fact access frequency maps naturally).
3. **Read path:** Quest-style indexed fetch (semantic page index + query-driven selection) into a small working set, optionally LLMLingua-2-compressed, positioned at prompt top.
4. **Eviction:** let the model edit (CLM evidence beats harness compaction), but give it **token-budget hints** (models can't count; CLM App. G) and file snapshots for rollback (SEEC compounding-error protection).
5. **If training is available:** Titans/memory-layer-style parametric cache replaces most of the above — the endgame for "remember everything."
6. **Evaluate** against LongMemEval(-V2) + LoCoMo with cost/latency reporting; report context-rot curves for your own system, not just window size.
