---
name: cross-method-batch-correction-benchmarking
description: Use when a scRNA-seq dataset with a batch/condition covariate (e.g. control
  vs interferon-beta stimulated PBMCs) has cell embeddings from GeneVector and you
  need to quantify batch alignment versus preservation of treatment-related transcriptional
  variation across correction methods.
when_to_use_negative:
- Batches differ substantially in cell type composition — the paper notes batch correction
  is challenging/unreliable in this setting, and users must manually verify correction
  vectors against cell type signatures
- The goal is to remove the biological condition signal itself (e.g. stimulation status)
  rather than technical batch effects — here the point is to preserve interferon-stimulated
  variation
- Input is already a corrected/normalized feature table with no embedding space —
  vector arithmetic requires GeneVector gene/cell embeddings
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Provides the cell embedding and batch_correct operation (V_reference − V_batch
    vector arithmetic added per cell) being benchmarked
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Recomputes UMAP embeddings on corrected/uncorrected representations and plots
    the comparison panels
- name: AnnData
  role: Container for raw counts in .X and batch/cell-type annotations in .obs used
    for silhouette scoring
- name: PyTorch
  role: Backend for GeneVector's neural embedding training underlying the corrected
    cell representations
- name: NumPy/SciPy
  role: Vector arithmetic on embeddings and silhouette coefficient computation over
    batch labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:4b9969fec4f1c71f13e65071e8ef375e2fb721d66b56c754264bf9fe72e42508
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - NumPy/SciPy
  - PyTorch
  - Scanpy
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-method-batch-correction-benchmarking

## Summary

Benchmarks GeneVector embedding-based batch correction against alternative methods (e.g. Harmony) by recomputing UMAPs on corrected and uncorrected cell representations and scoring per-cell-type batch silhouette coefficients, while checking that biological variation such as interferon-stimulated (ISG) programs in myeloid cells is preserved rather than removed.

## When to use

Use when a scRNA-seq dataset with a batch/condition covariate (e.g. control vs interferon-beta stimulated PBMCs) has cell embeddings from GeneVector and you need to quantify batch alignment versus preservation of treatment-related transcriptional variation across correction methods.

## When NOT to use

- Batches differ substantially in cell type composition — the paper notes batch correction is challenging/unreliable in this setting, and users must manually verify correction vectors against cell type signatures
- The goal is to remove the biological condition signal itself (e.g. stimulation status) rather than technical batch effects — here the point is to preserve interferon-stimulated variation
- Input is already a corrected/normalized feature table with no embedding space — vector arithmetic requires GeneVector gene/cell embeddings

## Inputs

- AnnData object (.h5ad) of interferon-beta stimulated PBMC scRNA-seq raw counts in .X
- GeneVector cell embedding (cell_embed) produced by prior GeneVector training on the same dataset
- Batch/condition labels in adata.obs (e.g. sample or stimulation status column)
- Cell type annotations or marker sets for per-cell-type silhouette stratification

## Outputs

- Three-panel UMAP comparison figure (batch_correction_umap_comparison.png) covering uncorrected, GeneVector-corrected, and Harmony-corrected embeddings
- Per-cell-type batch silhouette coefficient summary table for all three representations
- Per-cell-type stimulation-program (ISG) scores across the three UMAPs demonstrating preservation vs removal of treatment variation

## How to apply

Load the interferon-beta stimulated PBMC AnnData and the GeneVector cell embedding (cell_embed). Compute the correction vector V_correction = V_reference − V_batch by subtracting the average gene embedding of one batch from the reference batch, then add this vector to each individual cell embedding (per-cell vector addition, or use cell_embed.batch_correct with a reference column value). Recompute the UMAP with Scanpy on three representations: GeneVector-corrected, uncorrected, and Harmony-corrected embeddings for benchmarking. Following the paper's benchmarking scheme (ARI, kBET, cLISI, and silhouette score), compute per-cell-type batch silhouette coefficients (silhouette over batch labels within each cell type) on all three representations and assemble a three-panel UMAP comparison figure with a silhouette summary table. As a biological-preservation check, score stimulation programs (ISG metagenes) per cell type across the UMAPs: GeneVector correction should preserve interferon-stimulated myeloid variation while Harmony removes it, and GeneVector does not align myeloid cell types consistent with their stronger interferon response.

## Related tools

- **GeneVector** (Provides the cell embedding and batch_correct operation (V_reference − V_batch vector arithmetic added per cell) being benchmarked) — https://github.com/nceglia/genevector
- **Scanpy** (Recomputes UMAP embeddings on corrected/uncorrected representations and plots the comparison panels)
- **AnnData** (Container for raw counts in .X and batch/cell-type annotations in .obs used for silhouette scoring)
- **PyTorch** (Backend for GeneVector's neural embedding training underlying the corrected cell representations)
- **NumPy/SciPy** (Vector arithmetic on embeddings and silhouette coefficient computation over batch labels)

## Examples

```
cell_embed.batch_correct(column="sample", reference="control"); adata_corrected = cell_embed.get_adata()
```

## Evaluation signals

- The batch silhouette coefficient for CD14+ Monocytes is the largest among cell types in the GeneVector-corrected embedding
- Interferon-stimulated (ISG) transcriptional variation within myeloid cells remains detectable in the GeneVector-corrected UMAP but is absent under Harmony
- Cell types are aligned between control and interferon-stimulated batches after GeneVector correction (reduced within-cell-type silhouette over batch labels relative to uncorrected)
- GeneVector-corrected embeddings do not align myeloid cell types, consistent with their stronger interferon response
- Correction vectors should be manually verified against cell type signatures, as recommended when batches differ in cell type composition

## Limitations

- Batch correction is unreliable when batches differ in cell type composition; correction vectors must be manually checked against cell type signatures
- Benchmarking metrics (ARI, kBET, cLISI, silhouette score) are shown only in Supplementary Fig. 6A–L, not in the main text, creating a reproducibility gap
- GeneVector's derived correction vectors are agnostic to direction of regulation — MI cannot distinguish activation from repression, limiting interpretation of decomposed correction signatures

## Evidence

- [discussion] Benchmarking metrics referenced only as Supplementary Fig. 6A–L: "using a series of benchmarking metrics (ARI, kBET, cLISI, and silhouette score)49,50 (Supplementary Fig. 6A–I)"
- [discussion] Interpretable batch correction: "GeneVector can produce an interpretable batch correction by decomposing the derived correction vectors into transcriptional signatures"
- [discussion] Batch composition caveat: "Batch correction is challenging in the presence of batch-specific differences in cell type composition"
- [readme] README batch correction API: "# Optional: Batch correction
cell_embed.batch_correct(column="sample", reference="control")
adata_corrected = cell_embed.get_adata()"
