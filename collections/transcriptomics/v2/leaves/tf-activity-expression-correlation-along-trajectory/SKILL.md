---
name: tf-activity-expression-correlation-along-trajectory
description: Use when working with paired scRNA-seq/scATAC-seq (e.g., Seurat coembed.trajectory.rds)
  along a pseudotime trajectory and you must select candidate TFs for GRN inference,
  since chromVAR activity alone cannot separate TFs of the same family with similar
  motifs.
when_to_use_negative:
- When only scRNA-seq or only scATAC-seq data are available, since the correlation
  requires paired per-cell TF activity and TF expression measurements
- When the goal is motif enrichment or chromVAR deviation-score estimation itself
  — this skill is the downstream candidate-TF filter, not the activity-computation
  step
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scMEGA
  role: Provides AddMotifs, RunChromVAR, and SelectTFs used to match motifs, compute
    per-cell TF activity, and correlate activity with expression
  repo: github.com/CostaLab/scMEGA
- name: chromVAR
  role: Computes per-cell TF binding activity (deviation scores) for all motifs on
    the ATAC assay
- name: JASPAR2020
  role: Source of position frequency matrices for motif matching (CORE collection,
    vertebrates)
- name: TFBSTools
  role: getMatrixSet is used to retrieve the JASPAR2020 PFMs
- name: Seurat
  role: Holds the paired multiomic object and supplies functions wrapped by scMEGA
  repo: github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: ATAC-seq assay infrastructure for motif annotation in the Seurat object
  repo: github.com/stuart-lab/signac (branch seurat5)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:8720aabd53f14fa70277f9974a01b22a7edded0c240a7b2d7c039b997f819e8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Hsapiens.UCSC.hg38
  - JASPAR2020
  - R
  - Seurat
  - Signac
  - TFBSTools
  - chromVAR
  - scMEGA
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# tf-activity-expression-correlation-along-trajectory

## Summary

Identifies active transcription factors in single-cell multiomic data by correlating per-cell chromVAR TF binding (deviation) activity with TF gene expression along an inferred pseudotime trajectory, rather than relying on motif activity alone.

## When to use

Use when working with paired scRNA-seq/scATAC-seq (e.g., Seurat coembed.trajectory.rds) along a pseudotime trajectory and you must select candidate TFs for GRN inference, since chromVAR activity alone cannot separate TFs of the same family with similar motifs.

## When NOT to use

- When only scRNA-seq or only scATAC-seq data are available, since the correlation requires paired per-cell TF activity and TF expression measurements
- When the goal is motif enrichment or chromVAR deviation-score estimation itself — this skill is the downstream candidate-TF filter, not the activity-computation step

## Inputs

- Paired multiomic Seurat object with scRNA-seq and scATAC-seq assays for cells in the trajectory (e.g., coembed.trajectory.rds)
- JASPAR2020 position frequency matrices retrieved via TFBSTools getMatrixSet (CORE collection, vertebrates, all_versions = FALSE)
- BSgenome.Hsapiens.UCSC.hg38 genome for motif matching

## Outputs

- df.cor TF table with tf and time_point columns (saved as selected_tfs.csv)
- TF activity dynamics heatmap (ht object returned by SelectTFs with return.heatmap = TRUE)

## How to apply

Load the paired multiomic Seurat object for cells in the inferred pseudotime trajectory, retrieve JASPAR2020 position frequency matrices via TFBSTools getMatrixSet (collection = 'CORE', tax_group = 'vertebrates', all_versions = FALSE), then add motif matching to the ATAC assay with scMEGA's AddMotifs against BSgenome.Hsapiens.UCSC.hg38. Compute per-cell TF binding activity as chromVAR deviation scores with RunChromVAR on the ATAC assay. Run scMEGA's SelectTFs to correlate chromVAR TF activity scores with TF gene expression along the trajectory, retaining TFs passing the correlation criterion (cor.cutoff = 0.1 in the cardiomyocyte tutorial); the function returns the TF table df.cor and, with return.heatmap = TRUE, a TF activity dynamics heatmap (ht). The correlation step is crucial because TF-binding activity alone cannot differentiate between TFs of the same family sharing similar motifs.

## Related tools

- **scMEGA** (Provides AddMotifs, RunChromVAR, and SelectTFs used to match motifs, compute per-cell TF activity, and correlate activity with expression) — github.com/CostaLab/scMEGA
- **chromVAR** (Computes per-cell TF binding activity (deviation scores) for all motifs on the ATAC assay)
- **JASPAR2020** (Source of position frequency matrices for motif matching (CORE collection, vertebrates))
- **TFBSTools** (getMatrixSet is used to retrieve the JASPAR2020 PFMs)
- **Seurat** (Holds the paired multiomic object and supplies functions wrapped by scMEGA) — github.com/satijalab/seurat (branch seurat5)
- **Signac** (ATAC-seq assay infrastructure for motif annotation in the Seurat object) — github.com/stuart-lab/signac (branch seurat5)

## Examples

```
pfm <- getMatrixSet(JASPAR2020, opts = list(collection='CORE', tax_group='vertebrates', all_versions=FALSE)); obj <- AddMotifs(obj, genome=BSgenome.Hsapiens.UCSC.hg38); obj <- RunChromVAR(obj, genome=BSgenome.Hsapiens.UCSC.hg38); res <- SelectTFs(obj, cor.cutoff=0.1, return.heatmap=TRUE)
```

## Evaluation signals

- Every retained TF in df.cor has an activity-expression correlation exceeding the specified cor.cutoff (0.1 in the cardiomyocyte tutorial)
- The output selected_tfs.csv contains tf and time_point columns as produced by SelectTFs
- The returned ht heatmap shows TF activity dynamics along the trajectory for the retained TFs
- chromVAR deviation scores are computed for all JASPAR2020 CORE vertebrate motifs added to the ATAC assay, confirming AddMotifs/RunChromVAR ran before SelectTFs

## Limitations

- TF-binding activity alone cannot differentiate between TFs of the same family sharing similar motifs — hence the correlation filter is required
- Software/package versions are not specified in the tutorials (sessionInfo() is the only reference)
- The correlation cutoff (cor.cutoff = 0.1) is tutorial-specific and may need tuning for other datasets

## Evidence

- [other] The correlation-based identification of active TFs is a crucial step in scMEGA's candidate-TF filter because TF-binding activity alone cannot differentiate between TFs of the same family sharing similar motifs.: [evidence span withheld — anchor status: not_found]
- [methods] TF activity estimation via motif matching and chromVAR deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [methods] AddMotifs workflow: "obj <- AddMotifs(
  object = obj,
  genome = BSgenome.Hsapiens.UCSC.hg38,"
- [methods] JASPAR2020 PFM retrieval: "pfm <- getMatrixSet(
  x = JASPAR2020,"
