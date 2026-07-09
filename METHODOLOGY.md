# Methodology

CONFIRM-1 was a **confirmation** campaign, not an exploration. Every expected outcome was stated in advance, the system configuration was frozen at a single commit, and every run was sealed at creation. Its purpose was to produce a uniform evidence base: replicated means with dispersion for every number that appears in the paper.

This document summarises the pre-registered specification. It omits the system-internal configuration values that are not published.

---

## Design

Five arms. Each changes exactly one variable relative to the control, so its difference isolates that component. Arms were never combined.

| Manipulated variable | Levels | Arms |
|---|---|---|
| Storage unit | Round-level / turn-level | H-C0 / H-A1 |
| Re-ranking | MMR / cosine only | H-C0 / H-A2 |
| Memory layer | On / off (full history in context) | H-C0 / H-B0 |
| Actor | Claude Haiku 4.5 / GPT-4o | H-C0 / G-C0 |

Uniform n=3 across every arm. Fifteen counted runs. No mixed replication counts anywhere in the campaign or the paper.

Held identical across all runs and verified by manifest diff: benchmark file hash, judge snapshot and parameters, scoring script, frozen commit, embedding weights checksum, container image digests, hardware, and dataset order.

---

## Benchmark

**LongMemEval-S**, the small variant, all 500 questions per run. No sampling.

File `longmemeval_s_cleaned.json`, from the authors' Hugging Face mirror `xiaowu0162/longmemeval-cleaned`, used unmodified under the upstream MIT licence. Its SHA256 is recorded in every manifest and must match across runs; this detects silent dataset drift.

Categories: single-session-user (70), single-session-assistant (56), single-session-preference (30), knowledge-update (78), temporal-reasoning (133), multi-session (133).

The preference category is the smallest at 30 questions. We flag its small sample wherever it is discussed.

### On LongMemEval-V2

LongMemEval-V2 targets agent-experience memory over multimodal web-agent trajectories with a latency-weighted leaderboard metric. It is a different task on different data and does not supersede LongMemEval-S for conversational memory question answering. CONFIRM-1 makes no V2 claim, and we scope it as future work.

---

## Actors and judge

| Role | Model | Max tokens | Temperature |
|---|---|---|---|
| Primary actor | Claude Haiku 4.5 | 512 | Provider default 1.0, unset in code |
| Secondary actor | GPT-4o (`gpt-4o-2024-08-06`) | 1,024 | Unset |
| Judge | GPT-4o (`gpt-4o-2024-08-06`) | 10 | 0 |

All model identifiers are pinned to dated snapshots and recorded in every manifest. Floating model aliases are prohibited.

Actors run at provider-default temperature, so runs are stochastic. Replication is by independent repeated runs, not seed control, and run-to-run dispersion is the reported uncertainty. This is stated up front rather than treated as a caveat.

Scoring uses the upstream LongMemEval evaluation script, unchanged, to preserve comparability with published usage of the benchmark. Judge retries use exponential backoff on rate-limit and API errors, and every retry event is logged.

---

## Measurement definitions

1. **Mean and SD**: sample mean and sample standard deviation (n-1 denominator) over the runs in an arm.
2. **95% CI**: t-distribution, df = n-1, on the arm mean.
3. **Arm delta**: mean(arm) − mean(H-C0), reported with both standard deviations.
4. **Paired test**: McNemar's exact test on the 500 paired per-question outcomes. The pairing key is the question id. Reported on the consensus outcome per arm.
5. **p95 latency**: the 95th percentile of per-question package-assembly latency within a single run. The latency claim uses **only** solo-flagged runs; all other runs report latency as context with their concurrency width attached.
6. **Rounding**: one decimal place for percentages, two for standard deviations.
7. **Stability**: the percentage of the 500 questions whose judged outcome is identical across all three runs of an arm.
8. **Context cost**: mean input tokens per question and total run tokens per arm, from captured usage metadata.

---

## Pre-registered analyses

Stated before the first counted run. No expectation could be edited after that point; the specification became append-only, and deviations were logged with timestamps rather than edited in place.

