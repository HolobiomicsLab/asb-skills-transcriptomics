---
name: cell-type-proportion-quantification
description: Use when you have raw scRNA-seq UMI counts from a heterogeneous cell
  suspension (e.g. 68k PBMCs) and need cluster-level cell-type composition percentages
  validated against a published population breakdown.
when_to_use_negative:
- When unwanted technical variation (batch/plate effects) is not modeled — the low-rank
  W matrix may capture confounding effects instead of cell identity
- When you need supervised differential expression per cluster — the article states
  this capability is only envisioned for a future zinbwave version
- When a preliminary normalization step is already applied and desired as the primary
  basis — the method is designed to model raw counts directly
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model on raw PBMC counts and infers
    the K=10 latent-factor W matrix used for clustering
  repo: https://bioconductor.org/packages/zinbwave
- name: ZINB-WaVE
  role: The dimensionality-reduction method (replacing PCA) whose low-dimensional
    representation defines the clustering and t-SNE space
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Provides access to the 10x Genomics 68k PBMC UMI count data set
- name: scater
  role: 'Optional: explore correlation between estimated latent factors and QC measures,
    and optionally include QC measures as model covariates'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:b2bdc903a35abc0dea966a8693270160007579c448a0e1f8c6a65f43670277e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZINB-WaVE
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-type-proportion-quantification

## Summary

Clusters single-cell RNA-seq data (10x Genomics 68k PBMC UMI counts) in a ZINB-WaVE low-dimensional space and tabulates the fraction of cells per annotated population, reproducing the Zheng et al. composition (~80% T-cells, 6% NK, 5% B-cells, 3% CD14+ monocytes, etc.).

## When to use

Use when you have raw scRNA-seq UMI counts from a heterogeneous cell suspension (e.g. 68k PBMCs) and need cluster-level cell-type composition percentages validated against a published population breakdown.

## When NOT to use

- When unwanted technical variation (batch/plate effects) is not modeled — the low-rank W matrix may capture confounding effects instead of cell identity
- When you need supervised differential expression per cluster — the article states this capability is only envisioned for a future zinbwave version
- When a preliminary normalization step is already applied and desired as the primary basis — the method is designed to model raw counts directly

## Inputs

- 10x Genomics 68k PBMC UMI count matrix (raw counts, via scRNAseq / drib package ecosystem)
- R/Bioconductor environment (R >=3.4, Bioconductor >=3.6)

## Outputs

- pbmc_zinbwave_clusters.csv with cluster assignments and per-cluster cell proportions
- Cells x 10 ZINB-WaVE W matrix (low-dimensional representation)
- t-SNE embedding of cell-to-cell Euclidean distances in the W space (Fig. 3c analogue)

## How to apply

Load the 10x Genomics 68k PBMC UMI count matrix (scRNAseq/Bioconductor ecosystem) into R (>=3.4) / Bioconductor (>=3.6), then run zinbwave on raw counts with K=10 latent factors — no prior normalization is needed because the cell-level intercept acts as a global-scaling normalization factor. Extract the cells x 10 W matrix, build a shared-nearest-neighbor graph on W, and cluster Seurat-style. Compute Euclidean cell-to-cell distances in the ZINB-WaVE space and embed with t-SNE (as in Fig. 3c) for visualization. Tabulate cells per cluster and annotate the major and rare populations (CD4+/CD8+ T-cells, cytotoxic T-cells, NK, B-cells, CD14+/CD16+ monocytes, dendritic cells, megakaryocytes), writing assignments and proportions to pbmc_zinbwave_clusters.csv.

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model on raw PBMC counts and infers the K=10 latent-factor W matrix used for clustering) — https://bioconductor.org/packages/zinbwave
- **ZINB-WaVE** (The dimensionality-reduction method (replacing PCA) whose low-dimensional representation defines the clustering and t-SNE space) — https://github.com/drisso/zinbwave
- **scRNAseq** (Provides access to the 10x Genomics 68k PBMC UMI count data set)
- **scater** (Optional: explore correlation between estimated latent factors and QC measures, and optionally include QC measures as model covariates)

## Examples

```
BiocManager::install("zinbwave"); res <- zinbwave(pbmc_counts, K = 10); W <- assay(res, "normalizedValues")
```

## Evaluation signals

- Cluster proportions within a few percentage points of the published composition: ~80% T-cells, 9% activated cytotoxic T-cells, 6% NK, 5% B-cells, 3% CD14+ monocytes, 2.5% CD16+ monocytes, 1% CD1C+ DC, 0.5% pDC, 0.2% megakaryocytes
- Clusters 0–1 annotate as CD4+ T-cells and Clusters 2–6 as CD8+ T-cells, together covering ~80% of cells
- t-SNE embedding computed on Euclidean distances in the W space visually separates the annotated major populations (Fig. 3c analogue)
- Output file pbmc_zinbwave_clusters.csv contains one row per cell with cluster label and per-cluster proportion sums to 1
- zinbwave was run on raw (unnormalized) counts with the cell-level intercept enabled as global-scaling normalization

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant — unaccounted technical variation will be captured in W
- Computational cost from non-convex likelihood optimization, though computing time is approximately linear in cells/genes and quadratic in latent factors
- Supervised differential expression capability is not yet available in zinbwave

## Evidence

- [abstract] we present a general and flexible zero-inflated negative binomial model (ZINB-WaVE): "we present a general and flexible zero-inflated negative binomial model (ZINB-WaVE)"
- [methods] Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6).: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [discussion] An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE.: "An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE."
- [discussion] However, there is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant: "However, there is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant"
- [readme] To exactly reproduce the analyses of the paper, version `0.1.1` of the `zinbwave` package is required.: [evidence span withheld — anchor status: external_doc]
