---
name: tsne-embedding-of-scrna-seq-counts
description: Use when you have a raw scRNA-seq count matrix (e.g., Tabula Muris FACS
  counts CSV) and need a 2D t-SNE embedding whose per-cell quality can be statistically
  assessed against a signal-less null.
when_to_use_negative:
- You only need a static t-SNE visualization without per-sample statistical quality
  assessment — a plain openTSNE/FIt-SNE run without null embeddings suffices and is
  far cheaper.
- 'Your dataset is not normalized counts (e.g., raw non-normalized pipelines): the
  article notes EMBEDR''s null-generation process has not been extended to non-normalized
  datasets.'
- You require p values with resolution finer than what the number of null embeddings
  supports, since empirical null distributions bound the lower limit of attainable
  p values.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: EMBEDR
  role: Python package whose EMBEDR class wraps openTSNE-backed t-SNE to fit data
    and null embeddings and cache intermediates
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Underlying t-SNE implementation used by EMBEDR as its dimensionality reduction
    algorithm (DRA='tsne')
- name: FIt-SNE
  role: Fast FFT-accelerated t-SNE implementation used when fftw is installed
  repo: https://github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Loads and holds the count matrix (e.g., np.loadtxt of counts) passed to EMBEDR.fit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:65f0151b62b3a4294202a5ceee664da49545a55e6c4ad2f70424ec4da4ca82f5
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# tsne-embedding-of-scrna-seq-counts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embed scRNA-seq count matrices (e.g., ~4,800 FACS-sorted Tabula Muris bone marrow cells, GEO GSE109774) into 2D using an openTSNE-backed t-SNE implementation, as packaged in the EMBEDR Python library, so that per-sample embedding quality can be visualized by coloring cells with EMBEDR p values.

## When to use

Use when you have a raw scRNA-seq count matrix (e.g., Tabula Muris FACS counts CSV) and need a 2D t-SNE embedding whose per-cell quality can be statistically assessed against a signal-less null.

## When NOT to use

- You only need a static t-SNE visualization without per-sample statistical quality assessment — a plain openTSNE/FIt-SNE run without null embeddings suffices and is far cheaper.
- Your dataset is not normalized counts (e.g., raw non-normalized pipelines): the article notes EMBEDR's null-generation process has not been extended to non-normalized datasets.
- You require p values with resolution finer than what the number of null embeddings supports, since empirical null distributions bound the lower limit of attainable p values.

## Inputs

- scRNA-seq count matrix as a numpy array (e.g., ~4,800 FACS-sorted Tabula Muris bone marrow cells from GEO GSE109774)
- FACS counts CSV from Tabula Muris (00_facs_raw_data/FACS/{Tissue}-counts.csv)

## Outputs

- 2D t-SNE embedding of the cells
- Embedding figure colored by per-cell EMBEDR p values (e.g., embedr_pvalue_embedding.png)
- Cached data and null embeddings and intermediate nearest-neighbor/affinity results in the project directory

## How to apply

Clone the ejohnson643/EMBEDR repository and install with `pip install .` (requires numpy, scikit-learn, scipy, conda, numba; seaborn for figures; fftw for fast FIt-SNE). Load the scRNA-seq counts (e.g., Tabula Muris FACS bone marrow counts for ~4,800 cells) into a numpy array. Initialize an EMBEDR object with DRA='tsne' (a custom variation of the openTSNE implementation) and set perplexity to 10% of the number of samples (the recommended default); set n_data_embed and n_null_embed to 3-10 and 1 respectively, since the defaults of 1 are only for quick tests. Call the `fit` method to generate the data embeddings and a marginally-resampled null embedding, caching intermediates in a project directory (set project_dir/project_name; use do_cache=False to skip caching). Call the `plot` method to render the t-SNE embedding colored by EMBEDR p value.

## Related tools

- **EMBEDR** (Python package whose EMBEDR class wraps openTSNE-backed t-SNE to fit data and null embeddings and cache intermediates) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Underlying t-SNE implementation used by EMBEDR as its dimensionality reduction algorithm (DRA='tsne'))
- **FIt-SNE** (Fast FFT-accelerated t-SNE implementation used when fftw is installed) — https://github.com/KlugerLab/FIt-SNE
- **numpy** (Loads and holds the count matrix (e.g., np.loadtxt of counts) passed to EMBEDR.fit)

## Examples

```
embObj = EMBEDR(project_dir='./projects/', project_name='tabula_muris_bm', DRA='tsne', n_data_embed=5, n_null_embed=1); embObj.fit(X); embObj.plot()
```

## Evaluation signals

- Output figure embedr_pvalue_embedding.png exists after calling fit and plot
- All per-cell EMBEDR p values lie within [0, 1]
- Number of data embeddings generated equals n_data_embed (3-10 recommended) and null embeddings equals n_null_embed (1)
- Re-running with do_cache enabled reuses previously generated embeddings and does not recompute nearest-neighbor graphs/affinity matrices for matching parameters
- Embedding of the Tabula Muris bone marrow data (~4,800 cells) reproduces the structure shown in Figure 2D when colored by p value

## Limitations

- Runtime depends on the speed of current DR algorithm implementations used to generate many (null) embeddings of the same data
- P values have finite resolution because null distributions are computed empirically; the number of nulls bounds the lower limit on p values, and no theoretical description of the null distribution tails exists
- EMBEDR's null-generation process has not been extended to non-normalized datasets
- fftw must be installed to use the fast FIt-SNE implementation of t-SNE

## Evidence
