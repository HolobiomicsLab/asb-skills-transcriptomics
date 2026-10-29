---
name: silhouette-width-cluster-separation-assessment
description: Use when comparing low-dimensional representations (e.g., PCA vs ZIFA
  vs ZINB-WaVE) of scRNA-seq data for a fixed set of known cell-type labels, and you
  need a single cluster-separation score per embedding.
when_to_use_negative:
- No reliable ground-truth cell-type labels exist — silhouette widths would measure
  an arbitrary clustering rather than recovery of known biology
- Comparing clusterings on the same fixed embedding rather than comparing alternative
  embeddings/normalizations of the same data
edam_operation: http://edamontology.org/operation_3432
tools:
- name: cluster
  role: R package providing the silhouette criterion used to score cluster separation
    in each low-dimensional representation
- name: zinbwave
  role: Produces the ZINB-WaVE low-dimensional representation fitted directly on raw
    counts for one arm of the silhouette comparison
  repo: https://bioconductor.org/packages/zinbwave
- name: ZIFA
  role: Zero-inflated dimensionality reduction producing embeddings scored by silhouette
    widths after log2(1+counts) transformation
  repo: https://github.com/epierson9/ZIFA
- name: edgeR
  role: Provides TMM normalization factors for the normalization arm compared via
    silhouette widths
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:5bd3b0758b3e184787406ebb79a420337eb2e37df635cc0d642d7da9ed8fd485
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - cluster
  - edgeR
  - zinbwave
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# silhouette-width-cluster-separation-assessment

## Summary

Quantifies how well author-provided cell-type labels separate in a low-dimensional embedding by computing average silhouette widths on Euclidean distances; in the ZINB-WaVE paper this metric is used to compare PCA, ZIFA, and ZINB-WaVE across RAW, TC, TMM, and FQ normalizations.

## When to use

Use when comparing low-dimensional representations (e.g., PCA vs ZIFA vs ZINB-WaVE) of scRNA-seq data for a fixed set of known cell-type labels, and you need a single cluster-separation score per embedding.

## When NOT to use

- No reliable ground-truth cell-type labels exist — silhouette widths would measure an arbitrary clustering rather than recovery of known biology
- Comparing clusterings on the same fixed embedding rather than comparing alternative embeddings/normalizations of the same data

## Inputs

- Low-dimensional representation matrix (cells x latent dimensions) from PCA, ZIFA, or ZINB-WaVE (zinbwave W matrix)
- Author-provided per-cell type/cluster labels (integer or factor vector, one label per cell)
- scRNA-seq count matrices for V1, S1/CA1, glioblastoma, and mESC data sets from drisso/zinb_analysis

## Outputs

- Average silhouette width per data set per normalization per dimensionality-reduction method
- silhouette_widths.csv with one row per data set x normalization x method combination

## How to apply

For each data set (V1, S1/CA1, glioblastoma, mESC from drisso/zinb_analysis), build the low-dimensional representation: PCA and ZIFA on each normalization variant (RAW counts, TC scaling, TMM via edgeR, FQ), plus ZINB-WaVE via the zinbwave package (R >=3.4, Bioconductor >=3.6) fitted directly on unnormalized raw counts. Note ZIFA requires a log2(1 + counts) transform of count data before running. Then compute pairwise Euclidean distances in each low-dimensional space and evaluate the silhouette criterion with the author-provided labels, taking the mean silhouette width as the embedding score. Tabulate one row per data set per normalization per method (silhouette_widths.csv) and compare orderings: the paper's expected pattern is that normalization choice matters more than PCA-vs-ZIFA, and that ZINB-WaVE on raw counts matches the best normalization without a preliminary step.

## Related tools

- **cluster** (R package providing the silhouette criterion used to score cluster separation in each low-dimensional representation)
- **zinbwave** (Produces the ZINB-WaVE low-dimensional representation fitted directly on raw counts for one arm of the silhouette comparison) — https://bioconductor.org/packages/zinbwave
- **ZIFA** (Zero-inflated dimensionality reduction producing embeddings scored by silhouette widths after log2(1+counts) transformation) — https://github.com/epierson9/ZIFA
- **edgeR** (Provides TMM normalization factors for the normalization arm compared via silhouette widths)

## Examples

```
library(cluster); sil <- silhouette(cell_labels, dist(W)); avg_width <- mean(sil[,3])
```

## Evaluation signals

- silhouette_widths.csv exists and contains exactly one row per data set per normalization (RAW, TC, TMM, FQ) per method (PCA, ZIFA, ZINB-WaVE)
- Relative ordering reproduces the paper's reported pattern: normalization choice has a larger effect on average silhouette width than the PCA-vs-ZIFA choice
- ZINB-WaVE fitted on unnormalized raw counts attains average silhouette widths comparable to PCA/ZIFA under the best-performing normalization
- Silhouette widths fall in the valid [-1, 1] range and are computed on Euclidean distances within each low-dimensional space

## Limitations

- ZINB-WaVE optimization is non-convex, so the representation being scored may vary across runs; computational cost is nontrivial
- There is no guarantee that the low-dimensional signal extracted (and thus scored) is biologically relevant if unwanted technical variation is unaccounted for
- Results depend on the author-provided labels; with unknown or noisy labels the silhouette score can be misleading

## Evidence

- [abstract] ZINB-WaVE outperforms PCA/ZIFA without normalization: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [readme] ZIFA log transform of count data: [evidence span withheld — anchor status: external_doc]
- [discussion] Modeling raw counts without prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [intro] ZINB-WaVE gives more stable low-dimensional representation than PCA and ZIFA: [evidence span withheld — anchor status: not_found]
