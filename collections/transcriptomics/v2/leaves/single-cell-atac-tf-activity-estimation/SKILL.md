---
name: single-cell-atac-tf-activity-estimation
description: Use when a single-cell ATAC-seq peak matrix (e.g. from a paired multiome
  Seurat object such as coembed.trajectory.rds) must be converted into per-cell TF
  activity scores to prioritize candidate TFs for enhancer-based GRN inference.
when_to_use_negative:
- The input is a single-modality scRNA-seq dataset with no ATAC assay or peak matrix
  — motif-based deviation scores cannot be computed
- The genome build of the ATAC peaks has no matching BSgenome package (the tutorial
  uses BSgenome.Hsapiens.UCSC.hg38), or the TFs of interest are absent from the JASPAR2020
  CORE vertebrates motif set
- A curated activity matrix already exists and only the expression-correlation filter
  (SelectTFs) is required — re-running motif matching and chromVAR is redundant
edam_operation: http://edamontology.org/operation_0239
tools:
- name: scMEGA
  role: Provides AddMotifs and RunChromVAR wrappers that add motif annotations to
    the ATAC assay and compute chromVAR deviation scores within a Seurat/Signac workflow
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: Hosts the multiomic object structure (v5, seurat5 branch) on which motif enrichment
    and deviation scores are computed
  repo: https://github.com/satijalab/seurat
- name: Signac
  role: Single-cell chromatin analysis framework (seurat5 branch) underlying the ATAC
    assay handling
  repo: https://github.com/stuart-lab/signac
- name: chromVAR
  role: Computes per-cell TF activity as deviation scores from motif-peak accessibility
- name: JASPAR2020
  role: Source of position frequency matrices (CORE collection, vertebrates) used
    for motif matching
- name: TFBSTools
  role: getMatrixSet retrieves the JASPAR2020 PFMs passed to AddMotifs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:cb6a3ff05e39775c3695d715ad6924b5d161360db8ae739e8d8931ca18ce6205
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - BSgenome.Hsapiens.UCSC.hg38
  - JASPAR2020
  - R
  - Seurat
  - Signac
  - TFBSTools
  - chromVAR
  - scMEGA
  edam_operation: http://edamontology.org/operation_0239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-atac-tf-activity-estimation

## Summary

Estimate per-cell transcription factor (TF) activity from single-cell ATAC-seq data by matching JASPAR2020 motifs to peaks and computing chromVAR deviation scores, as implemented in scMEGA's AddMotifs and RunChromVAR wrappers for Seurat/Signac objects.

## When to use

Use when a single-cell ATAC-seq peak matrix (e.g. from a paired multiome Seurat object such as coembed.trajectory.rds) must be converted into per-cell TF activity scores to prioritize candidate TFs for enhancer-based GRN inference.

## When NOT to use

- The input is a single-modality scRNA-seq dataset with no ATAC assay or peak matrix — motif-based deviation scores cannot be computed
- The genome build of the ATAC peaks has no matching BSgenome package (the tutorial uses BSgenome.Hsapiens.UCSC.hg38), or the TFs of interest are absent from the JASPAR2020 CORE vertebrates motif set
- A curated activity matrix already exists and only the expression-correlation filter (SelectTFs) is required — re-running motif matching and chromVAR is redundant

## Inputs

- Seurat v5 / Signac object containing an ATAC assay with a peak-by-cell matrix (e.g. coembed.trajectory.rds from cells in an inferred pseudotime trajectory)
- JASPAR2020 motif database (CORE collection, vertebrates, latest versions only) retrieved via TFBSTools::getMatrixSet
- Genome assembly BSgenome.Hsapiens.UCSC.hg38

## Outputs

