---
name: tsne-perplexity-hyperparameter-sweep
description: Use when a t-SNE (or UMAP) embedding's quality depends on an unknown
  scale hyperparameter and you need to pick perplexity/kEff or detect characteristic
  neighborhood scales using per-sample p values rather than visual inspection.
when_to_use_negative:
- You only need a single t-SNE embedding for visualization and do not intend to compare
  scales or optimize perplexity — plain EMBEDR (single fit) suffices.
- Your question is about cluster identity or annotation of an existing embedding rather
  than whether the embedding's structure exceeds what a marginally-resampled null
  would produce.
- You cannot afford multiple embeddings per scale — with only 1 data and 1 null embedding
  the empirical p values have coarse resolution and sweep conclusions weaken.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Python package providing the EMBEDR and EMBEDR_sweep classes, sweep_boxplot/sweep_lineplot,
    and fit_samplewise_optimal used to run and summarize the perplexity sweep
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation used as the t-SNE DRA backend whose perplexity hyperparameter
    is swept
- name: FIt-SNE
  role: Fast FFT-accelerated t-SNE implementation used by EMBEDR to speed up the many
    (null) embeddings required by the sweep; requires fftw
  repo: https://github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Loads and preprocesses the count matrix (e.g. np.loadtxt of the feature matrix)
    before fitting the sweep
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:5d46f09fc63449ec7c966aaac5dc94e68854abcc3e89edac635565d1de5189f8
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tsne-perplexity-hyperparameter-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweep the t-SNE perplexity (kEff) hyperparameter with EMBEDR_sweep, embedding data and marginally-resampled null data at each scale to obtain per-sample EMBEDR p values, so the characteristic neighborhood scales and the optimal perplexity for a dataset can be identified statistically rather than by eye.

## When to use

Use when a t-SNE (or UMAP) embedding's quality depends on an unknown scale hyperparameter and you need to pick perplexity/kEff or detect characteristic neighborhood scales using per-sample p values rather than visual inspection.

## When NOT to use

- You only need a single t-SNE embedding for visualization and do not intend to compare scales or optimize perplexity — plain EMBEDR (single fit) suffices.
- Your question is about cluster identity or annotation of an existing embedding rather than whether the embedding's structure exceeds what a marginally-resampled null would produce.
- You cannot afford multiple embeddings per scale — with only 1 data and 1 null embedding the empirical p values have coarse resolution and sweep conclusions weaken.

## Inputs

- High-dimensional feature matrix (e.g. Tabula Muris bone marrow scRNA-seq counts, ~4,800 cells; or MNIST 2500-digit matrix loaded via np.loadtxt)
- Sweep parameter list of perplexity/kEff values
- EMBEDR_sweep configuration: DRA, sweep_type, n_data_embed, n_null_embed, project_dir/project_name

## Outputs

- Per-sample EMBEDR p values at each swept perplexity/kEff value
- Cached data and null embeddings per scale in the project directory
- sweep_boxplot / sweep_lineplot figures summarizing p-value distributions across the sweep
- Sample-wise (cell-wise) optimal embedding after fit_samplewise_optimal

## How to apply

Instantiate EMBEDR_sweep with DRA='tsne', sweep_type='perplexity', and a range of kEff/perplexity values spanning the scales of interest (e.g. including ~150 and ~1200 for the ~4,800-cell Tabula Muris marrow data); set n_data_embed to 3 or more since 3-10 embeddings are recommended in practice. Call fit so that, at each kEff, multiple data embeddings with different random initializations plus a marginally-resampled null embedding are generated and cached in the project directory (do_cache, default ./projects/). EMBEDR compares high-dimensional neighborhood affinities of each sample in the data embeddings against the empirical null distribution to compute a per-sample p value at every scale. Summarize the sweep with sweep_boxplot and sweep_lineplot to locate the kEff with the largest fraction of minimal p values (the global optimum, kEff~1200 in the paper) and any secondary plateaus (kEff~150, a second native scale). Optionally call fit_samplewise_optimal to build a cell-wise optimal embedding using each sample's best perplexity from the swept values.

## Related tools

- **EMBEDR** (Python package providing the EMBEDR and EMBEDR_sweep classes, sweep_boxplot/sweep_lineplot, and fit_samplewise_optimal used to run and summarize the perplexity sweep) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation used as the t-SNE DRA backend whose perplexity hyperparameter is swept)
- **FIt-SNE** (Fast FFT-accelerated t-SNE implementation used by EMBEDR to speed up the many (null) embeddings required by the sweep; requires fftw) — https://github.com/KlugerLab/FIt-SNE
- **numpy** (Loads and preprocesses the count matrix (e.g. np.loadtxt of the feature matrix) before fitting the sweep)

## Examples

```
sweepObj = EMBEDR_sweep(project_name='EMBEDR_Sweep_Example', project_dir='./projects/', DRA='tsne', n_jobs=-1, n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[25, 100, 250]); sweepObj.fit(X); sweepObj.sweep_boxplot(); sweepObj.sweep_lineplot()
```

## Evaluation signals

- Validation criterion from the paper: the sweep_boxplot figure reproduces the Figure 4D pattern, with the median fraction of minimal p values peaking at a single kEff (~1200 cells) and a secondary plateau near kEff~150.
- At each swept perplexity, the stored project directory contains n_data_embed data embeddings (>=3 recommended) plus the null embedding; incremental re-runs (e.g. raising n_data_embed from 3 to 5) should generate only the 2 missing embeddings per scale due to file caching.
- Per-sample p values must lie in [0,1] and be computed against the empirical marginal-resampling null distribution, not a parametric one.
- The chosen optimal perplexity must be one of the sweep_values actually used — fit_samplewise_optimal selects only from swept values.
- sweep_lineplot should show a coherent curve of p-value summary vs kEff enabling identification of characteristic scales; scattered non-monotonic noise suggests too few embeddings.

## Limitations

- Calculated p values have finite resolution because null distributions are computed empirically; the number of null embeddings bounds the lower limit of attainable p values.
- No theoretical description of the tails of the null distributions or principled parameterization of the null exists.
- Runtime rests on the speed of current DR implementations that must be chained to generate many (null) embeddings; sweep timings are only reported in supplementary Table S2.
- EMBEDR's null-generation process has not been extended to non-normalized datasets.
- Optimal perplexities from fit_samplewise_optimal are restricted to the values included in the sweep.

## Evidence
