---
name: myeloid-isg-variation-preservation-assessment
description: Use when a batch-corrected scRNA-seq embedding (e.g., interferon-beta
  stimulated PBMCs) must be checked for whether treatment-related variation in myeloid
  cells survives correction, or when comparing GeneVector batch correction against
  Harmony using silhouette scores and ISG program scoring.
when_to_use_negative:
- Batches differ substantially in cell type composition — the article warns batch
  correction is challenging in the presence of batch-specific differences in cell
  type composition, so correction vectors can conflate batch and cell-type signals
  and must be manually verified against cell type signatures first
- The goal is to infer directionality of regulation (activation vs. repression) —
  GeneVector's mutual information is agnostic to the direction of the statistical
  dependency
- No cell-type annotations or stimulation labels are available, since per-cell-type
  silhouette coefficients and ISG program scoring cannot be computed without them
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: GeneVector
  role: Provides the cell embedding, vector-arithmetic batch correction (batch_correct
    with reference batch), and interpretable decomposition of correction vectors into
    transcriptional signatures
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Recomputes UMAP embeddings on uncorrected, GeneVector-corrected, and Harmony-corrected
    representations and visualizes stimulation program scores
- name: AnnData
  role: Container format (raw counts in .X) carrying the PBMC interferon-beta stimulated
    dataset, batch labels, and corrected embeddings
- name: Harmony
  role: Alternative batch-correction benchmark whose correction removes interferon-stimulated
    myeloid variation, used as the contrast condition
- name: SciPy
  role: Supports silhouette coefficient computation over batch labels within each
    cell type
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:7bcde55da4351d634ea3ecad13390ee439db6e74963e8376b2669c167199c084
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - Harmony
  - NumPy
  - PyTorch
  - Scanpy
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# myeloid-isg-variation-preservation-assessment

## Summary

Benchmarks whether a batch-corrected single-cell embedding preserves interferon-stimulated (ISG) transcriptional variation within myeloid cells, using per-cell-type batch silhouette coefficients (CD14+ Monocytes expected largest after GeneVector correction) and stimulation-program scoring to contrast GeneVector vector-arithmetic correction against Harmony, which removes treatment-related variation.

## When to use

Use when a batch-corrected scRNA-seq embedding (e.g., interferon-beta stimulated PBMCs) must be checked for whether treatment-related variation in myeloid cells survives correction, or when comparing GeneVector batch correction against Harmony using silhouette scores and ISG program scoring.

## When NOT to use

- Batches differ substantially in cell type composition — the article warns batch correction is challenging in the presence of batch-specific differences in cell type composition, so correction vectors can conflate batch and cell-type signals and must be manually verified against cell type signatures first
- The goal is to infer directionality of regulation (activation vs. repression) — GeneVector's mutual information is agnostic to the direction of the statistical dependency
- No cell-type annotations or stimulation labels are available, since per-cell-type silhouette coefficients and ISG program scoring cannot be computed without them

## Inputs

- AnnData object (.h5ad) of interferon-beta stimulated PBMC scRNA-seq with raw counts in .X and batch/stimulus labels in obs (e.g., 'sample' column)
- GeneVector cell embedding (cell_embed) produced from the trained gene vectors (genes.vec, emb_dimension=100)
- Cell type annotations (e.g., CD14+ Monocytes, myeloid cells, T cells)
- Marker gene sets for stimulation programs / ISG metagene scoring
- Optional: Harmony-corrected embedding for benchmarking contrast

## Outputs

- Batch-corrected cell embedding (AnnData) via per-cell vector addition of V_correction
- Three recomputed UMAPs: uncorrected, GeneVector-corrected, Harmony-corrected
- Per-cell-type batch silhouette coefficients (silhouette over batch labels within each cell type)
- Stimulation/ISG program scores per cell type across the three UMAPs
- Three-panel comparison figure saved as batch_correction_umap_comparison.png
- Silhouette-coefficient summary table

## How to apply

On the interferon-beta stimulated PBMC AnnData and its GeneVector cell embedding (cell_embed), derive the batch correction vector V_correction = V_reference − V_batch from average gene embeddings of the two batches and add it to every cell embedding, or call GeneVector's built-in correction (e.g., batch_correct with a sample column and a control reference). Recompute UMAPs with Scanpy on three representations: uncorrected, GeneVector-corrected, and Harmony-corrected embeddings. For each cell type, compute the silhouette coefficient over batch labels (the paper's benchmarking scheme uses ARI, kBET, cLISI, and silhouette score); verify CD14+ Monocytes show the largest batch silhouette coefficient in the GeneVector-corrected embedding. Then score stimulation programs (ISG metagenes) per cell type across all three UMAPs: GeneVector-corrected data should keep stimulated myeloid variation detectable and should NOT align myeloid cell types between control and stimulated batches (consistent with a stronger interferon response), whereas Harmony removes it. Assemble a three-panel UMAP comparison figure and a silhouette-coefficient summary table (batch_correction_umap_comparison.png) and only accept the result if both validation conditions hold.

## Related tools

- **GeneVector** (Provides the cell embedding, vector-arithmetic batch correction (batch_correct with reference batch), and interpretable decomposition of correction vectors into transcriptional signatures) — https://github.com/nceglia/genevector
- **Scanpy** (Recomputes UMAP embeddings on uncorrected, GeneVector-corrected, and Harmony-corrected representations and visualizes stimulation program scores)
- **AnnData** (Container format (raw counts in .X) carrying the PBMC interferon-beta stimulated dataset, batch labels, and corrected embeddings)
- **Harmony** (Alternative batch-correction benchmark whose correction removes interferon-stimulated myeloid variation, used as the contrast condition)
- **SciPy** (Supports silhouette coefficient computation over batch labels within each cell type)

## Examples

```
cell_embed.batch_correct(column="sample", reference="control"); adata_corrected = cell_embed.get_adata()
```

## Evaluation signals

- CD14+ Monocytes exhibit the largest batch silhouette coefficient among all cell types in the GeneVector-corrected embedding
- Interferon-stimulated (ISG) program scores remain detectable in myeloid cells on the GeneVector-corrected UMAP
- Stimulated myeloid variation is absent in the Harmony-corrected UMAP (expected contrast condition)
- GeneVector does not align myeloid cell types between control and stimulated batches, consistent with a stronger interferon response in these cells
- Output accepted only if both the largest-silhouette condition and myeloid-ISG-preservation condition hold; the paper's benchmarking scheme is ARI, kBET, cLISI, and silhouette score

## Limitations

- Batch correction is unreliable when batches differ in cell type composition; users must manually verify correction vectors against cell type signatures
- GeneVector cannot distinguish activation from repression, as it builds mutual information that is agnostic to the direction of the statistical dependency
- The paper's benchmarking metrics (ARI, kBET, cLISI, silhouette) are only referenced as Supplementary Fig. 6A–L and not shown in the main text, a reproducibility gap for this comparison

## Evidence

- [discussion] interpretable batch correction: "GeneVector can produce an interpretable batch correction by decomposing the derived correction vectors into transcriptional signatures"
- [discussion] batch composition caveat: "Batch correction is challenging in the presence of batch-specific differences in cell type composition"
- [discussion] benchmarking metrics: "using a series of benchmarking metrics (ARI, kBET, cLISI, and silhouette score)49,50 (Supplementary Fig. 6A–I)"
- [discussion] ISG metagene in PBMCs: "In interferon-stimulated PBMCs, we identify a cell type independent ISG metagene"
- [readme] built-in batch correction API: "cell_embed.batch_correct(column="sample", reference="control")
adata_corrected = cell_embed.get_adata()"
