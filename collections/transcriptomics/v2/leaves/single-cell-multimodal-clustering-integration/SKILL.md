---
name: single-cell-multimodal-clustering-integration
description: Use when you have multimodal single-cell data (e.g. CITE-Seq, Multiome,
  TEA-Seq) with multiple conflicting annotation sets from Leiden clustering at different
  resolutions, and need a reconciled per-cell population assignment.
when_to_use_negative:
- When only one modality with a single resolution clustering exists and no conflicting
  annotations need reconciliation
- When more than 15 annotation sets are provided and runtime matters — scTriangulate
  substitutes Shapley selection with combined rank-based selection
- When you need integration of multiple samples of the same modality or across technologies
  — the article notes this support is planned but not yet available
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scTriangulate
  role: 'Core decision-level integration engine: Shapley-value mix-and-match of conflicting
    cluster annotations with stability pruning (lazy_run, win_fraction_cutoff=0.25)'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Per-modality preprocessing and Leiden clustering via scanpy_recipe (log-CPTT
    RNA, CLR ADT, HVG selection, PCA/UMAP)
- name: Seurat-WNN
  role: Baseline multimodal integration benchmark (3-way weighted nearest neighbor
    on RNA, ADT, ATAC with Seurat v4 FindClusters)
- name: TotalVI
  role: Baseline RNA+ADT variational autoencoder joint latent space benchmark (scvi-tools
    0.14.3, 400 epochs)
- name: ClusterEnsembles
  role: Baseline ensemble clustering (HGPA, MCLA, HBGF with default options, CSPA
    excluded) on the 9 Leiden results
  repo: https://github.com/tsano430/ClusterEnsembles
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:134075d7ac12278972e1c811641d74f742aa8e5e983b8dc7728ede8b5065abc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - Seurat-WNN
  - TotalVI
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# single-cell-multimodal-clustering-integration

## Summary

A decision-level strategy for reconciling conflicting single-cell clustering solutions across modalities (RNA, ATAC, ADT), algorithms, and resolutions using scTriangulate's Shapley-value coalitional framework plus stability metrics, benchmarked against Seurat-WNN, TotalVI, and ensemble clustering (MCLA, HGPA, HBGF).

## When to use

Use when you have multimodal single-cell data (e.g. CITE-Seq, Multiome, TEA-Seq) with multiple conflicting annotation sets from Leiden clustering at different resolutions, and need a reconciled per-cell population assignment.

## When NOT to use

- When only one modality with a single resolution clustering exists and no conflicting annotations need reconciliation
- When more than 15 annotation sets are provided and runtime matters — scTriangulate substitutes Shapley selection with combined rank-based selection
- When you need integration of multiple samples of the same modality or across technologies — the article notes this support is planned but not yet available

## Inputs

- Multimodal count matrices (RNA-ATAC combined h5 files and ADT count CSV, e.g. from NCBI GEO GSM4949911 TEA-Seq)
- Normalized AnnData with log-CPTT RNA, CLR-normalized ADT, log-CPTT ATAC, and obsm['X_umap']
- ≥2 conflicting annotation columns in adata.obs (e.g. Leiden r=1,2,3 per modality)
- Optional reference annotation (e.g. Azimuth human-pbmc level-2) for benchmarking

## Outputs

- Final per-cell scTriangulate population assignments with reconciled cluster labels
- Stability-pruned annotations (unstable clusters flagged at winning fraction < 0.25)
- UMAP visualizations colored by scTriangulate labels
- Homogeneity/Completeness/V-Measure table comparing all clustering solutions against the reference

## How to apply

Normalize each modality separately in scanpy 1.7.2 — RNA log-CPTT with top-3000 hypervariable genes and 15 PCs, ATAC log-CPTT with top-60000 peaks and 100 PCs, ADT CLR normalization with 15 PCs — and run Leiden clustering at resolutions r=1,2,3 per modality, then merge AnnData objects with scTriangulate.preprocessing.concat_rna_and_other. Run scTriangulate (v0.9.0) lazy_run on the combined AnnData, passing all 9 conflicting annotation columns via the query argument, using win_fraction_cutoff=0.25 so clusters with winning fraction < 0.25 are labeled unstable; each annotation set is treated as a player and the Shapley value selects the most stable combination rather than a consensus. Unstable cells are pruned by fraction of final versus original assignment (abs_thresh minimum cluster size default 10) and re-classified by nearest-neighbor. Benchmark against Seurat 3-way WNN, TotalVI (scvi-tools, RNA+ADT), and ClusterEnsembles (HGPA, MCLA, HBGF; CSPA excluded) by computing Homogeneity, Completeness, and V-Measure against a reference such as Azimuth human-pbmc level-2 annotations.

## Related tools

- **scTriangulate** (Core decision-level integration engine: Shapley-value mix-and-match of conflicting cluster annotations with stability pruning (lazy_run, win_fraction_cutoff=0.25)) — https://github.com/frankligy/scTriangulate
- **scanpy** (Per-modality preprocessing and Leiden clustering via scanpy_recipe (log-CPTT RNA, CLR ADT, HVG selection, PCA/UMAP))
- **Seurat-WNN** (Baseline multimodal integration benchmark (3-way weighted nearest neighbor on RNA, ADT, ATAC with Seurat v4 FindClusters))
- **TotalVI** (Baseline RNA+ADT variational autoencoder joint latent space benchmark (scvi-tools 0.14.3, 400 epochs))
- **ClusterEnsembles** (Baseline ensemble clustering (HGPA, MCLA, HBGF with default options, CSPA excluded) on the 9 Leiden results) — https://github.com/tsano430/ClusterEnsembles

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- scTriangulate attains the highest V-Measure relative to Seurat-WNN, TotalVI, and MCLA/HGPA/HBGF baselines on the TEA-Seq benchmark against Azimuth L2 reference
- Final annotation set recovers rare populations (NK CD56+ bright, B-memory, HSCP)
- Clusters with winning fraction < 0.25 are flagged unstable and re-classified via nearest-neighbor
- QC filter retains nuclei with min_gene >= 300, min_counts >= 500, pct_counts_mt <= 20%, min_atac_peaks >= 1000 (8,213 nuclei for TEA-Seq)
- Output directory contains reconciled annotations, per-cell labels, and marker gene tables from lazy_run

## Limitations

- Integration of cluster predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime; over 15 annotation sets triggers rank-based selection instead of Shapley
- Programmatic default analytical settings may not be optimal for all study designs
- Population naming of the resulting clusters remains largely non-standardized

## Evidence

- [abstract] Shapley value selects stable solutions: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [intro] Decision-level integration: "While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5), scTriangulate integrates results at a decision level"
- [methods] Versioned preprocessing: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [methods] Concatenation function: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
- [methods] Winning fraction cutoff: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] Benchmark baselines: "using the Python package ClusterEnsembles with default options (https://github.com/tsano430/ClusterEnsembles)."
- [readme] README minimum example: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])"
- [discussion] Runtime caveat: "the integration of cluster predictions from dozens of sources (e.g., resolutions, modalities, reference annotations), will significantly increase runtime"
