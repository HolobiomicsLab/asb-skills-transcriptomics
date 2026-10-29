---
name: highly-variable-gene-selection
description: Use when a preprocessed scRNA-seq expression matrix (cells x genes, log2(x+1)
  normalized, genes in ≥3 cells retained) needs dimensionality reduction to a gene
  subset that preserves signal for rare cell type detection.
when_to_use_negative:
- Input is raw unnormalized counts without size-factor normalization and the genes-in-≥3-cells
  filter — the dispersion ranking is not comparable to the published pipeline
- The goal is differential expression testing or marker discovery rather than feature
  selection for clustering/rare cell detection
- Only a small targeted gene panel is available, where mean-dispersion binning across
  the genome is meaningless
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: Pipeline whose ensemble feature selection consumes the top 2000 HVG set merged
    with the top 2000 random-forest-importance genes
  repo: github.com/xuyp-csu/scCAD
- name: Seurat
  role: Reference implementation of the mean vs dispersion HVG selection with mean-expression
    binning that this skill reproduces
- name: Scanpy
  role: Python dependency (v1.9.1 in scCAD requirements.txt) providing the Seurat-style
    HVG computation in the scCAD environment
- name: scikit-learn
  role: Provides the random forest classifier (Gini impurity importance) for the companion
    top 2000 HIG arm of the ensemble
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:8ed93ef4a3ef395bf64f190a554b1a2f3168e4774a29722457a3c9cbc5fa9cb4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - Scanpy
  - Seurat
  - scCAD
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# highly-variable-gene-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select the top 2000 highly variable genes (HVG) from a log-normalized scRNA-seq matrix using the Seurat-style mean vs dispersion approach with expression-binning, as the first arm of scCAD's ensemble feature selection for rare cell type identification.

## When to use

Use when a preprocessed scRNA-seq expression matrix (cells x genes, log2(x+1) normalized, genes in ≥3 cells retained) needs dimensionality reduction to a gene subset that preserves signal for rare cell type detection.

## When NOT to use

- Input is raw unnormalized counts without size-factor normalization and the genes-in-≥3-cells filter — the dispersion ranking is not comparable to the published pipeline
- The goal is differential expression testing or marker discovery rather than feature selection for clustering/rare cell detection
- Only a small targeted gene panel is available, where mean-dispersion binning across the genome is meaningless

## Inputs

- Preprocessed log-normalized scRNA-seq expression matrix (cells x genes), e.g. an .h5 file such as 1%Jurkat.h5 with X, gene names, and cell names
- Louvain cluster labels or cell type annotations (required only for the companion HIG arm of the ensemble)
- Gene filter already applied: genes expressed in at least three cells

## Outputs

- List of the top 2000 highly variable gene names (HVG set)
- Merged, de-duplicated ensemble gene set (≤4000 unique genes from HVG ∪ HIG), e.g. written to selected_genes.csv

## How to apply

Start from the log-normalized matrix produced by cell-specific size-factor normalization with a pseudo-count of 1 followed by log2(x+1) transformation, having already dropped genes expressed in fewer than three cells. Compute each gene's mean expression and dispersion (variance/mean) across all cells, bin genes with similar mean expression as in the Seurat approach, and retain the top 2000 most variable genes. Use Scanpy/Seurat conventions for the mean-dispersion binning so the ranking is comparable to the published pipeline. The output HVG set is then merged with the top 2000 random-forest Gini-importance genes (HIG) and de-duplicated to form scCAD's ensemble feature set, so keep gene names intact for the set union. As a benchmark, HVG-only selection retained on average 67.80% of rare cell type-specific reference DE genes (OR1 overlap rate), lower than HIG (80.20%) and the scCAD ensemble (86.75%).

## Related tools

- **scCAD** (Pipeline whose ensemble feature selection consumes the top 2000 HVG set merged with the top 2000 random-forest-importance genes) — github.com/xuyp-csu/scCAD
- **Seurat** (Reference implementation of the mean vs dispersion HVG selection with mean-expression binning that this skill reproduces)
- **Scanpy** (Python dependency (v1.9.1 in scCAD requirements.txt) providing the Seurat-style HVG computation in the scCAD environment)
- **scikit-learn** (Provides the random forest classifier (Gini impurity importance) for the companion top 2000 HIG arm of the ensemble)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('data.h5ad'); sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='seurat', bin_n=20); hvg = list(adata.var_names[adata.var['highly_variable']])
```

## Evaluation signals

- The selected HVG set contains exactly 2000 genes ranked by mean-binned dispersion
- The merged ensemble gene list (HVG ∪ HIG after deduplication) contains at most 4000 unique genes and is parseable as a gene-name table (selected_genes.csv)
- OR1 overlap rate with a Wilcoxon-derived reference DE set (Seurat FindMarkers, FDR 0.05, |log2FC| cutoff 1.5) falls near the reported HVG average of 67.80% (vs 86.75% for the full scCAD ensemble)
- OR2 = |S_ref ∩ S_select| / |S_ref ∪ S_select| and OR3 = |S_ref ∩ S_select| / |S_select| are computable and finite for each dataset
- Per-dataset values aggregate to averages matching Supplementary Table 8 figures when written to feature_selection_overlap_rates.csv

## Limitations

- HVG-only selection under-preserves rare cell type-specific genes relative to HIG and the ensemble (67.80% vs 80.20% and 86.75% average OR1), so it should not be used alone when rare cell signals matter
- Gene version packages with slightly different dependencies (Scanpy, scikit-learn, numpy versions pinned in scCAD's requirements.txt) can shift the dispersion ranking; the README pins e.g. Scanpy 1.9.1 and scikit-learn 1.0.2
- Mean-dispersion HVG selection is unsupervised and may miss low-variance but rare-type-specific marker genes, which is why scCAD merges it with random-forest HIG

## Evidence

- [methods] Ensemble feature selection: top 2000 highly variable genes plus top 2000 random-forest-importance genes: [evidence span withheld — anchor status: not_found]
