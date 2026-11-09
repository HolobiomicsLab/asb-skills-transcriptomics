---
name: guilt-by-association-feature-expansion
description: Use when a scRNA-seq clustering pipeline needs a final feature gene set
  from DUBStepR seed features, i.e., when optimise.features=TRUE and the optimal number
  of feature genes must be determined.
when_to_use_negative:
- The input is already a curated feature table or marker gene list rather than raw/normalized
  scRNA-seq counts needing feature expansion.
- The goal is differential expression testing or cell-type label assignment rather
  than unsupervised feature selection for clustering.
- Genes have not been pre-filtered (mitochondrial, ribosomal, spike-in, and pseudogenes
  should be filtered out before DUBStepR constructs the GGC matrix).
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DUBStepR
  role: Provides the DUBStepR() call with optimise.features=TRUE and getOptimalFeatureSet()
    to expand seed features and select the Density Index-optimal feature set
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Holds the log-normalized input data and receives the optimal feature gene
    set as variable features for downstream PCA/clustering
- name: R
  role: Runtime environment for DUBStepR (requires R >= 3.5.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:70ef7bd01959e6394d7caa48fbf879a3a6ed362b678bce4ddec2de340c80e46a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# guilt-by-association-feature-expansion

## Summary

Expands DUBStepR's minimally redundant stepwise-regression seed features into the final feature set by iteratively adding genes most correlated with the current set (guilt-by-association ordering), and selects the optimal feature set size by maximizing the Density Index.

## When to use

Use when a scRNA-seq clustering pipeline needs a final feature gene set from DUBStepR seed features, i.e., when optimise.features=TRUE and the optimal number of feature genes must be determined.

## When NOT to use

- The input is already a curated feature table or marker gene list rather than raw/normalized scRNA-seq counts needing feature expansion.
- The goal is differential expression testing or cell-type label assignment rather than unsupervised feature selection for clustering.
- Genes have not been pre-filtered (mitochondrial, ribosomal, spike-in, and pseudogenes should be filtered out before DUBStepR constructs the GGC matrix).

## Inputs

- Log-normalized scRNA-seq count matrix (e.g., Seurat object from the 10X pbmc_1k_v2 filtered feature-barcode matrix, normalization.method='LogNormalize')
- Gene-gene correlation (GGC) matrix from getGGC(data)
- Stepwise-regression seed feature set and ordered gene list from the DUBStepR call

## Outputs

- Guilt-by-association ordered gene list
- Optimal feature gene set (size maximizing the Density Index), e.g., written to optimal_feature_genes.csv
- Seurat variable features assignment: seuratObj@assays$RNA@var.features

## How to apply

After DUBStepR constructs the gene-gene correlation (GGC) matrix and performs stepwise regression to identify a minimally redundant set of seed features, expand the seed set by guilt-by-association: at each iteration add the candidate gene most strongly correlated with the current set from the ordered gene list. Recompute the Density Index at each candidate feature set size via getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error) and select the size that maximizes the Density Index as the optimal feature gene set. Typical call parameters on a log-normalized Seurat object (e.g., the 10X pbmc_1k_v2 matrix) are min.cells = 0.05*ncol(seuratObj), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0. Assign the resulting optimal feature gene set to the Seurat variable features slot for downstream PCA and clustering.

## Related tools

- **DUBStepR** (Provides the DUBStepR() call with optimise.features=TRUE and getOptimalFeatureSet() to expand seed features and select the Density Index-optimal feature set) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Holds the log-normalized input data and receives the optimal feature gene set as variable features for downstream PCA/clustering)
- **R** (Runtime environment for DUBStepR (requires R >= 3.5.0))

## Examples

```
DUBStepR(input.data = norm.data, min.cells = 0.05*ncol(seuratObj), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The optimal feature set size is the argmax of the Density Index curve computed by getOptimalFeatureSet across expanded feature set sizes.
- The ranked output preserves guilt-by-association ordering: each added gene is the candidate most correlated with the current feature set.
- The returned feature set is a superset of the minimally redundant stepwise-regression seed features.
- The optimal feature set is correctly assigned to seuratObj@assays$RNA@var.features and used for subsequent PCA (num.pcs = 20) and k-NN clustering (k = 10).
- Resulting clustering (e.g., silhouette index) should approach DUBStepR's maximum at roughly 200-500 feature genes on benchmark PBMC data.

## Limitations

- The elbow point computation used in feature set optimization is not described in the main text (deferred to Supp. Note).
- Key supporting results (e.g., silhouette index behavior at 200-500 feature genes) are deferred to supplementary figures not included in the main text.
- Genes must first be filtered based on a correlation range score; unfiltered inputs can degrade the guilt-by-association expansion.

## Evidence

- [intro] Seed expansion by guilt-by-association: "which are then expanded by adding correlated features (guilt-by-association)"
- [intro] Stepwise seed feature identification: "DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features"
- [methods] Optimal feature set function signature: "getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error)"
- [readme] Correlation-range filtering precedes feature prioritization: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix"
- [discussion] Expected feature gene count range: "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
