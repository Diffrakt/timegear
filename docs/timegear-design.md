# TimeGear: a fixed-window memory architecture

Synthesis of Context Language Models (arXiv 2609.37725), Victor Taelin's
OptChat/UniiChat gist, two r/LocalLLaMA critique threads, and 2024-2026
memory research. Sources are cited inline; claims from unverified sources
are labeled.

## Verdict

Yes, the combination is possible, and the two main sources are
complementary rather than competing:

| Property | CLM (Shao et al.) | OptChat (Taelin) | Combined |
|---|---|---|---|
| What manages context | The model, freely editing it as a file | A fixed harness recipe (tree + zoom) | Tree as invariant skeleton, model edits regions and leaves |
| Eviction policy | Learned (emergent, or RL'd) | Fixed (binary-counter due-pair merges) | Hybrid: model edits inside stable regions, merges under System 1 |
| Retrieval | Whatever the model writes | zoom() down a summary tree, verbatim log | zoom + semantic index + quoted leaves |
| Exact recall | Yes (file holds verbatim text) | Yes (append-only log, zoom to message) | Yes at every tier, cheap path always reachable |
| Prompt cache | Breaks on mid edits (SCR mitigates, stale-KV caveat) | ~98.6% hit (append-only view, batched merges) | Append-only envelope keeps the hit rate |
| Failure handling | Safety/persistence attacks, stale KV | Summarizer hallucination poisons tree | Quoted leaves + append-only tombstones + multi-candidate compaction |
| Evidence | Strong controlled benchmarks | Anecdotal (author self-study ~75% recall) | Inherits both; needs its own benchmark (see section 6) |

What can be claimed outright: perfect, lossless, unbounded memory is
information-theoretically impossible inside a fixed window. A summary of
N bytes cannot answer every possible question about what it summarizes.
But the correct standard is not "never lose a byte", it is "lose nothing
that is ever needed, and pay bounded cost to recover anything else". That
standard is achievable, because a fixed window is a cache, not storage:
the store can be unbounded outside the window, and what matters is the
the admission, eviction, and fetch policy between them. The rest of this
document is that policy.

## 1. What each source contributes

### CLM (Shao et al., arXiv 2609.37725)

- Definition: a standard LM only appends (c_{t+1} = c_t + f(c_t)). A CLM
  maps context to context, c_{t+1} = f(c_t), implemented as a context
  file the model edits with ordinary tools, synced back to the live
  context. That single change is the whole architecture.
- Zero-shot evidence (no training): BrowseComp-Plus 59.4% at a 32K
  budget, 11.4% relative over Codex-style summarization with 21.5%
  fewer prefix-reuse FLOPs; TB2.1 parity at 70% of the FLOPs; TBLite
  73.7 vs 67.0; EdgeBench-10 44.6 vs 42.3 at 179 vs 437 PFLOPs
  (Qwen3.6-27B, 12 hours); Software World 65% greater downstream
  speedup at equal spend; best score on all four OpenEvolve math
  problems.
- Learnable: one-sentence steering changes compaction behavior; evolved
  skill documents lift held-out ContextBench accuracy up to 35.9 points;
  stepwise GRPO with a success-gated efficiency advantage lifts
  Qwen3.5-9B from 28.8 to 42.5 on BrowseComp-Plus, matching the trained
  summary harness at 1.34 vs 2.19 PFLOPs per question.
- Serving: Suffix Cache Reuse keeps KV of surviving tokens across
  mid-context edits (65% of standard SGLang compute on BCP), but those
  spans keep stale KV. The paper claims this can even help; the Reddit
  rebuttal is correct that the accuracy effect is unverified.
- Weaknesses the paper itself documents: models have poor context-length
  awareness (they guess in buckets; token-count hints near the edit
  point help substantially), and a model-editable context is a
  persistence channel for injected instructions (the paper cites
  observed unauthorized instructions inside compaction summaries).

### OptChat / UniiChat (Taelin gist; community-tested)

- Core: the append-only log IS the memory. A cheap model builds a binary
  tree of one-line summaries (each node <= 512 bytes, built once, never
  rebuilt). Every turn is a fresh call over a constant 64-128 KB "view"
  of the whole chat, fine near the present and coarse in the past, and
  zoom(id, n) opens any line into its two halves down to the verbatim
  message.
- Eviction is a solved ordering problem, not a heuristic: the
  rollback-push / binary-counter merge order (due = (T - last) / 2^l,
  merge the most-due built pair, batch when the view passes 128 KB,
  drop back to 64 KB). O(log N) lines cover N messages; old lines stay
  stable, new lines churn.
- Cache engineering: fixed [tools][system][view][message] layout, the
  view only grows at its end between merge batches, batched merges cut
  rewrites about 4x. Measured 98.6% prefix cache hits on turns.
- Retrieval discipline lives in the prompt: zoom before acting, never
  answer from summaries, "say in your reply what you learned", and
  user words dominate the summaries so corrections survive up the tree.
- Honest gaps: no public recall benchmark (author's own blind study:
  ~75% on ~40 hard questions), summarizer hallucination poisons the
  tree upward (worse with small models), the positional tree buries
  topics, and zoom-only navigation costs turns (community regex search
  cut zoom calls from 6 to 2.4 per question).

## 2. The Reddit criticisms, with verdicts

Thread 1 (CLM paper), 14 criticisms examined. Full analysis:
sources/reddit-clm-thread.md (raw capture in this repo).

| # | Criticism | Verdict | Addressable |
|---|---|---|---|
| 1 | Any edit invalidates cache after the change point | Valid | Partly: SCR/splice; edit-tail-only discipline |
| 2 | Spliced KV after an edit is stale (encodes the old prefix); length changes need key rotation; worse for exotic attention | Valid, fundamental | Only full recompute truly fixes it; treat stale-KV as an error rate to measure |
| 3 | "Stale KV must hurt accuracy somewhere" | Valid | Same as #2: measure fault rate vs compaction |
| 4 | Edits near the top force full reprocessing on SWA models | Valid for SWA architectures | Keep early context immutable |
| 5 | Summary baseline nearly matches; only small Qwen models tested | Partly valid: the out-of-box deficit is 9B-specific (paper says so), but large-model coverage is a real gap | Independent replication |
| 6 | Efficiency numbers lean on a dirty SGLang hack | Partly: most results do not use SCR and still win on FLOPs | Cleaner engine support |
| 7 | "Surgery on your own brain", brittle | Plausible pre-RL | RL, guardrails, undo |
| 8 | No comparison vs Ralph-loop + handoff | Valid gap | Run the comparison |
| 9 | "Just cache compaction that drives models insane" | Invalid: model-chosen edits beat fixed templates; 20+ hr runs held up | n/a |
| 10 | llama.cpp has no such cache; latency cost real | Valid | Engine work only |
| 11 | Cache-busting economics punish edits; "cache is king" | Valid | Append-only envelope (this design) |
| 12 | Editable context amplifies prompt-injection persistence | Valid, structural | Diffable edit history + per-edit revert |
| 13 | "SillyTavern in paper form" | Partly: editing is old; model-driven editing is the new part | n/a |
| 14 | Engines should let models edit during thinking | Valid idea | Engine feature |

Thread 2 (pi-optchat), 12 criticisms examined. Full analysis:
sources/reddit-optchat-thread.md (raw capture in this repo).

| # | Criticism | Verdict | Addressable |
|---|---|---|---|
| 1 | zoom = disk reads mid-inference; no latency benchmarks | Valid | Build the benchmark |
| 2 | Tiered views should kill KV caching | Raised well, already answered: stable positional view, 90-95% hit | n/a |
| 3 | Summaries are focus-dependent; the hidden detail never invites a zoom | Valid, deepest structural criticism | Quoted leaves + multi-tree (this design) |
| 4 | Vague "keep what's important" prompts; no comparison vs long context | Valid; author concedes | Empirics needed |
| 5 | Summarizers cannot anticipate future needs | Valid for topical memory | Positional tree + search mitigates |
| 6 | Small-model summarizers invent details that poison the tree | Valid, worst failure mode | Never let a small model write unquoted memory |
| 7 | Paid summarization is off-theme for LocalLLaMA | Valid locally | Local summarizer is the bottleneck |
| 8 | pi-vcc is instant and needs no LLM | Valid tradeoff | Pick per use case |
| 9 | Imported sessions: which reply belongs to which session | Valid, unresolved | Tag lines with session ids |
| 10 | "Anything works; 'infinite memory' is a lie" | Valid skepticism | Ship the benchmark |
| 11 | OP's replies were Claude-written | Non-technical | n/a |
| 12 | "Fancy memory system, big woop" | No substance | n/a |

Alternate systems named in the threads: Ralph-loop + handoff, RLM
(recursive language models), pi-vcc (instant compaction), billion-context
("bili"), prompt-based memory files + fact index, agent self-edited
context, LRU-decay knowledge base, dynamic knowledge graphs. All are
special cases of the same cache equation; none adds a mechanism this
design does not already place.

## 3. The architecture: TimeGear

Four tiers. Three of them never change size. Only the store grows.

| Tier | Contents | Size | Who writes it | Source |
|---|---|---|---|---|
| T0 Hot slab | Verbatim tail + "quoted leaves" (compressed message pairs that keep key spans verbatim) | Fixed (8-16% of window) | Harness (mechanical) | OptChat leaves |
| T1 The view | id+n|line summary tree over the store | Fixed (60-70%) | Compactor (cheap model) | OptChat tree + due-merge |
| T2 Slot index | One-line structured slots: entity, fact, value, valid_from, valid_to, src_msg | Fixed (10-20%) | System 1 + LLM reconcile | Zep intervals, Mem0 write path |
| T3 Store | Append-only log + tree nodes + slot rows + dirty queue + diff journal | Unbounded | Append only | OptChat log; deletion rule |

The prompt layout is fixed and stateless: [tools][system][T2 slots][T1
view][T0 slab][new message]. Nothing with a date or counter ever enters
tools or system prompt (OptChat mistake list). The model gets a budget
telemetry line after every tool result, because CLM Appendix G shows
models cannot count their own context.

The dirty queue is the other half of the CLM lesson: when the model edits
its own file, it lists which regions changed and why. Only those lines
are re-summarized. This is surgical self-editing at line granularity
(CLM's win) without mid-context cache invalidation (CLM's cost), since
the edit lands in an append queue and the affected lines merge at the
next batch, like any other pair.

Write path, per message (System 1, all cheap, no frontier model):

1. Every message is appended to T3. Deletion is forbidden; anything
   removed from the window leaves a tombstone with a handle (a
   no-delete policy, applied to memory).
2. Admission filter (TinyLFU/ARC: count-min sketch over recency window,
   ghost-list tests): is this candidate worth a slot? Churn and one-shot
   mentions are rejected without an LLM call.
3. LRU-K promoted candidates go to a cheap extract/reconcile call
   (Mem0's write path): ADD, or supersede an existing row by closing
   its valid_to and opening a new one (Zep). Contradictions are
   versioned, never overwritten.
4. The leaf compactor runs OptChat's job with one change: it returns the
   line plus 1-3 verbatim quoted spans (ids, paths, numbers, exact
   corrections). The new node is line + quotes: a Quoted Leaf. A quote
   is copy, not generation, so a small-model summarizer cannot invent it.

Read path, per turn (System 2, the frontier model, 1-3 zooms):

1. The model reads the fixed view and the slot index.
2. zoom(id,n) opens a leaf into its two halves, down to the verbatim
   message; zoom(id,1) with quotes usually answers without step 3.
3. grep/recall (exact and regex) over T3, promoted to T2 or T0 as a
   side effect. This answers the focus-dependence criticism: retrieval
   is by index and exact text, not by trusting a summary to have kept
   the right thing.

Merge policy: exactly OptChat's. due = (T - last)/2^l, merge the
most-due built pair, oldest on ties; append-only between batches; one
batch when the view crosses its high-water mark; persist view.json and
never rebuild it (a rebuilt view differs and kills every cache entry).

## 4. The System 1 / System 2 mapping

This is where the dual-process framing lands concretely. The rule:
every decision that is a pattern match goes to a fast, cheap, untrained
mechanism; every decision that needs judgment goes to the model; and
frequent System 2 verdicts get distilled back into System 1.

| Decision | System 1 (fast, cheap, no LLM) | System 2 (model) |
|---|---|---|
| What to admit to memory | TinyLFU sketch + ARC ghost tests (microseconds) | Only reconciles admitted candidates |
| What to evict | due = (T - last)/2^l ordering; fixed budgets per tier | Marks regions dirty; can pin |
| What to fetch | Slot index lookup; exact/regex search (milliseconds) | Chooses zoom targets, resolves ambiguity |
| What is urgent | KPIs, failures, corrections ride the always-verbatim T0 slab | Decides what becomes a slot |
| When to compact | High/low water marks (128K/64K analog) | Emergency compaction when telemetry says so |
| Policy itself | Fixed defaults | CLM skill-document evolution loop refines the written policy; RL (stepwise GRPO + efficiency advantage) trains the model if weights are available |

The compactor itself is a System 1 model: Haiku-class, xhigh effort,
same system prompt and tools as the turn so its input shares the turn's
cache prefix (OptChat mistake #6). On a local llama.cpp stack a 3-8B
compactor is the practical floor; the honest
caveat is that small models hallucinate in summaries, which is exactly
why quoted leaves exist.

## 5. Why the combination is strictly better than either half

- OptChat alone loses information silently when a summary omits the
  detail a future question needs (criticism T2#3/#5) and has no fact
  model (stale "truths"). Quoted leaves make loss recoverable at the
  leaf level; slots with validity intervals make staleness explicit;
  grep guarantees a path that does not trust summaries at all.
- CLM alone pays cache invalidation and stale-KV risk on every edit
  (criticism T1#1-3) and amplifies prompt-injection persistence
  (T1#12). The append-only envelope keeps ~90%+ prefix cache hits; the
  diff journal with per-edit revert makes every edit bidirectional and
  auditable.
- Both alone fail on benchmarks nobody ran. Section 6 fixes that.

Failure modes and their counters, in one place: summarizer
hallucination -> quoted spans are copied, and merges only combine
already-built nodes (errors cannot compound past one level without a
verbatim anchor); prompt injection -> compaction input is data, never
obeyed (OptChat prompt line), plus diff/revert; small models -> they
never write unquoted memory; lost-in-the-middle -> slots at the top,
hot slab at the bottom, view in between; context rot -> the window
holds curated tiers, never raw accumulation (Chroma 2025); cache
death -> nothing is ever rewritten in place except one batch at the
water mark.

## 6. Honest limits and the missing benchmark

What cannot be claimed: zero information loss (impossible at fixed
size), perfect recall of arbitrary detail (the ~75% self-study figure
is the only empirical number in this whole space, and it is the honest
prior), or that stale-KV splicing is harmless (unverified, as Reddit
correctly insists). What can be claimed: bounded cost per turn, bounded
loss per tier with a cheap recovery path, and a measurable design.

The benchmark, because neither source has one: run LongMemEval-V2 and
LoCoMo plus the community's "40 hard questions" protocol against (a)
plain long context, (b) Codex-style compaction, (c) OptChat as
published, (d) TimeGear. Report accuracy, prefix-cache hit rate,
cost/turn, zooms/question, and a recall-under-topic-density axis, which
is where positional trees are weakest. Anything short of that is the
same "trust me" both Reddit threads called out.

Build order, minimal: (1) OptChat replica (log, tree, view, zoom,
batched merges) with view.json persistence; (2) quoted leaves in the
compaction prompt; (3) grep over the log; (4) T2 slots behind grep with
TinyLFU admission; (5) dirty-queue self-editing; (6) the benchmark; (7)
skill-document evolution loop. Steps 1-3 are a weekend; step 6 is what
makes the claims real.