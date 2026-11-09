---
name: per-cell-optimal-neighborhood-scale-selection
description: Use when an EMBEDR perplexity (or n_neighbors) sweep has been fit on
  scRNA-seq/scATAC-seq data and you want per-sample optimal scales, or a sample-wise
  optimal embedding, instead of a single global hyperparameter.
when_to_use_negative:
- No EMBEDR sweep has been fit yet — the optimal perplexity can only be selected from
  values used in the sweep, so the skill requires a completed sweep first
- The goal is a single globally optimal hyperparameter rather than a per-cell heterogeneous
  neighborhood-scale embedding (use the sweep boxplot/lineplot summaries instead)
- The dataset is not normalized — the article notes EMBEDR's null-generation process
  has not been extended to non-normalized datasets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Provides the EMBEDR class for fitting data/null embeddings and the EMBEDR_sweep
    class whose fit_samplewise_optimal and plot_embedding methods implement per-cell
    perplexity selection and rendering of the cell-wise optimal embedding
  repo: https://github.com/ejohnson643/EMBEDR
- name: scikit-learn
  role: Supplies DBSCAN used to cluster the cell-wise optimal embedding for validation
    against cell-type annotations (eps = 1.5th percentile of pairwise distances)
- name: FIt-SNE
  role: Fast t-SNE implementation used by EMBEDR as the DRA; requires fftw to be installed
  repo: https://github.com/KlugerLab/FIt-SNE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:37bfc6e2161cdc98b77c6df6638deaf1e687aab5088a101aa540e781211dcea6
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
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# per-cell-optimal-neighborhood-scale-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects, for each cell/sample individually, the hyperparameter (perplexity for t-SNE) whose EMBEDR p-value is lowest, then uses those per-cell optimal scales to build a 'cell-wise optimal' embedding whose neighborhood size varies across the data. This reveals characteristic neighborhood scales per cell rather than forcing one global hyperparameter.

## When to use

Use when an EMBEDR perplexity (or n_neighbors) sweep has been fit on scRNA-seq/scATAC-seq data and you want per-sample optimal scales, or a sample-wise optimal embedding, instead of a single global hyperparameter.

## When NOT to use

- No EMBEDR sweep has been fit yet — the optimal perplexity can only be selected from values used in the sweep, so the skill requires a completed sweep first
- The goal is a single globally optimal hyperparameter rather than a per-cell heterogeneous neighborhood-scale embedding (use the sweep boxplot/lineplot summaries instead)
- The dataset is not normalized — the article notes EMBEDR's null-generation process has not been extended to non-normalized datasets

## Inputs

- High-dimensional single-cell data matrix (e.g. Tabula Muris bone marrow scRNA-seq counts, ~4,800 cells; MNIST-style numeric matrices)
- Completed EMBEDR_sweep results containing per-cell EMBEDR p-values across the hyperparameter sweep (e.g. perplexity 25, 100, 250)
- Optional: per-cell annotations (e.g. Tabula Muris cell-type labels) for downstream validation

## Outputs

- Per-cell optimal perplexity values (kEff) selected from the sweep grid
- Cell-wise (sample-wise) optimal 2D embedding of the data (e.g. samplewise_optimal_embedding.png)
- Optional downstream artifacts such as a DBSCAN cluster-label confusion table against annotations (cluster_confusion_table.csv)

## How to apply

First run an EMBEDR_sweep over discrete hyperparameter values (e.g. sweep_type='perplexity', sweep_values=[25, 100, 250]) with multiple data embeddings (n_data_embed=3-10 recommended) and null embeddings, storing per-cell EMBEDR p-values. Call EMBEDR_sweep.fit_samplewise_optimal, which selects for each cell the perplexity (kEff) minimizing its EMBEDR p-value, restricted to the values used in the sweep; the class caches intermediate results in a project directory (or set do_cache=False). Build the per-cell adaptive affinity matrix and generate the cell-wise optimal t-SNE embedding via the EMBEDR package's tSNE DRA, then render it with EMBEDR_sweep.plot_embedding. In the Tabula Muris marrow application, the resulting embedding was clustered with scikit-learn DBSCAN (eps = 1.5th percentile of pairwise cell-cell distances) to validate that per-cell structure matched expert annotations.

## Related tools

- **EMBEDR** (Provides the EMBEDR class for fitting data/null embeddings and the EMBEDR_sweep class whose fit_samplewise_optimal and plot_embedding methods implement per-cell perplexity selection and rendering of the cell-wise optimal embedding) — https://github.com/ejohnson643/EMBEDR
- **scikit-learn** (Supplies DBSCAN used to cluster the cell-wise optimal embedding for validation against cell-type annotations (eps = 1.5th percentile of pairwise distances))
- **FIt-SNE** (Fast t-SNE implementation used by EMBEDR as the DRA; requires fftw to be installed) — https://github.com/KlugerLab/FIt-SNE

## Examples

```
sweepObj = EMBEDR_sweep(project_name='marrow', DRA='tsne', sweep_type='perplexity', sweep_values=[25, 100, 250], n_data_embed=3, n_null_embed=1); sweepObj.fit(X); sweepObj.fit_samplewise_optimal(); sweepObj.plot_embedding()
```

## Evaluation signals

- The set of selected per-cell optimal perplexities contains only values drawn from the original sweep grid (e.g. {25, 100, 250}) — no interpolated or off-grid values
- The rendered samplewise_optimal_embedding.png is produced only after fit_samplewise_optimal has been called on a completed sweep
- DBSCAN on the cell-wise optimal embedding (eps = 1.5th percentile of pairwise distances) recovers clusters concordant with the seven annotated Tabula Muris bone marrow cell types, with near-disjoint confusion rows except granulocyte-monocyte progenitor cells (Figure 7D/7E)
- Cached intermediate results in the project directory allow re-runs to reuse previously generated embeddings and nearest-neighbor/affinity computations rather than recomputing
- Per-cell EMBEDR p-values at the chosen scale should be lower (more significant structure) than at other swept scales for most cells

## Limitations

- Optimal perplexity values are restricted to the discrete values used in the sweep; continuous kEff is not supported
- p-value resolution is finite because null distributions are empirical — the number of null embeddings bounds the lowest achievable p-value, which limits per-cell scale discrimination
- There is no theoretical description of the null distribution tails and no principled parameterization, so extremely small p-values may be unreliable
- Hyperparameter sweeps are computationally expensive; runtimes depend on the speed of underlying DR implementations, and only a limited set of DRAs (tSNE, UMAP) are currently supported in the package

## Evidence

- [results] DBSCAN with eps set to the 1.5th percentile of pairwise distances in the cell-wise optimal embedding detected seven clusters, and each clearly corresponded to a class of bone marrow cell types with almost no overlap between annotations except granulocyte-monocyte progenitor cells.: [evidence span withheld — anchor status: not_found]
