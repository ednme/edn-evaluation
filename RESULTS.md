# CONFIRM-1 Results

All values are computed from the fifteen sealed run packages. Overall accuracy is read from each run's `stats.json`; per-question outcomes from `judgements.jsonl`.

Status: **final**. Sealed June 2026 (AEST).

Benchmark: LongMemEval-S, 500 questions across six categories.
Metric: answer accuracy, the fraction of the 500 questions judged correct by GPT-4o (`gpt-4o-2024-08-06`).
Replication: each configuration ran three times.

---

## Arms

Each arm changes exactly one thing relative to the control, so its difference isolates that component.

| Arm | Actor | Configuration |
|---|---|---|
| H-C0 | Claude Haiku 4.5 | Control: full EDN memory, all components enabled |
| H-A1 | Claude Haiku 4.5 | Round-level records disabled (turn-level storage) |
| H-A2 | Claude Haiku 4.5 | MMR diversity re-ranking disabled (cosine only) |
| H-B0 | Claude Haiku 4.5 | No EDN layer: full conversation history in context |
| G-C0 | GPT-4o | Control: full EDN memory, all components enabled |

`H-` denotes a Haiku actor, `G-` a GPT-4o actor. `C0` is a control, `A` an ablation, `B0` the baseline.

---

## 1. Overall accuracy

Mean, sample standard deviation (n-1), and t-based 95% confidence interval (df=2, t=4.303) across the three runs of each arm.

| Arm | Description | Mean | SD | 95% CI |
|---|---|---|---|---|
| H-C0 | Haiku control | 0.854 | 0.0087 | [0.832, 0.876] |
| H-A1 | Haiku, ROUND off | 0.629 | 0.0061 | [0.614, 0.644] |
| H-A2 | Haiku, MMR off | 0.847 | 0.0046 | [0.835, 0.858] |
| H-B0 | Haiku, no memory | 0.455 | 0.0081 | [0.435, 0.475] |
| G-C0 | GPT-4o control | 0.814 | 0.0131 | [0.781, 0.847] |

---

## 2. Component contributions

Delta is the arm mean minus the control mean (H-C0). McNemar's exact test is computed on the per-question consensus outcome (best of each arm's three runs) over the 500 shared questions. `b` counts questions the control answers correctly and the arm does not; `c` the reverse.

| Comparison | Component removed | Delta | b | c | p |
|---|---|---|---|---|---|
| H-C0 vs H-B0 | All EDN memory | -39.9 pp | 225 | 16 | 2.27 × 10<sup>-48</sup> |
| H-C0 vs H-A1 | ROUND records | -22.5 pp | 129 | 12 | 6.29 × 10<sup>-26</sup> |
| H-C0 vs H-A2 | MMR re-ranking | -0.7 pp | 13 | 8 | 0.383 (n.s.) |

The ordering is unambiguous and the two large effects are decisive. Diversity re-ranking is retained for its behaviour on redundant candidate sets and because it re-ranks an already-fetched set at negligible latency cost, but its measured accuracy contribution at n=3 is not statistically distinguishable from zero. We report that plainly rather than round it up.

---

## 3. Per-category accuracy

Six LongMemEval-S categories. `n` is the number of questions in that category. `d` is the arm minus the control.

| Category | n | H-C0 | H-A1 (d) | H-A2 (d) | H-B0 (d) | G-C0 (d) |
|---|---|---|---|---|---|---|
| single-session-user | 70 | 0.957 | 0.819 (-0.138) | 0.952 (-0.005) | 0.576 (-0.381) | 0.962 (+0.005) |
| single-session-assistant | 56 | 0.869 | 0.411 (-0.458) | 0.869 (0.000) | 0.845 (-0.024) | 0.881 (+0.012) |
| single-session-preference | 30 | 0.833 | 0.778 (-0.056) | 0.833 (0.000) | 0.178 (-0.655) | 0.656 (-0.178) |
| knowledge-update | 78 | 0.932 | 0.722 (-0.209) | 0.936 (+0.004) | 0.680 (-0.252) | 0.846 (-0.085) |
| temporal-reasoning | 133 | 0.822 | 0.614 (-0.208) | 0.795 (-0.028) | 0.303 (-0.519) | 0.764 (-0.058) |
| multi-session | 133 | 0.785 | 0.546 (-0.238) | 0.785 (0.000) | 0.308 (-0.477) | 0.774 (-0.010) |

Notes on the negative cells, as required by the campaign's reporting rule.

**Single-session-assistant collapses under ROUND-off** (-45.8 pp), by far the largest category effect in the campaign. This is the expected signature: these questions require recalling what the assistant said, and turn-level storage separates the assistant's reply from the user turn that prompted it. Round-level records keep the exchange whole.

**Single-session-assistant is nearly unaffected by removing the memory layer** (-2.4 pp), the only category where this holds. The assistant's own prior statements survive in the raw transcript, so a full-history baseline retains them even without governed memory. It is the categories requiring composition across sessions that collapse.

**Single-session-preference falls to 0.178 without the memory layer** (-65.5 pp), the largest single drop anywhere in the table. Stated preferences are exactly the content that a long undifferentiated transcript buries.

**GPT-4o trails Haiku on single-session-preference** by 17.8 pp. This category has 30 questions, the smallest in the benchmark; a small number of judged items moves the score substantially. We treat overall accuracy as the stable signal and do not read a cross-actor effect into this cell.

