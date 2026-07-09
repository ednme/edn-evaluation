# EDN CONFIRM-1: June 2026 Evaluation Results

Reproduction artefacts and sealed results for the confirmation campaign reported in *EDN - Episodic Dialogue Network: Provenance-Governed External Memory Middleware for Multi-Session LLM Assistants*.

This repository contains **evaluation artefacts only**. It does not contain EDN source code.

---

## What EDN is

EDN is deterministic memory middleware that sits between an application and an unmodified actor model. It classifies interaction history into typed records under a seven-class provenance model, governs which records are admissible as fact, retrieves under a fixed policy, re-ranks for diversity, and renders a bounded package at the front of the actor prompt.

It targets two failure modes that grow with conversation history:

- **Loss.** Models attend to long contexts unevenly and accuracy degrades as input grows, even before the window is full.
- **Fabrication.** Without a record of origin, an unsupported statement can be retrieved in a later session and reused as established fact.

---

## Headline results

LongMemEval-S, 500 questions, three independent runs per configuration, GPT-4o (`gpt-4o-2024-08-06`) as judge.

| Configuration | Actor | Mean | SD | 95% CI |
|---|---|---|---|---|
| Full EDN | Claude Haiku 4.5 | **85.4%** | 0.9 | [83.2, 87.6] |
| Full EDN | GPT-4o | **81.4%** | 1.3 | [78.1, 84.7] |

Component contributions, primary actor, exact paired McNemar on per-question outcomes:

| Component removed | Delta | b / c | p |
|---|---|---|---|
| EDN memory layer (full history in context instead) | **-39.9 pp** | 225 / 16 | < 10<sup>-47</sup> |
| Round-level representation (reverted to turn-level) | **-22.5 pp** | 129 / 12 | < 10<sup>-25</sup> |
| Diversity re-ranking (MMR) | -0.7 pp | 13 / 8 | 0.38 (n.s.) |

Three findings, stated plainly:

1. **Governing memory beats having all of it.** The no-EDN baseline is not an empty prompt. It hands the actor the entire conversation history in context. Structuring and governing that same information is worth 39.9 points over it.

2. **The unit of storage was the largest architectural lever.** Storing the complete user-assistant exchange as one retrievable record, rather than as separate turns, is worth 22.5 points. Diversity re-ranking, by contrast, is not statistically distinguishable from zero at this replication count.

