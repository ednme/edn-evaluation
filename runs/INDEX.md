# Run index

Fifteen sealed run packages. Directory names are the run ULIDs assigned at execution, exactly as sealed. Each directory verifies against its own `SHA256SUMS.txt`.

| Run ID | Arm | Run | Commit | Overall |
|---|---|---|---|---|
| `01KV7NZNDKZHA9XTZH61T394WH` | H-C0 | 1 | `25c6fca` | 0.848 |
| `01KV7WGMVG7NDX9XJERQZGHHV7` | H-C0 | 2 | `25c6fca` | 0.864 |
| `01KV820SYMSP7CW86SK5MY9GQ0` | H-C0 | 3 | `25c6fca` | 0.850 |
| `01KV880SFFQ65H43Q7GDKRD9H7` | H-A1 | 1 | `25c6fca` | 0.630 |
| `01KV8EPCYXRWBEWS93WP1PSZWA` | H-A1 | 2 | `25c6fca` | 0.622 |
| `01KV8N8P1A69ATA97TXFRHF8PF` | H-A1 | 3 | `25c6fca` | 0.634 |
| `01KV8W8TMPX65YPA7B5YDW13HE` | H-A2 | 1 | `25c6fca` | 0.844 |
| `01KV91JWJ5YR11XNCQD6RDH7FD` | H-A2 | 2 | `25c6fca` | 0.852 |
| `01KV971BZQHQ7DP2N60SCQ7V1Y` | H-A2 | 3 | `25c6fca` | 0.844 |
| `01KVC2KF0E1NBGP0KKEA55YP2C` | H-B0 | 1 | `63d5c4e` | 0.456 |
| `01KVC5AE06F251H10Y5AMM955J` | H-B0 | 2 | `63d5c4e` | 0.446 |
| `01KVC8K8BKH1M7538YQ3EY5772` | H-B0 | 3 | `63d5c4e` | 0.462 |
| `01KVA719KXCE0B8RJTDEJ9DMK5` | G-C0 | 1 | `6f87bcb` | 0.828 |
| `01KV9GEFT6J15XC5JQAKKA4BFJ` | G-C0 | 2 | `25c6fca` | 0.802 |
| `01KV9NVQ8K4ZGBDGG81NZ58H33` | G-C0 | 3 | `25c6fca` | 0.812 |

On the G-C0 run 1 commit, see the deviation note in [METHODOLOGY.md](../METHODOLOGY.md#recorded-deviations).

## Verifying

From any run directory:

```
sha256sum -c SHA256SUMS.txt
```

All fifteen verify. No file inside any package has been added, removed, renamed, or modified for publication.

## Package contents

| File | Contents |
|---|---|
| `manifest.json` | Pre-run configuration, written and validated before question 1 |
| `questions.jsonl` | Per question: qid, prompt hash, package hash, retrieval candidate ids, response, latency, tokens, retry events |
| `hyp.jsonl` | Model hypotheses in upstream LongMemEval format |
| `hyp.jsonl.eval-results-gpt-4o` | Upstream scoring script output |
| `judgements.jsonl` | Per-question judge label |
| `progress.jsonl` | Per-question execution progress |
| `telemetry.jsonl` | 60-second hardware samples |
| `stats.json` | Overall, per-category, p95, failures, fallbacks, safety counter, tokens, ingest, energy |
| `pip_freeze.txt` | Exact Python environment for the run |
| `SHA256SUMS.txt` | Checksums over every file above |

`questions.jsonl` records a hash of the prompt and a hash of the rendered session-initialisation package, not their text. Retrieval candidates are opaque identifiers. The H-B0 baseline builds no package, so its `sip_sha256` is null.
