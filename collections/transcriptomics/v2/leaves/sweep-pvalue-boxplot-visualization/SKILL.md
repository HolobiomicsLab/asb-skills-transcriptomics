---
name: sweep-pvalue-boxplot-visualization
description: Use when you have fitted an EMBEDR_sweep object over multiple perplexity/kEff
  values (with n_data_embed>=3 and null embeddings) and need to summarize which scales
  yield the largest fraction of minimal EMBEDR p-values.
when_to_use_negative:
- You only have a single embedding at one hyperparameter value rather than a sweep
  — use the EMBEDR class fit/plot instead of sweep_boxplot
- The dimensionality reduction algorithm is not t-SNE or UMAP — the EMBEDR package
  currently supports only DRA='tsne' and 'UMAP'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Provides the EMBEDR_sweep class whose sweep_boxplot and sweep_lineplot methods
    render the p-value sweep summaries
  repo: https://github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Custom variation used as the t-SNE DRA generating the embeddings swept over
    in the p-value figures
- name: seaborn
  role: Required package for generating the EMBEDR sweep figures (boxplots and lineplots)
- name: numpy
  role: Loads and types the input data matrix passed to the sweep (e.g., np.loadtxt(...).astype(float))
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:47127c782935f7578aa2046debc3c0bb0f9827d870a00082908556d85b60b160
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
  - Python
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# sweep-pvalue-boxplot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize EMBEDR p-value distributions across a DRA hyperparameter sweep (e.g., t-SNE perplexity/kEff) using the EMBEDR_sweep class's sweep_boxplot and sweep_lineplot functions, to identify optimal neighborhood scales such as the kEff~1200 optimum with a secondary plateau at kEff~150 in Tabula Muris marrow.

## When to use

Use when you have fitted an EMBEDR_sweep object over multiple perplexity/kEff values (with n_data_embed>=3 and null embeddings) and need to summarize which scales yield the largest fraction of minimal EMBEDR p-values.

## When NOT to use

- You only have a single embedding at one hyperparameter value rather than a sweep — use the EMBEDR class fit/plot instead of sweep_boxplot
- The dimensionality reduction algorithm is not t-SNE or UMAP — the EMBEDR package currently supports only DRA='tsne' and 'UMAP'

## Inputs

- Feature/count matrix (e.g., Tabula Muris bone marrow scRNA-seq counts, ~4,800 cells; or MNIST 2500 samples) loaded via numpy
- Fitted EMBEDR_sweep object with sweep_type='perplexity', a list of sweep_values (kEff/perplexity), n_data_embed>=3, and at least one null embedding per value

## Outputs

- Box-and-whisker figure of EMBEDR p-value distributions at each sweep value (sweep_boxplot)
- Line plot summarizing the sweep trend (sweep_lineplot)
- Identified optimal kEff/perplexity (e.g., ~1200) and secondary plateau scale (e.g., ~150)
- Cached intermediate embeddings, nearest-neighbor graphs, and affinity matrices in the project directory

## How to apply

After instantiating EMBEDR_sweep with DRA='tsne', sweep_type='perplexity', and a kEff range spanning the scales of interest (e.g., including ~150 and ~1200), call sweepObj.fit(X) so that multiple data embeddings and a marginally-resampled null embedding are generated per kEff with file caching in the project directory. Then call sweepObj.sweep_boxplot() and sweepObj.sweep_lineplot() to render the box-and-whisker and line summaries of per-sample EMBEDR p-values at each hyperparameter value. Read the figures to locate the kEff where the median fraction of minimal p-values peaks (Figure 4D pattern: peak at ~1200 with a secondary plateau near ~150, indicating two native scales). Optionally call fit_samplewise_optimal() afterward to build a cell-wise optimal embedding; optimal perplexity values can only be selected from those included in the sweep.

## Related tools

- **EMBEDR** (Provides the EMBEDR_sweep class whose sweep_boxplot and sweep_lineplot methods render the p-value sweep summaries) — https://github.com/ejohnson643/EMBEDR
- **openTSNE** (Custom variation used as the t-SNE DRA generating the embeddings swept over in the p-value figures)
- **seaborn** (Required package for generating the EMBEDR sweep figures (boxplots and lineplots))
- **numpy** (Loads and types the input data matrix passed to the sweep (e.g., np.loadtxt(...).astype(float)))

## Examples

```
sweepObj = EMBEDR_sweep(project_name='EMBEDR_Sweep_Example', DRA='tsne', n_data_embed=3, n_null_embed=1, sweep_type='perplexity', sweep_values=[25, 100, 250]); sweepObj.fit(X); sweepObj.sweep_boxplot(); sweepObj.sweep_lineplot()
```

## Evaluation signals

- The box-and-whisker figure reproduces the Figure 4D pattern: the median fraction of minimal p-values peaks (~1200 cells) at a single kEff and shows a secondary plateau near kEff~150
- The lineplot trend is consistent with the boxplot medians across all swept kEff values
- File caching avoids recomputation: changing n_data_embed from 3 to 5 generates only 2 new embeddings per sweep value because 3 already exist
- Each swept hyperparameter value has n_data_embed data embeddings and at least one marginally-resampled null embedding before p-values are computed
- Per-sample p-values are bounded below by the empirical null resolution set by the number of null embeddings

## Limitations

- EMBEDR p-values have finite resolution because null distributions are computed empirically; the number of null embeddings bounds the lower limit of p-values
- Sweep runtime depends on the speed of the underlying DRA implementations used to generate many (null) embeddings; GPU extensions were not included in this work
- Hyperparameter sweep timings are only reported in supplementary Table S2, not in the main text

## Evidence
