---
name: ilisi-rnn-integration-metric-computation
description: Use when you have integrated (batch-corrected) single-cell expression
  or PCA-space embeddings from ≥2 batches and need quantitative iLISI/RNN scores plus
  runtime to compare integration methods.
when_to_use_negative:
- The current BATMAN implementation only integrates two datasets, so this skill cannot
  score a single multi-batch BATMAN run across more than two inputs
- Only uncorrected, concatenated count matrices are available and no integration has
  been performed yet — iLISI/RNN evaluate corrected output, not raw data
- The goal is de novo batch correction rather than benchmarking; use the integration
  skill itself instead of the metric-computation skill
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: BATMAN
  role: Integration tool whose batch-corrected output is scored with iLISI and k-RNN
    metrics
  repo: https://github.com/mandricigor/batman
- name: Harmony
  role: PCA-space integration baseline compared against BATMAN via iLISI and 10-RNN
    scores
- name: Scanorama
  role: MNN-based comparison tool scored with iLISI and 50-RNN in the LB scenario
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:10d5b1ed081341ceaab77997befa2386ec81d519d2492a561346bb28dd1763ae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - Harmony
  - Scanorama
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ilisi-rnn-integration-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies batch-effect removal and local-structure preservation after scRNA-seq integration by computing mean iLISI (inverted Local Inverse Simpson Index) and RNN (retained-neighbor-neighbor, e.g. 50-RNN or 10-RNN) scores for each integration tool. Used in the BATMAN paper to rank BATMAN against Seurat V3.0, MNNcorrect, Scanorama, and Harmony on simulated (Splatter) and real (Muraro/Segerstolpe pancreas, 10X PBMC 8k + Pan T cells) datasets.

## When to use

Use when you have integrated (batch-corrected) single-cell expression or PCA-space embeddings from ≥2 batches and need quantitative iLISI/RNN scores plus runtime to compare integration methods.

## When NOT to use

- The current BATMAN implementation only integrates two datasets, so this skill cannot score a single multi-batch BATMAN run across more than two inputs
- Only uncorrected, concatenated count matrices are available and no integration has been performed yet — iLISI/RNN evaluate corrected output, not raw data
- The goal is de novo batch correction rather than benchmarking; use the integration skill itself instead of the metric-computation skill

## Inputs

- Batch-corrected expression matrices or embeddings from ≥2 integrated scRNA-seq datasets (e.g. BATMAN or Harmony output)
- Batch/dataset label per cell (needed for iLISI)
- Pre-integration reference embeddings for RNN neighborhood retention
- Wall-clock runtime per tool

## Outputs

- Mean iLISI per tool (e.g. 1.84 for BATMAN, 1.82 for original data in the LB scenario)
- Mean k-RNN per tool (e.g. 50-RNN: 0.92 BATMAN, 0.14 Scanorama; 10-RNN for PCA-space comparisons)
- Metric and runtime comparison table such as batman_harmony_pbmc_comparison.csv

## How to apply

After preprocessing each dataset with Seurat V3.0 steps to obtain log-counts (and, for the 10X PBMC 8k vs Pan T cells comparison, projecting both datasets into a shared PCA space), run each integration tool — e.g. BATMAN, Harmony, Seurat V3.0, MNNcorrect, Scanorama — on the same inputs. Compute mean iLISI over cells to measure batch mixing (higher is better; 1.0 means no mixing) and mean k-RNN (e.g. 50-RNN or 10-RNN) to measure retention of local cell-to-cell neighborhoods (1.0 = perfect retention). Record wall-clock runtime per tool alongside the metrics (in the 10X PBMC PCA-space comparison both BATMAN and Harmony completed in ~14 s). Write the metric and runtime table (e.g. batman_harmony_pbmc_comparison.csv) and judge integration quality as a trade-off: BATMAN achieved mean iLISI 1.84 and 50-RNN 0.92 versus 1.82/1.00 for uncorrected data in the LB scenario, while Harmony showed slightly better iLISI and 10-RNN than BATMAN in PCA space.

## Related tools

- **BATMAN** (Integration tool whose batch-corrected output is scored with iLISI and k-RNN metrics) — https://github.com/mandricigor/batman
- **Harmony** (PCA-space integration baseline compared against BATMAN via iLISI and 10-RNN scores)
- **Scanorama** (MNN-based comparison tool scored with iLISI and 50-RNN in the LB scenario)

## Evaluation signals

- Uncorrected/original-data mean iLISI ≈ 1.0 per batch structure; successful integration in the LB scenario raises it to ≥1.82 (BATMAN: 1.84)
- Mean 50-RNN close to 1.0 indicates local structure retention (BATMAN 0.92 vs Scanorama 0.14); severe drops flag over-correction
- Cross-tool sanity check: in PCA space on 10X PBMC 8k + Pan T cells, Harmony should be slightly better than BATMAN on both iLISI and 10-RNN
- Runtimes comparable (~14 s per tool on the 10X PBMC PCA-space comparison) confirm the same embeddings and scale were used
- Metrics are aggregated as means over cells and reported alongside wall-clock runtime in a single CSV table

## Limitations

- Current BATMAN implementation supports integration of only two datasets; multi-dataset integration is not implemented (only possible 'in principle')
- The choice of k for RNN (50-RNN vs 10-RNN) changes scores across experiments and must be matched to the context being compared
- MNN-based state-of-the-art methods can score well on mixing metrics while failing to maintain local dataset structure, so iLISI and RNN should be reported together

## Evidence
