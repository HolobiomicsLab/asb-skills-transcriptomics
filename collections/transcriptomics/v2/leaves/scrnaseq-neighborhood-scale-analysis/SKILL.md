---
name: scrnaseq-neighborhood-scale-analysis
description: Use when selecting a t-SNE perplexity/kEff for single-cell embeddings,
  or when characterizing whether scRNA-seq data have one or multiple intrinsic neighborhood
  scales via per-sample EMBEDR p-value sweeps.
when_to_use_negative:
- When the goal is a single quality-scored embedding at a fixed perplexity — use the
  plain EMBEDR class fit method instead of the sweep machinery
- 'When the data are non-normalized: EMBEDR''s null-generation process has not been
  extended to non-normalized datasets'
- When you need a specific cell count as the exact optimum regardless of the sweep
  grid — optima are only selectable from the perplexity values actually swept
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Provides EMBEDR_sweep class to run the perplexity sweep, compute per-sample
    p-values, and produce sample-wise optimal embeddings
  repo: github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation of the openTSNE implementation is the t-SNE engine used for
    the data and null embeddings
- name: FIt-SNE
  role: Fast FFT-accelerated t-SNE implementation used by EMBEDR; requires fftw installation
  repo: github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Loads and preprocesses the count matrix for embedding
- name: scikit-learn
  role: EMBEDR installation dependency
- name: scipy
  role: EMBEDR installation dependency
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:3d93640cb7b51ce23b4d9f178a774598a72e345aaeffcbf8ebe3f7134c05f331
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - FIt-SNE
  - Python
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# scrnaseq-neighborhood-scale-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweep the effective number of nearest neighbors (kEff/perplexity) for t-SNE embeddings of scRNA-seq data using EMBEDR p-values to identify the neighborhood scale at which the largest fraction of cells is significantly well embedded, revealing native data scales.

## When to use

Use when selecting a t-SNE perplexity/kEff for single-cell embeddings, or when characterizing whether scRNA-seq data have one or multiple intrinsic neighborhood scales via per-sample EMBEDR p-value sweeps.

## When NOT to use

- When the goal is a single quality-scored embedding at a fixed perplexity — use the plain EMBEDR class fit method instead of the sweep machinery
- When the data are non-normalized: EMBEDR's null-generation process has not been extended to non-normalized datasets
- When you need a specific cell count as the exact optimum regardless of the sweep grid — optima are only selectable from the perplexity values actually swept

## Inputs

- scRNA-seq count matrix (e.g. Tabula Muris FACS raw counts CSV, ~4,800 bone marrow cells)
- Range of kEff/perplexity values (sweep_values list spanning scales such as 25, 100, 150, 250 ... ~1200)
- EMBEDR package installed via pip (requires numpy, scikit-learn, scipy, conda, numba; seaborn for figures; fftw for FIt-SNE)

## Outputs

- Data and null embeddings at each swept kEff, file-cached in the project directory
- Per-sample EMBEDR p-values at each perplexity/kEff
- sweep_boxplot and sweep_lineplot figures showing the fraction of minimal p-values across the sweep
- Sample-wise (cell-wise) optimal embedding from fit_samplewise_optimal

## How to apply

Load the count matrix (e.g. ~4,800 Tabula Muris bone marrow cells) with numpy and preprocess it for embedding. Instantiate the EMBEDR_sweep class with DRA='tsne' (the openTSNE/FIt-SNE-based implementation), sweep_type='perplexity', and sweep_values spanning the scales of interest (including ~150), setting n_data_embed to at least 3 since 3-10 data embeddings are recommended over the default of 1. Fitting the sweep generates multiple data embeddings plus a marginally-resampled null-data embedding at each kEff, with intermediate results file-cached in a project directory so repeated runs reuse nearest-neighbor graphs, affinity matrices, and prior embeddings. Compute per-sample EMBEDR p-values at each kEff by comparing data-embedding neighborhood affinities against the empirical null distribution, then summarize the sweep with sweep_boxplot and sweep_lineplot to find the kEff where the median fraction of minimal p-values peaks (kEff≈1200 for Tabula Muris marrow) and any secondary plateau (kEff≈150), indicating two native scales. Optionally call fit_samplewise_optimal to generate a cell-wise optimal embedding, noting optimal perplexities can only be chosen from values in the sweep.

## Related tools

- **EMBEDR** (Provides EMBEDR_sweep class to run the perplexity sweep, compute per-sample p-values, and produce sample-wise optimal embeddings) — github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation of the openTSNE implementation is the t-SNE engine used for the data and null embeddings)
- **FIt-SNE** (Fast FFT-accelerated t-SNE implementation used by EMBEDR; requires fftw installation) — github.com/KlugerLab/FIt-SNE
- **numpy** (Loads and preprocesses the count matrix for embedding)
- **scikit-learn** (EMBEDR installation dependency)
- **scipy** (EMBEDR installation dependency)

## Examples

```
from EMBEDR import EMBEDR_sweep; sweepObj = EMBEDR_sweep(project_name='tm_marrow_sweep', project_dir='./projects/', DRA='tsne', n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[25, 100, 150, 250, 500, 1000, 1200]); sweepObj.fit(X); sweepObj.sweep_boxplot(); sweepObj.sweep_lineplot()
```

## Evaluation signals

- The sweep_boxplot reproduces the Figure 4D pattern: median fraction of minimal EMBEDR p-values peaks at a single kEff (~1200 for Tabula Muris marrow cells)
- A secondary plateau in the p-value curve near kEff≈150 is detectable, indicating two native scales for the data
- With file caching enabled, increasing n_data_embed from 3 to 5 generates only 2 new embeddings per perplexity value instead of recomputing all
- Per-sample p-values are computed against the empirical marginal-resampling null distribution at every swept kEff, not a single global null
- fit_samplewise_optimal runs only after a completed sweep and selects optimal perplexities from the swept values

## Limitations

- Calculated p values have finite resolution since null distributions are computed empirically; the number of nulls bounds the lower limit of p values
- Runtime depends on the speed of current DR algorithm implementations used to generate many null embeddings
- No theoretical description of the tails of the null distributions exists and no principled method for parameterizing the null distribution exists
- EMBEDR's null-generation process has not been extended to non-normalized datasets

## Evidence
