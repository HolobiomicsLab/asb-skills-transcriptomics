---
name: dbscan-density-clustering-of-embeddings
description: Use when you have a dimensionality-reduced embedding of single-cell data
  (e.g., an EMBEDR sample-wise optimal t-SNE) plus independent annotations, and you
  want to validate that density-separated clusters in the 2D embedding recover known
  biological classes.
when_to_use_negative:
- No EMBEDR perplexity sweep has been run — fit_samplewise_optimal can only be carried
  out after a sweep, and optimal perplexities are selected only from sweep values.
- The input is raw counts or a high-dimensional feature matrix rather than a fitted
  2D embedding; DBSCAN here is for validating embedding structure, not for primary
  clustering of expression data.
- You need a quantitative clustering metric rather than a qualitative label-vs-annotation
  confusion table — the article reports the DBSCAN run without concrete quantitative
  values beyond cluster counts.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: EMBEDR
  role: Generates per-cell EMBEDR p-values over the perplexity sweep and the cell-wise
    optimal t-SNE embedding that DBSCAN clusters (EMBEDR_sweep.fit_samplewise_optimal,
    plot_embedding).
  repo: https://github.com/ejohnson643/EMBEDR
- name: scikit-learn
  role: Provides the DBSCAN implementation used to cluster the embedding; a required
    EMBEDR dependency.
- name: numpy
  role: Computes pairwise cell-cell distances from which the 1.5th-percentile DBSCAN
    eps is set; a required EMBEDR dependency.
- name: scipy
  role: Supports distance/affinity computations for the per-cell adaptive affinity
    matrix; a required EMBEDR dependency.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:0d552b8b9789246350b4d471ca65ce10b80d81a4399c866f3a4db8b164348c89
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
  - Miscell
  - Python
  - Scanpy
  - Seurat
  - anndata
  - numpy
  - scVI
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- dbscan-density-clustering-on-embeddings
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# DBSCAN density clustering of embeddings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters a cell-wise optimal t-SNE embedding (built from EMBEDR per-cell p-values over a perplexity sweep) with scikit-learn DBSCAN to test whether embedding structure corresponds to expert cell-type annotations, as done for ~4,800 Tabula Muris bone marrow cells.

## When to use

Use when you have a dimensionality-reduced embedding of single-cell data (e.g., an EMBEDR sample-wise optimal t-SNE) plus independent annotations, and you want to validate that density-separated clusters in the 2D embedding recover known biological classes.

## When NOT to use

- No EMBEDR perplexity sweep has been run — fit_samplewise_optimal can only be carried out after a sweep, and optimal perplexities are selected only from sweep values.
- The input is raw counts or a high-dimensional feature matrix rather than a fitted 2D embedding; DBSCAN here is for validating embedding structure, not for primary clustering of expression data.
- You need a quantitative clustering metric rather than a qualitative label-vs-annotation confusion table — the article reports the DBSCAN run without concrete quantitative values beyond cluster counts.

## Inputs

- EMBEDR_sweep results containing per-cell EMBEDR p-values across the perplexity sweep (e.g., sweep_values=[25, 100, 250])
- Tabula Muris cell-type annotations for the ~4,800 bone marrow cells (from the czbiohub-sf/tabula-muris data ingest)
- Cell-wise optimal t-SNE embedding generated via EMBEDR_sweep.fit_samplewise_optimal (t-SNE DRA)

## Outputs

- Per-cell DBSCAN cluster labels (integer label per cell)
- samplewise_optimal_embedding.png rendered via EMBEDR_sweep.plot_embedding
- cluster_confusion_table.csv (DBSCAN labels vs. Tabula Muris annotations)

## How to apply

First run an EMBEDR_sweep over perplexity values (e.g., sweep_values=[25, 100, 250]) on the data, then call EMBEDR_sweep.fit_samplewise_optimal so each cell's perplexity is the one minimizing its EMBEDR p-value, restricted to the sweep values, and build the per-cell adaptive affinity matrix for the cell-wise optimal t-SNE embedding. Cluster that embedding with scikit-learn DBSCAN, setting eps to the 1.5th percentile of pairwise cell-cell distances, and record per-cell cluster labels. Render the sample-wise optimal embedding with EMBEDR_sweep.plot_embedding (saved as samplewise_optimal_embedding.png) and compute the confusion table of DBSCAN labels versus annotations (saved as cluster_confusion_table.csv). The rationale is that DBSCAN detects density-defined clusters without prespecifying cluster count, so concordance with annotations demonstrates the embedding's structure is biologically real rather than algorithm-induced distortion.

## Related tools

- **EMBEDR** (Generates per-cell EMBEDR p-values over the perplexity sweep and the cell-wise optimal t-SNE embedding that DBSCAN clusters (EMBEDR_sweep.fit_samplewise_optimal, plot_embedding).) — https://github.com/ejohnson643/EMBEDR
- **scikit-learn** (Provides the DBSCAN implementation used to cluster the embedding; a required EMBEDR dependency.)
- **numpy** (Computes pairwise cell-cell distances from which the 1.5th-percentile DBSCAN eps is set; a required EMBEDR dependency.)
- **scipy** (Supports distance/affinity computations for the per-cell adaptive affinity matrix; a required EMBEDR dependency.)

## Examples

```
sweepObj.fit_samplewise_optimal(); sweepObj.plot_embedding(); from sklearn.cluster import DBSCAN; labels = DBSCAN(eps=np.percentile(pdist(emb, 'euclidean'), 1.5)).fit_predict(emb)
```

## Evaluation signals

- The confusion table recovers seven DBSCAN clusters concordant with the seven annotated bone marrow cell types (Figure 7D/7E).
- eps for DBSCAN equals the 1.5th percentile of pairwise cell-cell distances in the cell-wise optimal embedding.
- Per-cell optimal perplexities used for the embedding are drawn only from the sweep values (e.g., 25, 100, 250).
- Almost no overlap between DBSCAN clusters and annotations, except granulocyte-monocyte progenitor cells.
- The rendered figure samplewise_optimal_embedding.png uses the sample-wise optimal embedding, and cluster_confusion_table.csv exists with DBSCAN label rows vs. annotation columns.

## Limitations

- Calculated EMBEDR p-values have finite resolution because null distributions are computed empirically; the number of nulls bounds the lower limit of p values, which affects which perplexity is chosen per cell.
- The sample-wise optimal embedding can only be built after a sweep, and optimal perplexity values will only be selected from those used in the sweep.
- Concordance is imperfect: granulocyte-monocyte progenitor cells overlap between DBSCAN clusters and annotations, and the article reports no quantitative cluster metrics beyond the printed cluster count.

## Evidence