| Analysis | Pre-stated expectation | Outcome |
|---|---|---|
| Control mean, SD, CI | Mid-80s, SD of order 0.6 pp | Met (85.4%, SD 0.9) |
| Cross-actor mean | Below control by low single digits | Met (81.4%, −4.0 pp) |
| Memory-layer delta and paired test | Large positive, decisive | Met (+39.9 pp, p < 10<sup>-47</sup>) |
| Round-level delta and paired test | Large positive, decisive | Met (+22.5 pp, p < 10<sup>-25</sup>) |
| MMR delta and paired test | Positive, significant | **Not met.** +0.7 pp, p = 0.38 |
| Safety counter | Zero, every run | Met (0 / 7,500) |
| p95 latency, solo runs | Under 80 ms | Met (30–35 ms) |

The MMR expectation was not met. We report the result as pre-registered, in the direction the data gives, and do not reframe it after the fact.

### Secondary endpoints

Preference-category behaviour under round-level ablation was pre-registered to be reported **regardless of direction**, as were per-category deltas for every ablation. Every negative cell in every reported table receives a sentence in the body text.

Any analysis not pre-registered is exploratory and must be labelled as such wherever it appears.

---

## Evidence rules

The campaign operated under explicit rules, fixed before any counted run.

| Rule | Statement |
|---|---|
| R1 | A run without a sealed, checksummed package is a failed run |
| R2 | The manifest is written and validated **before question 1**; the run aborts if it is incomplete |
| R3 | Dual archiving is part of the run, not a later chore |
| R4 | All actor and judge identifiers are pinned to dated snapshots |
| R5 | Every run carries a matrix ID and a declared purpose |
| R6 | Every negative cell in any reported table receives a body sentence |
| R7 | The harness refuses to start on a dirty git tree |
| R8 | If a defect forces a change to the system under test, the campaign restarts |

A run is **valid** if and only if: the manifest is complete; all 500 questions completed; zero failures; zero token-budget fallbacks; the safety counter is recorded; and the package is sealed and dual-archived with matching checksums. Invalid runs were re-run in full. Partial results were never reported.

Nothing was reported anywhere, in any medium, from an unsealed package.

### Recorded deviations

Under R8, a change to the system under test voids the campaign. Under E5, a change confined to tooling outside the system under test is logged and the affected runs stand.

One deviation was recorded. G-C0 run 1 executed on commit `6f87bcb` rather than `25c6fca`. The difference between the two commits is the addition of two files under `docs/runbooks/`: a post-hoc analysis script and the document it produces. Neither is imported by the engine or the harness, and neither is on the execution path of any run. The system under test is byte-identical across the two commits. E5 applies; the run stands and retains its place in the arm.

---

## Environment

Single NVIDIA DGX Spark: GB10 Grace Blackwell, 128 GB unified memory, aarch64. Containers ARM64 only, digests pinned in every manifest.

| Component | Value |
|---|---|
| Runtime | Python 3.12 |
| Persistent store | PostgreSQL 16 with pgvector |
| Cache | Valkey 8 |
| Embedding model | BAAI `bge-large-en-v1.5`, 1,024 dimensions, served in-process |
| Retrieval | Dense cosine over pgvector, with MMR re-ranking over the fetched candidate set |

No model was fine-tuned. No agent-orchestration framework was used. API keys were supplied via environment variables and never written to any package file.

Hardware telemetry was sampled every 60 seconds. Energy was captured from the GB10 GPU rail via DCGM fields 155 and 156; `nvidia-smi power.draw` returns N/A on this platform. The Grace CPU rail is firmware-locked and unmeasurable, so reported energy undercounts total node energy. This is stated wherever energy appears.

This was an evaluation environment. It is not a production specification, and no production deployment of EDN exists.

---

## Explicit non-goals

The following were out of scope and were not run. We list them so that their absence is not mistaken for an omission.

- The provenance stress-test (a workload that deliberately introduces AI-generated candidates)
- Retrieval-layer diagnostics: Recall@k, mean reciprocal rank, gold-evidence coverage
- A second benchmark, such as LoCoMo
- LongMemEval-V2
- Any cross-system leaderboard comparison
- Any configuration improvement discovered mid-campaign, however tempting

The configuration was frozen. Discovery was not the purpose.
