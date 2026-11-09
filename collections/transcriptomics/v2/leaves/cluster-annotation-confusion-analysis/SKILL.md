---
name: cluster-annotation-confusion-analysis
description: Use when you have a cell-wise (sample-wise) optimal EMBEDR embedding
  plus expert cell-type annotations and need to test whether unsupervised cluster
  structure in the embedding is biologically concordant with those annotations.
when_to_use_negative:
- Input is a raw feature/counts table rather than an already-computed cell-wise optimal
  EMBEDR embedding — the sweep and fit_samplewise_optimal steps must be completed
  first
- The perplexity values of interest were not included in the EMBEDR sweep, since optimal
  per-cell perplexities can only be selected from those used in the sweep
- You need a quantitative clustering benchmark with concrete metric values — the article's
  results only report a template-driven cluster count (print statement), not a validated
  numeric score
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EMBEDR
  role: Generates the cell-wise optimal t-SNE embedding via EMBEDR_sweep.fit_samplewise_optimal
    and renders it with plot_embedding
  repo: https://github.com/ejohnson643/EMBEDR
- name: scikit-learn
  role: Provides DBSCAN used to cluster the cell-wise optimal embedding at eps = 1.5th
    percentile of pairwise distances
- name: FIt-SNE
  role: Fast t-SNE implementation used by EMBEDR for embedding the Tabula Muris marrow
    data during the sweep
  repo: https://github.com/KlugerLab/FIt-SNE
- name: numpy
  role: Supports the pairwise-distance percentile computation and array handling for
    the affinity matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:e034de082e1df364bf987a3f1461a395e511b53e989e8794b5db0335a40396cf
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cluster-annotation-confusion-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that clusters found by DBSCAN on the cell-wise optimal t-SNE embedding (built from per-cell EMBEDR-optimal perplexities) correspond to expert cell-type annotations, e.g. the seven bone marrow cell types of the Tabula Muris marrow data, by computing a confusion table of DBSCAN labels versus annotations.

## When to use

Use when you have a cell-wise (sample-wise) optimal EMBEDR embedding plus expert cell-type annotations and need to test whether unsupervised cluster structure in the embedding is biologically concordant with those annotations.

## When NOT to use

- Input is a raw feature/counts table rather than an already-computed cell-wise optimal EMBEDR embedding — the sweep and fit_samplewise_optimal steps must be completed first
- The perplexity values of interest were not included in the EMBEDR sweep, since optimal per-cell perplexities can only be selected from those used in the sweep
- You need a quantitative clustering benchmark with concrete metric values — the article's results only report a template-driven cluster count (print statement), not a validated numeric score

## Inputs

- EMBEDR_sweep results containing per-cell EMBEDR p-values across a perplexity sweep (e.g. sweep_values=[25, 100, 250])
- Tabula Muris cell-type annotations for the ~4,800 bone marrow cells (expert-annotated labels)
- Per-cell optimal perplexities selected via EMBEDR_sweep.fit_samplewise_optimal
- Cell-wise optimal t-SNE embedding coordinates of the Tabula Muris marrow cells

## Outputs

- Per-cell DBSCAN cluster labels (eps set to the 1.5th percentile of pairwise cell-cell distances)
- Confusion table of DBSCAN cluster labels versus Tabula Muris annotations saved as cluster_confusion_table.csv
- Rendered cell-wise optimal embedding saved as samplewise_optimal_embedding.png

## How to apply

After running an EMBEDR_sweep (e.g. sweep_values=[25, 100, 250]), call fit_samplewise_optimal so each cell's perplexity is the one minimizing its EMBEDR p-value, restricted to the values used in the sweep, then build the per-cell adaptive affinity matrix and generate the cell-wise optimal t-SNE embedding with the EMBEDR package (tSNE DRA). Cluster the embedding with scikit-learn DBSCAN, setting eps to the 1.5th percentile of pairwise cell-cell distances, and record per-cell cluster labels. Render the embedding with EMBEDR_sweep.plot_embedding (saved as samplewise_optimal_embedding.png) and compute the confusion table of DBSCAN labels versus the Tabula Muris annotations (saved as cluster_confusion_table.csv). Correct application is judged by the confusion table recovering clusters concordant with the annotated bone marrow cell types (Figure 7D/7E), with almost no cross-annotation overlap except granulocyte-monocyte progenitor cells.

## Related tools

- **EMBEDR** (Generates the cell-wise optimal t-SNE embedding via EMBEDR_sweep.fit_samplewise_optimal and renders it with plot_embedding) — https://github.com/ejohnson643/EMBEDR
- **scikit-learn** (Provides DBSCAN used to cluster the cell-wise optimal embedding at eps = 1.5th percentile of pairwise distances)
- **FIt-SNE** (Fast t-SNE implementation used by EMBEDR for embedding the Tabula Muris marrow data during the sweep) — https://github.com/KlugerLab/FIt-SNE
- **numpy** (Supports the pairwise-distance percentile computation and array handling for the affinity matrix)

## Examples

```
sweepObj.fit_samplewise_optimal(); sweepObj.plot_embedding()
```

## Evaluation signals

- DBSCAN detects seven clusters, each corresponding to a class of bone marrow cell types, matching the seven annotated cell types (Figure 7D/7E)
- Confusion table shows almost no overlap between DBSCAN clusters and annotations except granulocyte-monocyte progenitor cells
- Per-cell optimal perplexities used for the embedding are drawn only from the sweep values (e.g. 25, 100, 250)
- Output files cluster_confusion_table.csv and samplewise_optimal_embedding.png are produced and the embedding renders via EMBEDR_sweep.plot_embedding
- eps parameter equals the 1.5th percentile of pairwise cell-cell distances in the cell-wise optimal embedding

## Limitations

- Optimal perplexity values can only be selected from those used in the sweep, so a poorly chosen sweep grid limits the cell-wise optimal embedding
- Calculated EMBEDR p-values have finite resolution because null distributions are computed empirically; the number of nulls bounds the lower limit of p values
- The article does not report concrete quantitative values for the EMBEDR/DBSCAN clustering run — the cluster count appears only as a template variable in a print statement
- Granulocyte-monocyte progenitor cells show overlap between DBSCAN clusters and annotations, so cluster-annotation concordance is not exact for all cell types

## Evidence

- [methods] fit_samplewise_optimal: "use this `perplexity` to generate a sample-wise optimal embedding of the data.  This is again faciliated by the `EMBEDR_sweep` class method `fit_samplewise_optimal`."
- [readme] sweep-restricted perplexities: "the optimal `perplexity` values will only be selected from those used int he sweep."
- [results] cluster detection print: "print(f"After clustering, {n_db_labels} clusters were detected!")"
- [abstract] per-cell quality statistic: "EMBEDR generates a "p" value for each sample, allowing for direct comparisons of DR algorithms and facilitating optimization of algorithm hyperparameters"
