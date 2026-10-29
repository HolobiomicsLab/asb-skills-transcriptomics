---
name: l2-cell-vector-normalization
description: Use when preprocessing scRNA-seq count matrices (CSV, JLD2, or 10x mtx.gz)
  where log normalization may introduce TGC-bias-driven artificial signals — e.g.,
  detected eigenvalue counts unexpectedly high after log normalization on biased or
  synthetic data.
when_to_use_negative:
- Input data lacks total-count (TGC) bias or sparsity-driven distortion — log normalization
  alone may already yield a stable signal count under the Tracy-Widom criterion
- Goal is removing low-quality signals from stochastic dropout rather than log-normalization-induced
  distortion — use the scLENS signal robustness test (robustness value >0.5) instead,
  as L2 normalization alone does not address dropout
- The pipeline already applies another normalization that uniformizes cell vector
  lengths — double normalization is unnecessary
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scLENS
  role: Julia/Python package whose pipeline performs log normalization with L=1, gene
    scaling, and L2 normalization of cell vector lengths before RMT-based noise filtering
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS (conda environment sclens-py with PyTorch CUDA)
    providing the preprocessing that includes L2 normalization
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:b0e51d1e1282fff6e6692d2a2cbba5f9d0f54c20eec2cc00a080240ad7c8fb48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CUDA
  - Julia
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# l2-cell-vector-normalization

## Summary

Applies L2 normalization of cell vector lengths after log normalization (scaling factor L=1) plus gene scaling to remove the artificial/false signals that conventional log normalization alone introduces in scRNA-seq data, as implemented in scLENS.

## When to use

Use when preprocessing scRNA-seq count matrices (CSV, JLD2, or 10x mtx.gz) where log normalization may introduce TGC-bias-driven artificial signals — e.g., detected eigenvalue counts unexpectedly high after log normalization on biased or synthetic data.

## When NOT to use

- Input data lacks total-count (TGC) bias or sparsity-driven distortion — log normalization alone may already yield a stable signal count under the Tracy-Widom criterion
- Goal is removing low-quality signals from stochastic dropout rather than log-normalization-induced distortion — use the scLENS signal robustness test (robustness value >0.5) instead, as L2 normalization alone does not address dropout
- The pipeline already applies another normalization that uniformizes cell vector lengths — double normalization is unnecessary

## Inputs

- scRNA-seq count matrix (CSV e.g. data/Z8eq.csv.gz, JLD2 DataFrame, or converted 10x matrix.mtx.gz/features.tsv.gz/barcodes.tsv.gz)
- QC-filtered preprocessed DataFrame from scLENS.preprocess (cells <200 expressed genes and genes <15 cells filtered)

## Outputs

- L2-normalized log-transformed expression matrix used to build cell similarity matrix G = XX^T/N
- Reduced eigenvalue/signal count (e.g. 33→6 on Z8eq, 84→42 on ZhengMix, ~400→0 on TGC-biased synthetic matrix)
- Signal counts written to CSV (e.g. fig2_signal_detection_results.csv, zhengmix_signal_counts.csv)

## How to apply

After log normalization with scaling factor L=1 and per-gene scaling (median scaling replaces z-score scaling in the memory-efficient scLENS implementation), divide each cell vector by its Euclidean (L2) length so all cell vectors are uniform length. Rationale: log normalization amplifies gaps between zero and non-zero values and leaves cell vector lengths uneven, which distorts the cell similarity matrix G=XX^T/N and produces artificial signals detectable via Marchenko-Pastur/Tracy-Widom RMT filtering. Decision point: rerun RMT-based noise filtering (eigenvalue decomposition, MP fit, Tracy-Widom threshold at p=0.05) on both log-only and log+L2 conditions and compare signal counts — on the Fig. 2 TGC-biased synthetic matrix, L2 normalization removed all ~400 artificial signals (0 afterward), and on the real Z8eq dataset counts dropped from 33 to 6; on ZhengMix PBMC data counts fell from 84 to 42. L2 normalization alone does not remove low-quality signals from stochastic dropout — that requires the subsequent signal robustness test in the full scLENS pipeline.

## Related tools

- **scLENS** (Julia/Python package whose pipeline performs log normalization with L=1, gene scaling, and L2 normalization of cell vector lengths before RMT-based noise filtering) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS (conda environment sclens-py with PyTorch CUDA) providing the preprocessing that includes L2 normalization) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev)
```

## Evaluation signals

- On a TGC-biased synthetic matrix (first 400 of 2000 rows halved), signal count under log-only should be ~400 and drop to 0 with L2 normalization
- On the real Z8eq dataset (data/Z8eq.csv.gz via scLENS.read_file), signals should drop from 33 (log) to 6 (log+L2)
- On ZhengMix PBMC data, signals should decrease from 84 (log) to 42 (log+L2) using the Marchenko-Pastur fit and Tracy-Widom threshold at p=0.05
- Verify all cell vectors in the normalized matrix have (numerically) unit Euclidean length before computing G = XX^T/N

## Limitations

- L2 normalization is described as a 'very simple approach' — it removes log-normalization-induced artificial signals but does not remove low-quality signals from stochastic dropout (requires the signal robustness test)
- RMT-based noise filtering after normalization requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory; even scLENS on large whole-gene data requires a computer with high specifications
- scLENS uses median scaling instead of conventional z-score gene scaling to reduce memory demand, so results may differ slightly from z-score-based re-implementations

## Evidence

- [abstract] L2 normalization identification: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [discussion] scaling factor discussion: "we set a scaling factor of 1 during the first step of log normalization"
- [discussion] L2 after log normalization: "scLENS reduces signal distortion during the preprocessing step by incorporating L2 normalization"
- [discussion] RMT threshold description: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [intro] log normalization distortion cause: "conventional log normalization can introduce false variability by amplifying gaps between zero and non-zero values and not uniformly normalizing genes across different expression levels"
- [readme] preprocess QC parameters: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [discussion] median scaling memory note: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