3. **The provenance rule is a verified structural invariant, not a measured accuracy lever.** Zero AI-generated records entered factual context across 7,500 constructed packages. The benchmark does not emit AI-generated records through the ingest path, so the provenance ablation returned a null accuracy result *by construction*. We report it as an invariant and make no empirical safety claim from this campaign. See [Limitations](#limitations).

Full tables, per-category breakdowns, run-by-run values, and systems metrics: **[RESULTS.md](RESULTS.md)**.

---

## Reproducibility

### Commits

The EDN arms ran on a frozen commit. The no-memory baseline arm carries its own commit, because rendering full conversation history into the actor context is a harness path that the EDN arms do not exercise.

| Arms | Commit |
|---|---|
| H-C0, H-A1, H-A2, G-C0 | `25c6fca` |
| H-B0 (no-memory baseline) | `63d5c4e` |

One exception, recorded for completeness: G-C0 run 1 (`01KVA719KXCE0B8RJTDEJ9DMK5`) executed on `6f87bcb`, which is `25c6fca` plus two files added under `docs/runbooks/` — a post-hoc analysis script and its output. Neither is on the execution path and the system under test is byte-identical between the two commits. The run stands, and the deviation is logged here rather than reconciled away.

Each run's manifest records the commit it executed against.

### Sealed packages

Under the campaign's evidence rule, *a run without a sealed, checksummed package is not evidence*. Every counted run produced a package containing:

```
manifest.json                    pre-run configuration, written and validated before question 1
questions.jsonl                  per-question record: qid, prompt hash, package hash,
                                 retrieval candidate ids, response, latency, tokens, retries
hyp.jsonl                        model hypotheses in upstream LongMemEval format
hyp.jsonl.eval-results-gpt-4o    upstream scoring script output
judgements.jsonl                 per-question judge label
progress.jsonl                   per-question execution progress
telemetry.jsonl                  60-second hardware samples
stats.json                       overall, per-category, p95, failures, fallbacks,
                                 safety counter, tokens, ingest, energy
pip_freeze.txt                   exact Python environment for the run
SHA256SUMS.txt                   checksums over every file in the package
```

Packages are published exactly as sealed. No file has been added, removed, renamed, or rewritten, so `sha256sum -c SHA256SUMS.txt` verifies in every run directory.

The provenance safety counter is reported in `stats.json` for each run.

Fifteen packages, one per counted run. All numbers in the paper and in `RESULTS.md` are recomputable from these packages alone, under the unchanged upstream LongMemEval scoring script.

### Run identifiers

| Arm | Run IDs |
|---|---|
| H-C0 | `01KV7NZNDKZHA9XTZH61T394WH`, `01KV7WGMVG7NDX9XJERQZGHHV7`, `01KV820SYMSP7CW86SK5MY9GQ0` |
| H-A1 | `01KV880SFFQ65H43Q7GDKRD9H7`, `01KV8EPCYXRWBEWS93WP1PSZWA`, `01KV8N8P1A69ATA97TXFRHF8PF` |
| H-A2 | `01KV8W8TMPX65YPA7B5YDW13HE`, `01KV91JWJ5YR11XNCQD6RDH7FD`, `01KV971BZQHQ7DP2N60SCQ7V1Y` |
| H-B0 | `01KVC2KF0E1NBGP0KKEA55YP2C`, `01KVC5AE06F251H10Y5AMM955J`, `01KVC8K8BKH1M7538YQ3EY5772` |
| G-C0 | `01KVA719KXCE0B8RJTDEJ9DMK5`, `01KV9GEFT6J15XC5JQAKKA4BFJ`, `01KV9NVQ8K4ZGBDGG81NZ58H33` |

---

## What is in this repository

```
README.md            this file
RESULTS.md           complete results: all arms, all categories, all runs
METHODOLOGY.md       benchmark, actors, judge, measurement definitions, pre-registration
CITATION.cff         citation metadata
runs/                fifteen sealed run packages
```

## What is not in this repository

EDN itself is not published. Specifically excluded:

- EDN source code
- The SIP renderer
- Prompt templates
- Any User Calibration Engine code

A provisional patent application covering the EDN Memory Engine was filed on 4 June 2026. This repository ships the evidence for the paper's claims, not the system that produced it. The sealed packages contain per-question responses, judge outputs, and configuration manifests; they do not contain the rendered packages or the prompts used to produce them.

---

## Benchmark and licensing

The benchmark is **LongMemEval-S** (`longmemeval_s_cleaned.json`), obtained from the authors' Hugging Face mirror `xiaowu0162/longmemeval-cleaned` and used unmodified under the MIT licence of the upstream LongMemEval project. Its SHA256 is recorded in every manifest. Scoring uses the upstream evaluation script, unchanged.

We are not submitting to a leaderboard and produce no cross-system comparison. Published memory systems use different actors, harnesses, and judges; a direct ranking would be methodologically unstable.

---

## Environment

All runs executed on a single NVIDIA DGX Spark (GB10 Grace Blackwell, 128 GB unified memory, aarch64). Backing services were PostgreSQL 16 with pgvector and Valkey 8, both ARM64 containers with digests pinned in each manifest. Embeddings were BAAI `bge-large-en-v1.5` at 1,024 dimensions, served in-process.

No model was fine-tuned. No agent-orchestration framework was used.

This was an evaluation environment, not a production deployment. No production deployment of EDN exists.

---

## Limitations

**Three runs per arm.** Sufficient to establish that the overall signal and the component ordering are real and stable. Not sufficient for high-precision claims about narrow sub-categories, particularly single-session-preference at 30 questions, where a handful of judged items moves the score several points.

**One benchmark.** EDN is validated on LongMemEval-S only. Generalisation requires a second benchmark. Newer memory benchmarks have appeared since this campaign was frozen; we have not run them, and we make no claim on LongMemEval-V2, which targets a different task on different data.

**The provenance rule is unexercised.** The fabrication failure mode that motivates the provenance gate is precisely the case this benchmark does not produce. The rule is verified as a structural invariant and nothing more. A dedicated stress-test that deliberately introduces AI-generated candidates, and reports leak rate, false-exclusion rate, and downstream contamination, is the single highest-value addition to this work. It has not been run.

**End-to-end accuracy only.** We report no retrieval-layer diagnostics (Recall@k, MRR, gold-evidence coverage). These are needed to localise the gains within the read path.

**A single LLM judge.** Several per-category margins are smaller than run-to-run variance.

**Provenance is not truth.** The gate records where a statement came from. It does not establish that the statement is correct. A user-stated fact can be wrong. EDN stores typed records, including verbatim exchanges, and therefore carries obligations a stateless prompt pipeline does not: memory poisoning, false confirmation, transcript recall, and retention.

---

## Citation

See [CITATION.cff](CITATION.cff). The paper link will be added on publication.

---

## Contact

contact@tensorpro.ai | TensorPRO, Brisbane, Australia.

© 2026 TensorPRO
