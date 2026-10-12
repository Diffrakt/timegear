# TimeGear

Research into a memory architecture that keeps an LLM useful forever inside a
**fixed context window**. The window is a cache, not storage: an append-only
store grows without bound, and a three-tier policy decides what the model sees.

**Verdict:** operational memory is achievable: nothing ever needed is
unrecoverable, and recovery cost is bounded. The design combines the two
strongest current sources:

- **Context Language Models** (arXiv 2609.37725) — the model edits its own
  context as a file and beats every hand-built compaction harness at lower cost.
- **OptChat / UniiChat** (Victor Taelin) — the append-only chat log as memory,
  a binary summary tree, a constant-size view, zoom-to-verbatim retrieval,
  ~98.6% prompt-cache hits.

plus temporal fact stores with validity intervals (Zep/Graphiti), sketch-based
write admission (TinyLFU/ARC behind a Mem0-style reconcile step), and
System 1 / System 2 routing so only judgment calls ever reach the model.

## Layout

| Path | What |
|---|---|
| `docs/timegear-design.md` | The design: source evidence, 26 Reddit criticisms with verdicts, the four-tier architecture, System 1 mapping, benchmark plan |
| `docs/memory-sota-survey.md` | 2024-2026 memory and long-context literature: what is proven, what is hype |
| `sources/` | Raw captures of the two r/LocalLLaMA discussions behind the criticism tables |

Status: design complete, benchmark not yet run. The build order is at the end
of the design doc.
