# AGENTS.md

Standing instructions for anyone working in this repo (agents included).

## What this is

TimeGear: research into memory architectures that keep an LLM useful inside a
fixed context window. Docs are prose research with citations and verdicts.
Code lands here when the build phase starts (see the build order at the end of
docs/timegear-design.md).

## Layout

- docs/ - finished research: claims, evidence, verdicts
- sources/ - raw captures backing the docs (public web content only)
- README.md - brief overview

## Hard rules

- Never commit personal data: local file paths, real names, emails, usernames,
  tokens, API keys, cookies, signed URLs, or anything about the owner's other
  projects or machines. This repo is public.
- Before every push, sweep staged files for: drive letters and absolute paths,
  tokens (jwt=, eyJ, gho_, ghp_, sk-, Bearer), email addresses, and anything
  that looks like a credential. If unsure, redact.
- Raw captures go in sources/ only when they are public content.
- Claims in docs/ must name their source (arXiv id, URL) or be labeled
  self-reported or unverified. The criticism tables set the format: every
  criticism gets a verdict and an addressability note.

## Workflow

- Research threads: read fully, extract every claim and criticism, write
  verdict tables into docs/, keep the raw capture in sources/.
- Keep docs ASCII-clean and grep-friendly.
- Small commits, plain messages, push to main.

## Repo state

- Done: source synthesis, literature survey, criticism analysis, architecture
  design (docs/timegear-design.md).
- Not done: the benchmark (LongMemEval-V2, LoCoMo, and a recall protocol) and
  the implementation. Neither source being synthesized had a recall benchmark;
  shipping one is what separates the claims here from the "trust me" memory
  systems the discussions criticized.
