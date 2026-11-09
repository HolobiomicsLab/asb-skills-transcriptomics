---
name: batch-effect-quantification-ilisi
description: Use when comparing batch-correction methods (e.g., BATMAN, Seurat V3.0,
  MNNcorrect, Scanorama) on integrated scRNA-seq datasets and you need a numeric mixing-vs-structure
  tradeoff score such as mean iLISI and mean 50-RNN.
when_to_use_negative:
- Integrating more than two datasets with BATMAN itself — the current implementation
  works only with two datasets (the metric can still be computed, but the correction
  step cannot)
- Data that has not been batch-integrated or for which batch labels are unavailable,
  since iLISI depends on batch composition of neighborhoods
- Assessing integration of datasets of very different library chemistries where iLISI
  mixing may be misleading without complementary structure metrics (pair with 50-RNN)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Primary batch-correction method whose integration output is scored with mean
    iLISI and mean 50-RNN; achieves the reference LB-scenario scores
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Provides the pre-processing (log-counts) fed to all integrators and serves
    as a comparison integrator benchmarked with iLISI
- name: Scanorama
  role: Comparison integrator scored with iLISI/50-RNN (mean iLISI 1.01, mean 50-RNN
    0.14 in the LB scenario)
- name: MNNcorrect
  role: Comparison integrator scored with iLISI/50-RNN (mean iLISI 1.00, mean 50-RNN
    0.51 in the LB scenario)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:3d097eea9f7971bbba99aac35c78c1b0669c0fcb84c5d1b9cad5263341c5254b
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
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# batch-effect-quantification-ilisi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify batch-effect removal quality of single-cell RNA-seq integration by computing mean iLISI (integration Local Inverse Simpson Index) alongside mean 50-RNN (retained neighbor relationships), as benchmarked in the BATMAN paper on 10X Genomics PBMC 8k vs Pan T cells.

## When to use

Use when comparing batch-correction methods (e.g., BATMAN, Seurat V3.0, MNNcorrect, Scanorama) on integrated scRNA-seq datasets and you need a numeric mixing-vs-structure tradeoff score such as mean iLISI and mean 50-RNN.

## When NOT to use

- Integrating more than two datasets with BATMAN itself — the current implementation works only with two datasets (the metric can still be computed, but the correction step cannot)
- Data that has not been batch-integrated or for which batch labels are unavailable, since iLISI depends on batch composition of neighborhoods
- Assessing integration of datasets of very different library chemistries where iLISI mixing may be misleading without complementary structure metrics (pair with 50-RNN)

## Inputs

- Integrated expression matrix from each batch-correction method (applied to Seurat V3.0 log-counts of two datasets, e.g., 10X Genomics PBMC 8k and Pan T cells)
- Batch/cell-type label vector per cell for computing iLISI
- Original (uncorrected) concatenated matrices as the baseline row

## Outputs

- CSV with one row per method (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) reporting mean iLISI, mean 50-RNN, and runtime
- Per-method mean iLISI and mean 50-RNN scalar scores

## How to apply

After running each integrator (BATMAN, Seurat V3.0, MNNcorrect, Scanorama) on the same Seurat V3.0 pre-processed log-count matrices, compute mean iLISI to measure how well batches mix and mean 50-RNN (retained relationships within 50 nearest neighbors) to measure local-structure preservation. In the LB scenario on 10X Genomics PBMC 8k (8,381 cells) vs Pan T cells (3,555 T cells), expect BATMAN to reach mean iLISI 1.84 (vs 1.00 Seurat V3.0, 1.00 MNNcorrect, 1.01 Scanorama, 1.82 original) with mean 50-RNN 0.92 (vs 0.58, 0.51, 0.14). Record each method's wall-clock runtime to capture the accuracy-speed tradeoff (BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s). Write one CSV row per method (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) with mean iLISI, mean 50-RNN, and runtime.

## Related tools

- **BATMAN** (Primary batch-correction method whose integration output is scored with mean iLISI and mean 50-RNN; achieves the reference LB-scenario scores) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Provides the pre-processing (log-counts) fed to all integrators and serves as a comparison integrator benchmarked with iLISI)
- **Scanorama** (Comparison integrator scored with iLISI/50-RNN (mean iLISI 1.01, mean 50-RNN 0.14 in the LB scenario))
- **MNNcorrect** (Comparison integrator scored with iLISI/50-RNN (mean iLISI 1.00, mean 50-RNN 0.51 in the LB scenario))

## Examples

```
python3.7 batman.py -data pbmc8k_logcounts.csv,panT_logcounts.csv -output batman_integrated.csv -anchors 100
```

## Evaluation signals

- Reproduce the LB-scenario reference values: mean iLISI 1.84 and mean 50-RNN 0.92 for BATMAN versus ≤1.01 iLISI for Seurat V3.0, MNNcorrect, and Scanorama
- Check the runtime ordering: BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s
- CSV must contain exactly 5 rows (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) with a BATMAN row
- Original-data baseline mean iLISI should be ~1.82, confirming the metric pipeline is calibrated before scoring corrected outputs
- A method should not be credited as superior unless it raises iLISI above baseline while keeping 50-RNN high (BATMAN: 0.92), i.e., mixing must not come at the cost of cell-to-cell relationships

## Limitations

- BATMAN's current implementation integrates only two datasets; multi-dataset integration is not implemented (only possible 'in principle')
- Greedy matching is an approximation with ratio 2 relative to the exact blossom-based minimum-weight matching; use the exact algorithm for accuracy on smaller data and greedy for extra-large datasets
- iLISI alone rewards batch mixing; a method can inflate iLISI while destroying local structure (e.g., Scanorama's mean 50-RNN of 0.14), so pair it with 50-RNN

## Evidence

- [results] In the LB scenario, mean iLISI was 1.84 for BATMAN, 1.00 for Seurat V3.0, 1.00 for MNNcorrect, and 1.01 for Scanorama, versus 1.82 for the original data: "MetricOriginalBATMANSeurat V3.0MNNcorrectScanoramaMean iLISI1.821.841.001.001.01"
- [results] In the LB scenario, mean 50-RNN was 0.92 for BATMAN, 0.58 for Seurat V3.0, 0.51 for MNNcorrect, and 0.14 for Scanorama: "Mean 50-RNN1.000.920.580.510.14"
- [results] In all the experiments, the authors used Seurat V3.0 pre-processing to obtain log-counts for each dataset before integration and scoring: "In all the experiments, we used Seurat V3.0 pre-processing steps to obtain log-counts for each dataset."
- [readme] The README documents that the greedy matching flag is recommended for extra large datasets, an accuracy/runtime decision point when generating integration outputs to score: "-greedy               use greedy heuristic for the minimum weight matching
                        (recommended for extra large datasets)"
- [methods] The BATMAN matching procedure builds a weighted bipartite graph whose edge weights are Euclidean distances between cells, underlying the corrected outputs evaluated by iLISI: "each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space)"
