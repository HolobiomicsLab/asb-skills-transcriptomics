---
name: krnn-nearest-neighbor-retention-metric-computation
description: 'Use when evaluating scRNA-seq integration quality beyond batch mixing:
  you have matched pre- and post-integration log-count matrices and need to quantify
  how many of each cell''s original top-k neighbors survive correction.'
when_to_use_negative:
- You only have a single dataset or a single corrected output with no matched pre-integration
  matrices, so the before/after neighbor comparison cannot be performed
- You want a batch-mixing score (e.g., iLISI) rather than a local-structure-preservation
  score; k-RNN rewards retention of within-dataset neighborhoods and says nothing
  about cross-dataset mixing by itself
- The corrected output is a low-dimensional embedding rather than an expression matrix
  in the original gene space, so Euclidean distances are not comparable to the pre-integration
  gene-space neighborhoods
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Integration method whose corrected outputs are evaluated with the k-RNN metric;
    also the tool producing the highest mean 50-RNN (0.92) in the paper's LB scenario
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Provides pre-processing steps to obtain log-counts for each dataset; its integration
    output is one of the methods scored with 50-RNN (0.58)
- name: MNNcorrect
  role: MNN-based integration method evaluated with the k-RNN metric (mean 50-RNN
    0.51)
- name: Scanorama
  role: MNN-based integration method evaluated with the k-RNN metric (mean 50-RNN
    0.14)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:01b2e7d6559c5a2b59e904ea452da5e5346013fe2aad21fa2a9b1dbd2c35790f
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
  - BERMUDA
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# krnn-nearest-neighbor-retention-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the k-RNN (retained nearest neighbors) score, a local-structure-preservation metric that quantifies what fraction of each cell's pre-integration top-k nearest neighbors are retained after batch correction. It is used to show that MNN-based integrators (Seurat V3.0, MNNcorrect, Scanorama) disrupt local dataset structure while BATMAN preserves it.

## When to use

Use when evaluating scRNA-seq integration quality beyond batch mixing: you have matched pre- and post-integration log-count matrices and need to quantify how many of each cell's original top-k neighbors survive correction.

## When NOT to use

- You only have a single dataset or a single corrected output with no matched pre-integration matrices, so the before/after neighbor comparison cannot be performed
- You want a batch-mixing score (e.g., iLISI) rather than a local-structure-preservation score; k-RNN rewards retention of within-dataset neighborhoods and says nothing about cross-dataset mixing by itself
- The corrected output is a low-dimensional embedding rather than an expression matrix in the original gene space, so Euclidean distances are not comparable to the pre-integration gene-space neighborhoods

## Inputs

- Pre-integration log-count matrices for each scRNA-seq dataset (e.g., Muraro CEL-Seq2 pancreas, 2,285 cells; Segerstolpe Smart-Seq2 pancreas, 2,394 cells)
- Post-integration (batch-corrected) log-count matrices for each integration method (BATMAN, Seurat V3.0, MNNcorrect, Scanorama)
- Parameter k for the neighborhood size (k=50 in the BATMAN paper)

## Outputs

- Mean k-RNN score per integration method (fraction of pre-integration top-k neighbors retained, averaged over all cells)
- Comparison table (krnn_scores.csv) of mean 50-RNN values across Original, BATMAN, Seurat V3.0, MNNcorrect, and Scanorama

## How to apply

Load the pre- and post-integration log-count matrices for each dataset (e.g., Muraro CEL-Seq2 with 2,285 cells and Segerstolpe Smart-Seq2 with 2,394 cells) produced by each integration method (BATMAN, Seurat V3.0, MNNcorrect, Scanorama). For every cell, compute its top-k nearest-neighbor set (k=50 in the paper) in the high-dimensional gene space using Euclidean distance, both before and after integration. For each cell, count how many of its pre-integration neighbors remain in its post-integration neighbor set (neighbors restricted to the cell's original dataset), and average this fraction across all cells in the union of the two datasets. The original (uncorrected) data yields a score of 1.00 by definition, serving as the upper bound. Report per-method mean 50-RNN values in a comparison table (krnn_scores.csv) alongside mixing metrics such as mean iLISI.

## Related tools

- **BATMAN** (Integration method whose corrected outputs are evaluated with the k-RNN metric; also the tool producing the highest mean 50-RNN (0.92) in the paper's LB scenario) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Provides pre-processing steps to obtain log-counts for each dataset; its integration output is one of the methods scored with 50-RNN (0.58))
- **MNNcorrect** (MNN-based integration method evaluated with the k-RNN metric (mean 50-RNN 0.51))
- **Scanorama** (MNN-based integration method evaluated with the k-RNN metric (mean 50-RNN 0.14))

## Examples

```
import numpy as np; from scipy.spatial.distance import cdist; pre = cdist(X_pre, X_pre, 'euclidean'); post = cdist(X_post, X_post, 'euclidean'); krnn = np.mean([len(set(np.argsort(pre[i])[:51]) & set(np.argsort(post[i])[:51]))/50 for i in range(len(X_pre))])
```

## Evaluation signals

- The uncorrected/original data must yield a mean k-RNN of exactly 1.00, since every pre-integration neighbor is trivially retained
- All per-cell scores must lie in [0, 1]; the per-method mean is the average of these fractions
- Reproduction check: mean 50-RNN values should match the reported pattern (Original 1.00, BATMAN 0.92, Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14) on the Muraro/Segerstolpe LB scenario
- Sanity check that neighbor sets are computed with Euclidean distance in high-dimensional gene space and that post-integration neighbors are restricted to each cell's original dataset
- The output table krnn_scores.csv must contain one mean 50-RNN row per method plus Original, consistent with the iLISI comparison table

## Limitations

- k-RNN rewards retention of within-dataset local structure and can be maximized by trivial (unmixed) integration; it must be interpreted jointly with mixing metrics like iLISI
- The choice of k (e.g., 50) and distance metric (Euclidean) affects scores; results are not directly comparable across different k or preprocessing pipelines
- MNN-based integration methods (Seurat V3.0, MNNcorrect, Scanorama) fail to both correct batch effects and maintain local dataset structure, so low k-RNN for them reflects the paradigm, not just implementation

## Evidence
