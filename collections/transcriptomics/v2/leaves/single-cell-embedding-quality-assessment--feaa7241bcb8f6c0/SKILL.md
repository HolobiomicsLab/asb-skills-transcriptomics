---
name: single-cell-embedding-quality-assessment
description: Use when you have a scRNA-seq or scATAC-seq count matrix (e.g. Tabula
  Muris marrow FACS counts, 10x PBMC scATAC peak matrix) and need to judge which parts
  of a t-SNE/UMAP embedding reflect biological signal versus distortion, or to choose
  embedding hyperparameters.
when_to_use_negative:
- Input is non-normalized data or an entire downstream processing pipeline output
  — the paper notes EMBEDR's null-generation process 'has not been extended to non-normalized
  datasets'
- You need very small p values beyond the empirical resolution of your null set —
  the number of null embeddings bounds the lower limit of p values
- The goal is a purely global embedding-quality score rather than local (per-cell)
  error estimates and null-hypothesis testing
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: 'Core Python package: EMBEDR and EMBEDR_sweep classes generate data/null embeddings,
    per-sample p values, sweep plots, and sample-wise optimal embeddings'
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Backend dimensionality-reduction algorithm (custom variation) used for data
    and null t-SNE embeddings
- name: FIt-SNE
  role: Fast FFT-accelerated t-SNE implementation used by EMBEDR; requires fftw to
    be installed
  repo: https://github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Array loading and manipulation of count matrices (e.g. np.loadtxt of count
    data)
- name: scipy
  role: Required dependency of the EMBEDR package
- name: scikit-learn
  role: Required dependency of the EMBEDR package
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:ee9091eec99bd9a94226de50f5fc3e5512f4a9200c858b75bcdaaa642345645f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  repo_url: https://github.com/ejohnson643/EMBEDR
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
triage_status: promoted
yes_rate: 0.76
n_claims: 25
---

# single-cell-embedding-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess whether single-cell dimensionality-reduction embeddings (t-SNE/UMAP) contain real structure versus algorithm-induced noise using EMBEDR, which computes a per-sample empirical p value by comparing data embeddings against marginally-resampled null embeddings. This enables per-cell quality scoring, DR algorithm/hyperparameter comparison, and detection of sample-specific optimal neighborhood scales.

## When to use

Use when you have a scRNA-seq or scATAC-seq count matrix (e.g. Tabula Muris marrow FACS counts, 10x PBMC scATAC peak matrix) and need to judge which parts of a t-SNE/UMAP embedding reflect biological signal versus distortion, or to choose embedding hyperparameters.

## When NOT to use

- Input is non-normalized data or an entire downstream processing pipeline output — the paper notes EMBEDR's null-generation process 'has not been extended to non-normalized datasets'
- You need very small p values beyond the empirical resolution of your null set — the number of null embeddings bounds the lower limit of p values
- The goal is a purely global embedding-quality score rather than local (per-cell) error estimates and null-hypothesis testing

## Inputs

- scRNA-seq raw counts matrix (e.g. Tabula Muris FACS bone marrow counts CSV, ~4,800 cells, GEO GSE109774 or Figshare deposit)
- scATAC-seq filtered counts-by-peaks matrix (e.g. atac_pbmc_10k_nextgem_filtered_peak_bc_matrix.h5) with cell/peak filtering applied (peaks in <= 10 cells and cells with <= 200 peaks removed)
- Cell-type annotations (e.g. from czbiohub tabula-muris data_ingest metadata)
- MNIST 2500 sample matrix (mnist2500_X.txt) as a test case

## Outputs

- Per-cell EMBEDR p-value table (all values in [0, 1])
- Embedding figure colored by EMBEDR p-value (e.g. embedr_pvalue_embedding.png)
- Perplexity/kEff sweep summary plots (sweep_boxplot / sweep_lineplot figures)
- Per-cell-type median and 90th-percentile p-value table per sweep value (e.g. figure6_celltype_summary.csv)
- Sample-wise (cell-wise) optimal embedding via fit_samplewise_optimal

## How to apply

Load the count matrix into a numpy array (e.g. ~4,800 Tabula Muris bone marrow FACS cells from GSE109774 or the Figshare deposit) and initialize an EMBEDR object with project_dir caching, DRA='tsne' (openTSNE/FIt-SNE backed), perplexity set to 10% of the number of samples, and n_data_embed of 3-10 with n_null_embed >= 1. Call the fit method to generate multiple data embeddings and a marginally-resampled null dataset embedding; the per-cell EES statistic (neighborhood correspondence between high- and low-dimensional space) is compared against the aggregated null EES* distribution, and the fraction of null statistics smaller than the observed one yields an empirical p value averaged across the data embeddings. To find characteristic scales, use the EMBEDR_sweep class to sweep perplexity/kEff values (e.g. ~150 up to ~1,200), relying on file caching of kNN graphs and affinity matrices to avoid recomputation, then visualize with sweep_boxplot and sweep_lineplot and optionally call fit_samplewise_optimal to produce a cell-wise optimal embedding. Intermediate results are stored in the projects directory, so re-runs with additional n_data_embed only compute the missing embeddings.

## Related tools

- **EMBEDR** (Core Python package: EMBEDR and EMBEDR_sweep classes generate data/null embeddings, per-sample p values, sweep plots, and sample-wise optimal embeddings) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Backend dimensionality-reduction algorithm (custom variation) used for data and null t-SNE embeddings)
- **FIt-SNE** (Fast FFT-accelerated t-SNE implementation used by EMBEDR; requires fftw to be installed) — https://github.com/KlugerLab/FIt-SNE
- **numpy** (Array loading and manipulation of count matrices (e.g. np.loadtxt of count data))
- **scipy** (Required dependency of the EMBEDR package)
- **scikit-learn** (Required dependency of the EMBEDR package)

## Examples

```
sweepObj = EMBEDR_sweep(project_name='EMBEDR_Sweep_Example', project_dir='./projects/', DRA='tsne', n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[25, 100, 250]); sweepObj.fit(X); sweepObj.sweep_lineplot()
```

## Evaluation signals

- All per-cell EMBEDR p-values lie within [0, 1]
- Output figure (e.g. embedr_pvalue_embedding.png) exists and reproduces the qualitative pattern of well-embedded clusters showing low p values
- Sweep summary reproduces the Figure 6 pattern: macrophages best embedded near kEff ≈ 150 while granulocytes and B cells prefer kEff ≈ 1,200, with all reported p-value statistics in [0, 1]
- Caching behavior: changing n_data_embed from 3 to 5 after a run generates only 2 new embeddings per sweep value, not a full re-run
- Number of clusters detected in downstream DBSCAN runs is reported explicitly (n_db_labels), not left as a template variable

## Limitations

- Runtime depends on the speed of current DR algorithm implementations used to generate many null embeddings; GPU extensions and quadratic-rate optimization schemes were not included
- Calculated p values have finite resolution since null distributions are computed empirically; the number of nulls determines the lower bound on p values
- No theoretical description of the tails of the null distributions exists, and no principled method for parameterizing the null distribution exists
- The null-generation process has not been extended to non-normalized datasets
- Sample-wise optimal perplexity values can only be selected from the values used in the sweep
- Hyperparameter sweep timings are only reported in supplementary Table S2, not the main text

## Evidence
