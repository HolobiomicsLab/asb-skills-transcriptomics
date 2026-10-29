---
name: leiden-multiresolution-clustering
description: Use when you need multiple candidate cluster label sets for scTriangulate
  integration — no prior annotations exist and one single Leiden resolution will not
  capture all discrete cell states.
when_to_use_negative:
- Only one clustering solution or a single prior annotation exists — scTriangulate
  requires at least two conflicting annotation columns in adata.obs to mix-and-match.
- You need a joint latent-space or geometric-graph integration (e.g., Seurat-WNN,
  TotalVI) rather than decision-level reconciliation of separate per-modality clusterings.
- Data is not QC-filtered count/normalized data loaded into the software — scanpy_recipe
  assumes post-QC matrices.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: Provides scanpy_recipe preprocessing wrapper to run Leiden sequentially at
    multiple resolutions when no prior annotations exist
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Underlying engine for normalization, HVG selection, neighbor graphs, PCA,
    UMAP, and Leiden clustering within scanpy_recipe
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:c4f447862d5bb04814403a096d5b6b06b8c0e62cda385d2d394ad62a9230896a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# leiden-multiresolution-clustering

## Summary

Runs Leiden clustering sequentially at multiple resolutions (e.g., 1, 2, and 3) per modality to generate conflicting annotation columns that scTriangulate can later mix-and-match via Shapley-based integration.

## When to use

Use when you need multiple candidate cluster label sets for scTriangulate integration — no prior annotations exist and one single Leiden resolution will not capture all discrete cell states.

## When NOT to use

- Only one clustering solution or a single prior annotation exists — scTriangulate requires at least two conflicting annotation columns in adata.obs to mix-and-match.
- You need a joint latent-space or geometric-graph integration (e.g., Seurat-WNN, TotalVI) rather than decision-level reconciliation of separate per-modality clusterings.
- Data is not QC-filtered count/normalized data loaded into the software — scanpy_recipe assumes post-QC matrices.

## Inputs

- Single-cell count matrix (AnnData h5ad; e.g., 10x Multiome PBMC h5 with ~10,991 nuclei passing QC)
- QC-filtered AnnData (RNA min_genes>=300, min_counts>=500, pct_counts_mt<=20%; >=1000 ATAC peaks per nucleus)
- Resolution list for Leiden (paper used resolutions 1, 2, 3 per modality)

## Outputs

- adata.obs annotation columns per resolution per modality (e.g., 'sctri_rna_leiden_1', 'sctri_rna_leiden_2', 'sctri_rna_leiden_3')
- Neighbor graphs and dimensionality reductions (adata.obsm['X_pca'] / ATAC reduction, adata.obsm['X_umap'])
- Hypervariable-feature-selected, log CPTT (RNA) / CLR (other) normalized AnnData ready for ScTriangulate.query

## How to apply

Normalize RNA to log CPTT (or CLR for ADT/other modalities) with scTriangulate.preprocessing.Normalization, select hypervariable features (RNA top 3000 genes; ATAC top 100,000 peaks), and call scTriangulate.preprocessing.scanpy_recipe to build neighbor graphs (n_neighbors=15), reduce dimensions (RNA n_pc=15, ATAC n_pc=100), and run Leiden at resolutions 1, 2, and 3 for each modality. Each resolution produces its own annotation column in adata.obs (e.g., 'sctri_rna_leiden_1', 'sctri_rna_leiden_2', 'sctri_rna_leiden_3') plus adata.obsm['X_umap'] for visualization. Concatenate modality-specific AnnData objects with concat_rna_and_other if multimodal. The rationale, per the paper, is that different informative populations emerge at different resolutions, so the resolution ensemble becomes the 'players' that scTriangulate's Shapley-based lazy_run reconciles (e.g., win_fraction_cutoff=0.35, add_metrics='tfidf5' in the multiome PBMC analysis with scTriangulate 0.9.0 and scanpy 1.7.2).

## Related tools

- **scTriangulate** (Provides scanpy_recipe preprocessing wrapper to run Leiden sequentially at multiple resolutions when no prior annotations exist) — https://github.com/frankligy/scTriangulate
- **scanpy** (Underlying engine for normalization, HVG selection, neighbor graphs, PCA, UMAP, and Leiden clustering within scanpy_recipe) — https://github.com/frankligy/scTriangulate

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- adata.obs contains at least two annotation columns (one per Leiden resolution/modality) with non-identical label distributions — a single resolution column means the ensemble step was skipped.
- adata.obsm['X_umap'] exists so lazy_run can auto-generate visualizations; scanpy_recipe should have produced it.
- Resolution grid matches the paper's setup (Leiden resolutions 1, 2, 3 for each modality) with RNA n_pc=15 / ATAC n_pc=100 and n_neighbors=15.
- Downstream lazy_run outputs (scTriangulate results dir, e.g., metrics CSV with Homogeneity/Completeness/V-Measure) reproduce reported superiority over individual clustering predictions (Supplementary Fig. 5d).
- QC thresholds enforced before clustering: RNA min_genes<300, min_counts<500, pct_counts_mt>20% filtered; >=1000 ATAC peaks per nucleus required.

## Limitations

- Running Leiden at many resolutions plus dozens of annotation sources significantly increases scTriangulate runtime; over 15 annotation sets triggers automatic substitution of Shapley selection with combined rank-based selection.
- No single resolution will identify the most discrete cell states across modalities, but programmatic default settings may not be optimal for all study designs.
- Each modality's clustering is only as good as its QC and normalization choice (log CPTT for RNA, CLR for ADT); unfiltered matrices propagate into the resolution ensemble.

## Evidence

- [intro] resolution rationale: "It is likely that no single-resolution in single-cell analyses will identify the most discrete cell states across different modalities"
- [methods] leiden via scTriangulate: "This step may also include running Leiden clustering sequentially through scTriangulate when no prior annotations exist."
- [methods] versions: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [readme] readme usage: "Integrate results from the same or multiple unsupervised clustering algorithms (i.e. Leiden, Seurat, SnapATAC) using different resolutions."
- [readme] readme query requirement: [evidence span withheld — anchor status: external_doc]
