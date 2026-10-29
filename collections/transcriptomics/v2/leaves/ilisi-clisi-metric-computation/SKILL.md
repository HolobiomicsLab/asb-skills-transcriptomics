---
name: ilisi-clisi-metric-computation
description: Use when two preprocessed scRNA-seq datasets have been integrated (e.g.,
  by BATMAN, Seurat V3.0, MNNcorrect, or Scanorama) and you need quantitative batch-mixing
  (iLISI) and cell-type-consistency (cLISI) scores to compare correction methods.
when_to_use_negative:
- Integrating more than two datasets — the BATMAN implementation whose outputs these
  metrics benchmark supports only two datasets per run
- Single-dataset input with no batch structure to integrate — iLISI is undefined without
  batch labels
- Unintegrated, raw concatenated data being treated as corrected output — the paper
  keeps an explicit 'Original' (no-correction) baseline rather than scoring raw counts
  as if corrected
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Integration method whose corrected output is scored with mean iLISI/cLISI;
    the integration itself uses a minimum-weight bipartite matching of anchors with
    Euclidean-distance edge weights and local shifting vectors for batch correction
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Provides the pre-processing to log-counts that all compared methods receive,
    and is itself one of the comparison methods scored by iLISI/cLISI
- name: Scanorama
  role: Comparison integration method scored by mean iLISI/cLISI (e.g., mean iLISI
    1.01 and worst-tier iLISI in the 1-held-out experiment)
- name: MNNcorrect
  role: Comparison integration method scored by mean iLISI/cLISI (e.g., mean iLISI
    1.00 in the LB scenario and worst-tier iLISI in the 1-held-out experiment)
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:e8a14ee611956454926ddad0a81577df1736b0c09b9000a44e63c4356724be9e
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
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ilisi-clisi-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computation of mean iLISI (integration Local Inverse Simpson Index) and mean cLISI on integrated single-cell RNA-seq data to quantify batch mixing versus cell-type purity, as used to benchmark BATMAN against Seurat V3.0, MNNcorrect, and Scanorama.

## When to use

Use when two preprocessed scRNA-seq datasets have been integrated (e.g., by BATMAN, Seurat V3.0, MNNcorrect, or Scanorama) and you need quantitative batch-mixing (iLISI) and cell-type-consistency (cLISI) scores to compare correction methods.

## When NOT to use

- Integrating more than two datasets — the BATMAN implementation whose outputs these metrics benchmark supports only two datasets per run
- Single-dataset input with no batch structure to integrate — iLISI is undefined without batch labels
- Unintegrated, raw concatenated data being treated as corrected output — the paper keeps an explicit 'Original' (no-correction) baseline rather than scoring raw counts as if corrected

## Inputs

- Integrated log-count matrix or embedding from two scRNA-seq datasets (post Seurat V3.0 pre-processing)
- Batch/dataset labels for each cell (e.g., Muraro CEL-Seq2 vs Segerstolpe Smart-Seq2, or Original vs corrected)
- Cell-type annotations (e.g., pancreatic alpha, beta, ductal cells; or simulated Splatter cell types) for cLISI

## Outputs

- Mean iLISI per integration method (batch-mixing score)
- Mean cLISI per integration method (cell-type consistency score)
- Comparison table (e.g., lb_integration_metrics.csv or pancreas_1heldout_results.csv) with iLISI, cLISI, and runtime per method

## How to apply

First apply Seurat V3.0 pre-processing to obtain log-counts for each dataset, run the integration method (e.g., BATMAN's minimum-weight matching with exact blossom or greedy ratio-2 approximation, or the comparison tools), then compute mean iLISI and cLISI on the integrated data using batch labels and cell-type annotations. Mean iLISI near the number of batches (2 for two-dataset experiments) indicates good batch mixing, while values near 1 indicate no mixing; cLISI near 1 indicates cell types remain locally consistent. In the paper's large-batch-effects (LB) simulation scenario, the reference values were mean iLISI 1.82/1.84/1.00/1.00/1.01 and mean 50-RNN 1.00/0.92/0.58/0.51/0.14 for Original/BATMAN/Seurat V3.0/MNNcorrect/Scanorama. In the 1-held-out pancreas condition (alpha cells, 37% of CEL-Seq2 cells, excluded from Muraro CEL-Seq2 versus Segerstolpe Smart-Seq2), BATMAN achieved the best iLISI (1.39) while MNNcorrect and Scanorama scored worst. Record the metrics per method in a comparison table such as lb_integration_metrics.csv or pancreas_1heldout_results.csv.

## Related tools

- **BATMAN** (Integration method whose corrected output is scored with mean iLISI/cLISI; the integration itself uses a minimum-weight bipartite matching of anchors with Euclidean-distance edge weights and local shifting vectors for batch correction) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Provides the pre-processing to log-counts that all compared methods receive, and is itself one of the comparison methods scored by iLISI/cLISI)
- **Scanorama** (Comparison integration method scored by mean iLISI/cLISI (e.g., mean iLISI 1.01 and worst-tier iLISI in the 1-held-out experiment))
- **MNNcorrect** (Comparison integration method scored by mean iLISI/cLISI (e.g., mean iLISI 1.00 in the LB scenario and worst-tier iLISI in the 1-held-out experiment))

## Evaluation signals

- In the LB simulation scenario, reproduced values should be mean iLISI 1.82/1.84/1.00/1.00/1.01 and mean 50-RNN 1.00/0.92/0.58/0.51/0.14 for Original/BATMAN/Seurat V3.0/MNNcorrect/Scanorama (within reasonable simulation variance)
- Mean iLISI close to 1 signals no batch mixing; a corrected method should improve over or match the Original baseline (e.g., BATMAN 1.84 vs Original 1.82 in the LB scenario)
- In the 1-held-out pancreas condition, BATMAN should achieve the best iLISI (1.39), with MNNcorrect and Scanorama producing the worst iLISI scores
- All methods should leave alpha cells from the Smart-Seq2 dataset unmatched in the 1-held-out condition — a qualitative check accompanying the metric
- cLISI should stay near 1 for a correct integration so that batch mixing does not come at the cost of cell-type purity

## Limitations

- iLISI/cLISI do not capture local structure preservation; the paper complements them with mean 50-RNN, where Scanorama scored 0.14 versus BATMAN's 0.92 despite similar iLISI behavior
- Metric comparisons depend on the number of batches; with two datasets iLISI ranges from 1 (no mixing) to 2 (perfect mixing), so values are not directly comparable across experiments with different batch counts
- The comparison is currently restricted to two-dataset integrations, and simulated benchmarks (Splatter, 100 datasets per scenario, 1,000 genes) carry simulation variance

## Evidence