**GPT-4o trails on knowledge-update** (-8.5 pp) and **temporal-reasoning** (-5.8 pp). Absolute numbers remain actor-dependent; the architecture effect does not.

**H-A2 shows small negative deltas** on single-session-user (-0.5 pp) and temporal-reasoning (-2.8 pp), against a positive cell on knowledge-update (+0.4 pp). These are within run-to-run variance and are consistent with the non-significant overall MMR result.

---

## 4. Secondary endpoint: single-session-preference under ROUND

Pre-registered as endpoint S1, reported regardless of direction.

H-C0 = 0.833, H-A1 = 0.778, delta = -5.6 pp over the 30 questions. McNemar: b=4, c=1, p=0.375.

The direction matches an earlier single-run observation, but the effect does not reach significance on 30 questions and we do not claim a replicated trade-off. The preference retrieval pathway remains enabled in the full configuration.

---

## 5. Run-to-run stability

Percentage of the 500 questions whose judged outcome is identical across all three runs of an arm.

| Arm | Stability |
|---|---|
| H-C0 | 91.6% |
| H-A1 | 91.4% |
| H-A2 | 89.6% |
| H-B0 | 79.6% |
| G-C0 | 83.6% |

Actors run at provider-default temperature, so runs are stochastic and dispersion is the reported uncertainty. The no-memory baseline is the least stable arm, which is itself informative: when the model must find the answer somewhere in a long undifferentiated context, whether it succeeds varies more between attempts.

---

## 6. Run-by-run overall accuracy

| Arm | Run | Run ID | Overall |
|---|---|---|---|
| H-C0 | r1 | `01KV7NZNDKZHA9XTZH61T394WH` | 0.848 |
| H-C0 | r2 | `01KV7WGMVG7NDX9XJERQZGHHV7` | 0.864 |
| H-C0 | r3 | `01KV820SYMSP7CW86SK5MY9GQ0` | 0.850 |
| H-A1 | r1 | `01KV880SFFQ65H43Q7GDKRD9H7` | 0.630 |
| H-A1 | r2 | `01KV8EPCYXRWBEWS93WP1PSZWA` | 0.622 |
| H-A1 | r3 | `01KV8N8P1A69ATA97TXFRHF8PF` | 0.634 |
| H-A2 | r1 | `01KV8W8TMPX65YPA7B5YDW13HE` | 0.844 |
| H-A2 | r2 | `01KV91JWJ5YR11XNCQD6RDH7FD` | 0.852 |
| H-A2 | r3 | `01KV971BZQHQ7DP2N60SCQ7V1Y` | 0.844 |
| H-B0 | r1 | `01KVC2KF0E1NBGP0KKEA55YP2C` | 0.456 |
| H-B0 | r2 | `01KVC5AE06F251H10Y5AMM955J` | 0.446 |
| H-B0 | r3 | `01KVC8K8BKH1M7538YQ3EY5772` | 0.462 |
| G-C0 | r1 | `01KVA719KXCE0B8RJTDEJ9DMK5` | 0.828 |
| G-C0 | r2 | `01KV9GEFT6J15XC5JQAKKA4BFJ` | 0.802 |
| G-C0 | r3 | `01KV9NVQ8K4ZGBDGG81NZ58H33` | 0.812 |

---

## 7. Context cost

Mean per run across the 500 questions.

| Arm | Tokens in | Tokens out |
|---|---|---|
| H-C0 | 5,256,905 | 116,414 |
| H-A1 | 911,521 | 113,173 |
| H-A2 | 5,256,905 | 115,899 |
| H-B0 | 51,030,653 | 74,328 |
| G-C0 | 4,694,903 | 52,660 |

The governed configuration injects roughly an order of magnitude less context than the full-history baseline, at 39.9 points higher accuracy. H-A1 consumes less still, because turn-level records are smaller, and pays 22.5 points for it.

---

## 8. Systems behaviour

**Safety.** The provenance safety counter, recorded in each run's `stats.json`, is **0** across all fifteen runs, covering 7,500 constructed packages. Zero AI-generated records were admitted to factual context.

**Integrity.** Every run completed all 500 questions. Zero failures, zero token-budget fallbacks.

**Latency.** Package assembly p95 is 30 to 35 ms on the solo-flagged memory runs, against a pre-registered deployability target of 80 ms. H-B0 builds no package; its per-question p95 is 22.7 to 23.1 ms.

**Energy.** Measured on the GB10 GPU rail via DCGM fields 155 and 156. The Grace CPU rail is firmware-locked and unmeasurable, so these figures **undercount total node energy**. `nvidia-smi power.draw` returns N/A on this platform and cannot be used.

| Arm | Energy per run | Mean power |
|---|---|---|
| H-C0 | 100.3 Wh | 63.5 W |
| H-A1 | 123.1 Wh | 66.9 W |
| H-A2 | 90.1 Wh | 63.6 W |
| H-B0 | not captured | 11.3 W |
| G-C0 | 88.4 Wh | 66.6 W |

H-B0 watt-hours were not captured; mean power only. Its low power draw reflects that the actor call dominates and no local retrieval work is performed.

---

## Reading these numbers

Every figure above is recomputable from the sealed packages under the unchanged upstream scoring script. Nothing here was hand-transcribed from a log.

Three runs per arm is enough to fix the ordering of effects. It is not enough to support fine distinctions between adjacent per-category cells, and we have avoided drawing any.
