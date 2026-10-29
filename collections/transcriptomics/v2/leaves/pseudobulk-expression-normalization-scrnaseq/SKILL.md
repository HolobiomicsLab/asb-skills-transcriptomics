---
name: pseudobulk-expression-normalization-scrnaseq
description: Use when integrating multiple public time-course mouse scRNA-seq datasets
  (e.g., pre-implantation stages from GEO) into one expression matrix that must be
  batch-harmonized before trajectory, TRN, or ligand-receptor inference.
when_to_use_negative:
- Input is already quantile-normalized or on a comparable scale — re-normalization
  would distort the per-stage distributions.
- The analysis targets single-cell-level cell-cell communication on the Seurat object
  rather than stage-level pseudo-bulk averages; this skill operates on merged/stage-aggregated
  matrices.
- The data are bulk RNA-seq or single-stage data without a time-course/stage structure,
  since the pseudo-bulk step requires per-stage averaging.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: preprocessCore
  role: Provides normalize.quantiles (version 1.48.0) used to quantile-normalize the
    merged expression matrix
- name: Seurat
  role: CreateSeuratObject with min.features = 2000, min.cells = 0 filters cells before
    pseudo-bulk aggregation and normalization
- name: TimeTalk
  role: R package whose workflow consumes the normalized (log10(RPKM + 1)) matrix
    and pseudo-bulk profiles to build cell-type-specific TRNs and infer eLRs
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:211faf1efdf8d3d50595998626a61cabd637649eae0bf4a2f24cd4fd16b666c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - RTN
  - Seurat
  - TimeTalk
  - preprocessCore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Pseudobulk expression normalization for scRNA-seq

## Summary

Normalizes merged time-course scRNA-seq expression matrices from early embryo development by (a) computing pseudo-bulk expression as the average single cell per embryo stage and (b) quantile normalization of the merged matrix via preprocessCore, with an alternative log10(RPKM + 1) scheme used when building RTN transcriptional network objects.

## When to use

Use when integrating multiple public time-course mouse scRNA-seq datasets (e.g., pre-implantation stages from GEO) into one expression matrix that must be batch-harmonized before trajectory, TRN, or ligand-receptor inference.

## When NOT to use

- Input is already quantile-normalized or on a comparable scale — re-normalization would distort the per-stage distributions.
- The analysis targets single-cell-level cell-cell communication on the Seurat object rather than stage-level pseudo-bulk averages; this skill operates on merged/stage-aggregated matrices.
- The data are bulk RNA-seq or single-stage data without a time-course/stage structure, since the pseudo-bulk step requires per-stage averaging.

## Inputs

- Merged pre-implantation mouse scRNA-seq expression matrix (from public GEO datasets, refs 12, 13)
- Per-cell stage/cell-type annotation (Seurat object with orig.ident and ident)
- RPKM expression values for the log10(RPKM + 1) normalization path

## Outputs

- Quantile-normalized merged expression matrix (preprocessCore normalize.quantiles output)
- Pseudo-bulk per-stage average single-cell expression table
- log10(RPKM + 1) normalized matrix feeding the RTN 'tni' object (res/R/*RTN_rtna_object.rds pipeline)

## How to apply

First aggregate per-cell expression with custom R scripts into pseudo-bulk values representing the average single cell in each stage of the pre-implantation scRNA-seq data. Then apply quantile normalization to the merged expression matrix using normalize.quantiles from the R package preprocessCore (version 1.48.0), so that per-stage distributions become comparable across datasets. In the TimeTalk tTF branch, normalize instead (or additionally) to log10(RPKM + 1) before combining with the AnimalTFDB 3.0 TF list to build the RTN 'tni' object. Upstream, construct the Seurat object with CreateSeuratObject parameters min.features = 2000, min.cells = 0 so that poorly measured cells are excluded before aggregation. Rationale: quantile normalization removes distributional shifts between merged public datasets, while the log10(RPKM + 1) scale stabilizes variance for transcriptional network inference (RTN) and downstream master regulator analysis (tna.mra).

## Related tools

- **preprocessCore** (Provides normalize.quantiles (version 1.48.0) used to quantile-normalize the merged expression matrix)
- **Seurat** (CreateSeuratObject with min.features = 2000, min.cells = 0 filters cells before pseudo-bulk aggregation and normalization)
- **TimeTalk** (R package whose workflow consumes the normalized (log10(RPKM + 1)) matrix and pseudo-bulk profiles to build cell-type-specific TRNs and infer eLRs) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
library(preprocessCore); norm.mat <- normalize.quantiles(expr.mat)
```

## Evaluation signals

- The normalized matrix is column-quantile-normalized: the per-column (per-dataset) empirical distributions are identical after normalize.quantiles.
- All log10(RPKM + 1) values are non-negative and the pseudo-bulk table contains one average single-cell value per embryo stage.
- The RTN 'tni' object builds without error from the normalized matrix plus the AnimalTFDB 3.0 TF list.
- Upstream cell filtering removed cells with fewer than 2000 detected features (min.features = 2000, min.cells = 0 in CreateSeuratObject).
- kBET analysis in PCA space confirms data points from different batches are well-mixed within fixed-size random neighborhoods after normalization.

## Limitations

- Results depend on downstream parameter choices; the authors advise testing multiple parameter combinations for dependable TimeTalk outputs.
- Quantile normalization of merged public datasets does not remove all technical confounders; kBET was used as an explicit batch-effect check.
- The pseudo-bulk 'average single cell' representation discards within-stage cell-to-cell variability.

## Evidence

- [discussion] Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0).: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [discussion] Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage.: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0)71 to build the “tni” object: "the log10(RPKM + 1) normalized scRNA-seq data and TF list from AnimalTFDB (version 3.0)71 to build the “tni” object"
- [discussion] The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size.: "The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size."
- [discussion] “CreateSeuratObject” in Seurat (parameters: min.features = 2000, min.cells = 0).: "“CreateSeuratObject” in Seurat (parameters: min.features = 2000, min.cells = 0)."