- ATAC assay enriched with motif annotations (motif-by-peak matching stored in the Seurat object's motifs slot)
- Per-cell chromVAR deviation score matrix giving TF activity for every motif
- Downstream TF table df.cor with tf and time_point columns (e.g. saved as selected_tfs.csv) when combined with the expression-correlation filter
- TF activity dynamics heatmap (ht object returned by SelectTFs with return.heatmap = TRUE)

## How to apply

Retrieve JASPAR2020 position frequency matrices with TFBSTools::getMatrixSet using opts collection='CORE', tax_group='vertebrates', all_versions=FALSE. Add motif annotations to the ATAC assay with scMEGA's AddMotifs against BSgenome.Hsapiens.UCSC.hg38, which scans peaks for motif occurrences. Then compute per-cell TF activity as chromVAR deviation scores for all motifs via scMEGA's RunChromVAR. The rationale, per the article, is that 'an acitivty score for each TF in each cell' is obtained by 'first performing motif matching and then computing deviation scores'; these scores can subsequently be correlated with TF gene expression (e.g. SelectTFs with cor.cutoff = 0.1 in the cardiomyocyte tutorial) because binding activity alone cannot distinguish TFs of the same family sharing similar motifs. Record sessionInfo() for reproducibility, since package versions are not otherwise pinned in the tutorials.

## Related tools

- **scMEGA** (Provides AddMotifs and RunChromVAR wrappers that add motif annotations to the ATAC assay and compute chromVAR deviation scores within a Seurat/Signac workflow) — https://github.com/CostaLab/scMEGA
- **Seurat** (Hosts the multiomic object structure (v5, seurat5 branch) on which motif enrichment and deviation scores are computed) — https://github.com/satijalab/seurat
- **Signac** (Single-cell chromatin analysis framework (seurat5 branch) underlying the ATAC assay handling) — https://github.com/stuart-lab/signac
- **chromVAR** (Computes per-cell TF activity as deviation scores from motif-peak accessibility)
- **JASPAR2020** (Source of position frequency matrices (CORE collection, vertebrates) used for motif matching)
- **TFBSTools** (getMatrixSet retrieves the JASPAR2020 PFMs passed to AddMotifs)

## Examples

```
obj <- AddMotifs(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38); obj <- RunChromVAR(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38)
```

## Evaluation signals

- The ATAC assay's motifs slot is populated with JASPAR2020 motif-peak matches after AddMotifs, and RunChromVAR returns a deviation score matrix covering all motifs in the motif set
- The TF activity heatmap (ht) shows TF activity dynamics along the inferred pseudotime trajectory, confirming scores are ordered per cell
- When combined with the correlation filter, every retained TF in df.cor has an activity-expression correlation exceeding cor.cutoff = 0.1 (cardiomyocyte tutorial)
- sessionInfo() output is recorded, since software/package versions are otherwise not specified in the tutorials
- Motif matching is restricted to the JASPAR2020 CORE vertebrates collection with all_versions = FALSE, so the motif count matches that specification

## Limitations

- TF-binding activity alone cannot differentiate between TFs of the same family sharing similar motifs, so deviation scores must be paired with TF expression correlation to identify active TFs
- Software/package versions are not specified in the tutorials (only sessionInfo() is provided), so exact reproducibility may require matching versions
- The workflow is demonstrated on human hg38 with BSgenome.Hsapiens.UCSC.hg38 and JASPAR2020; other genomes or motif versions require adapting AddMotifs/RunChromVAR inputs

## Evidence

- [methods] TF activity estimation via motif matching and chromVAR deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [methods] JASPAR2020 motif retrieval parameters: "pfm <- getMatrixSet(
  x = JASPAR2020,"
- [methods] AddMotifs against hg38 genome: "obj <- AddMotifs(
  object = obj,
  genome = BSgenome.Hsapiens.UCSC.hg38,"
- [methods] chromVAR deviation scores via RunChromVAR: "obj <- RunChromVAR(
  object = obj,
  genome = BSgenome.Hsapiens.UCSC.hg38,"
- [full_text] Rationale for correlating activity with expression: [evidence span withheld — anchor status: not_found]
- [readme] scMEGA installation and tutorial context: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
