---
name: single-cell-count-matrix-quality-control
description: Use when raw 10x Genomics single-cell or multiome (RNA+ATAC) count matrices
  (h5/CSV) must be filtered and normalized (log CPTT for RNA, CLR for ADT) before
  Leiden clustering and scTriangulate integration.
when_to_use_negative:
- Input is already a set of conflicting cluster annotations in adata.obs (go straight
  to ScTriangulate.lazy_run instead of re-running QC)
- Data are images/spatial counts requiring Squidpy-style processing rather than 10x
  count-matrix QC
- Cells have already been QC'd and normalized (e.g. preprocessed matrices from Synapse
  syn26320732) — re-filtering would change cell counts vs. the reference
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scanpy 1.7.2
  role: Core engine for loading h5 counts, per-cell filtering (n_genes, counts, pct_counts_mt),
    HVG selection, and graph/UMAP construction via scanpy_recipe
  repo: https://github.com/frankligy/scTriangulate
- name: scTriangulate 0.9.0
  role: Provides preprocessing.scanpy_recipe (QC/filter/scale wrapper), Normalization.CLR_normalization
    for ADT, and concat_rna_and_other for merging RNA+ATAC AnnData after QC
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:50f42181ec0867f4dc38aaf25d351e77f54b8f7aa8d916ac361e055ca86f2628
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - scTriangulate
  - scTriangulate 0.9.0
  - scanpy
  - scanpy 1.7.2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Single-cell count matrix quality control

## Summary

Modality-specific QC of raw single-cell count matrices (10x h5 CSV/HDF5 inputs) before clustering and scTriangulate integration: RNA filters (min_genes<300, min_counts<500, pct_counts_mt>20%) and ATAC filter (>=1000 peaks per nucleus), yielding ~10,991 retained multiome PBMC nuclei.

## When to use

Use when raw 10x Genomics single-cell or multiome (RNA+ATAC) count matrices (h5/CSV) must be filtered and normalized (log CPTT for RNA, CLR for ADT) before Leiden clustering and scTriangulate integration.

## When NOT to use

- Input is already a set of conflicting cluster annotations in adata.obs (go straight to ScTriangulate.lazy_run instead of re-running QC)
- Data are images/spatial counts requiring Squidpy-style processing rather than 10x count-matrix QC
- Cells have already been QC'd and normalized (e.g. preprocessed matrices from Synapse syn26320732) — re-filtering would change cell counts vs. the reference

## Inputs

- 10x Genomics Multiome PBMC h5 (RNA+ATAC combined sparse matrix counts)
- Raw scRNA-seq count matrices (e.g. GEO GSE117825, pbmc_10k_v3) as CSV/H5AD
- ADT/ADT count CSV files (e.g. NCBI GEO GSM4949911) for CLR normalization

## Outputs

- QC-filtered, log CPTT-normalized RNA AnnData object with HVGs, PCA neighbors, and Leiden cluster columns (resolutions 1/2/3)
- QC-filtered ATAC (top 100,000 peaks) AnnData object for concat_rna_and_other
- Cleaned AnnData ready for scTriangulate lazy_run (no adata.raw, X_umap in obsm)

## How to apply

Load raw counts with scanpy/scTriangulate (version 0.9.0 with scanpy 1.7.2 per the paper) via the scTriangulate.preprocessing.scanpy_recipe wrapper, which handles QC, filtering, and scaling. Apply the RNA filters min_genes<300, min_counts<500, and pct_counts_mt>20% (drop cells failing them), and for ATAC require at least 1000 peaks per nucleus; on the 10x Multiome PBMC h5 dataset this retains ~10,991 nuclei. Normalize each modality to its modality-appropriate space (logarithmic CPTT for RNA; CLR normalization for ADT via scTriangulate.preprocessing.Normalization.CLR_normalization), select hypervariable features (RNA top 3000 genes, ATAC top 100,000 peaks), build neighbor graphs (n_neighbors=15) and reduced dimensions (RNA 15 PCs, ATAC 100 PCs). The rationale is that unstable or low-quality cells/bins would corrupt the Shapley-based mix-and-match of Leiden clusterings at resolutions 1, 2, and 3 downstream. Ensure the resulting AnnData has adata.X normalized, no adata.raw attribute, and UMAP coordinates in adata.obsm['X_umap'] before passing it to scTriangulate.

## Related tools

- **scanpy 1.7.2** (Core engine for loading h5 counts, per-cell filtering (n_genes, counts, pct_counts_mt), HVG selection, and graph/UMAP construction via scanpy_recipe) — https://github.com/frankligy/scTriangulate
- **scTriangulate 0.9.0** (Provides preprocessing.scanpy_recipe (QC/filter/scale wrapper), Normalization.CLR_normalization for ADT, and concat_rna_and_other for merging RNA+ATAC AnnData after QC) — https://github.com/frankligy/scTriangulate

## Examples

```
python -c "import scanpy as sc; adata = sc.read_10x_h5('pbmc.h5'); sc.pp.filter_cells(adata, min_genes=300); sc.pp.filter_cells(adata, min_counts=500); adata = adata[adata.obs.pct_counts_mt < 20].copy()"
```

## Evaluation signals

- Retained nuclei count on the 10x Multiome PBMC dataset matches the reported ~10,991 after applying the RNA and ATAC filters
- Downstream ScTriangulate V-Measure/Completeness in multiome_pbmc_sctriangulate_metrics.csv shows superiority over individual Leiden clustering predictions (Supplementary Fig. 5d)
- adata.X is verified as properly normalized (log CPTT for RNA, CLR for ADT) and adata.raw is absent before lazy_run
- QC'd objects contain the expected feature spaces: RNA top 3000 HVGs and ATAC top 100,000 peaks

## Limitations

- Programmatic default analytical/QC settings may not be optimal for all study designs, as the authors note these were largely programmatic defaults
- Thresholds are dataset-tuned (multiome PBMC); applying the same min_genes/mt/peak cutoffs to other tissues or chemistries may over- or under-filter
- QC on joint RNA+ATAC requires >=1000 peaks per nucleus; nuclei failing this ATAC criterion are lost even if RNA quality is acceptable

## Evidence

- [methods] QC thresholds for multiome PBMC: [evidence span withheld — anchor status: not_found]
- [methods] Tool versions for preprocessing: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [methods] QC wrapper function: "counts or normalized data matrices are loaded in the software for QC, filtering, and scaling (optional)."
- [readme] Normalization expectation on adata.X: [evidence span withheld — anchor status: external_doc]
- [methods] Concatenation step after QC: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
