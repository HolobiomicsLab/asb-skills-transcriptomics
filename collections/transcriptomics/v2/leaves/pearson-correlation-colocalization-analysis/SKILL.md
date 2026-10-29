---
name: pearson-correlation-colocalization-analysis
description: Use when you have Space Ranger-processed Visium spot-level expression
  for transcriptionally active regions (TARs/uTARs) plus known cell-type marker genes
  and need to test whether uTARs spatially colocalize with specific cell types across
  tissue sections.
when_to_use_negative:
- Input is single-cell (per-cell) scRNA-seq expression rather than spatial spot-level
  Visium data — the colocalization test requires spatial spot coordinates
- TAR expression has not been normalized (raw counts) — Pearson correlations must
  be computed on log-e-normalized matrices per the workflow
- No canonical marker genes are known for the candidate cell types, so correlation
  entries cannot be biologically interpreted
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: Seurat
  role: Loads Visium count matrices and log-e-normalizes the spatial feature expression
    matrices prior to correlation
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model segmentation of Space-Ranger-derived alignments to call
    spatial transcriptionally active regions (TARs)
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: Space Ranger
  role: Produces the Visium alignment files and count matrices used as correlation
    inputs
- name: rtracklayer
  role: Loads groHMM-generated BED track files of spatial TAR annotations
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Orchestrates the TAR-scRNA-seq workflow that generates uTARs and expression
    matrices feeding the correlation analysis
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:dcfefdf8c71e87a0cfdd641b32cb71f87f3dad92615e0dea2e905e7fa2f09562
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - Seurat
  - Snakemake
  - Space Ranger
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pearson-correlation-colocalization-analysis

## Summary

Quantifies spatial colocalization between uTAR and canonical cell-type marker expression in 10X Visium spot-level data by computing Pearson correlations on log-e-normalized (Seurat) expression matrices, then hierarchically clustering the uTAR-by-marker correlation matrix to reproduce dendrogram-style colocalization figures.

## When to use

Use when you have Space Ranger-processed Visium spot-level expression for transcriptionally active regions (TARs/uTARs) plus known cell-type marker genes and need to test whether uTARs spatially colocalize with specific cell types across tissue sections.

## When NOT to use

- Input is single-cell (per-cell) scRNA-seq expression rather than spatial spot-level Visium data — the colocalization test requires spatial spot coordinates
- TAR expression has not been normalized (raw counts) — Pearson correlations must be computed on log-e-normalized matrices per the workflow
- No canonical marker genes are known for the candidate cell types, so correlation entries cannot be biologically interpreted

## Inputs

- Space Ranger-processed 10X Visium count matrices (spot x feature), e.g., 747 day-4 spots from 5 hearts and 1995 day-14 spots from 1 heart
- Combined TAR + GRCg6a Ensembl v96 genome annotation (GTF/BED)
- Space-Ranger-derived alignment (BAM) files per time point for groHMM spatial TAR calling
- List of differentially expressed scRNA-seq uTARs for spatial overlap filtering
- Canonical cell-type marker genes (e.g., HBZ, TNNT2, COL1A1)

## Outputs

- Full uTAR-by-marker Pearson correlation matrix (e.g., spatial_utar_marker_correlations.csv)
- Hierarchically clustered dendrogram of uTAR-marker correlations (analogous to Fig. 4b)
- Spatial uTAR annotations with matched chromosome position and strandedness to scRNA-seq uTARs

## How to apply

Load the Space Ranger count matrices (e.g., 747 day-4 spots from 5 chicken hearts; 1995 day-14 spots from 1 heart) together with the combined TAR + GRCg6a Ensembl v96 annotation into Seurat and log-e-normalize the feature expression matrices. Run groHMM on the Space-Ranger-derived alignment files per time point to derive spatial TAR annotations, retaining spatial uTARs that overlap differentially expressed scRNA-seq uTARs in chromosome position and strandedness, and keeping only the highest-total-expression spatial uTAR when several overlap one scRNA-seq uTAR. Compute Pearson correlations of log-e-normalized spot expression between each spatial uTAR (e.g., the 11 day-4 spatial uTARs) and canonical markers such as HBZ (red blood cells), TNNT2 (myocytes), and COL1A1 (epicardial progenitors) for each section. Perform hierarchical clustering on the correlation values to build a dendrogram, and export the full uTAR-by-marker correlation matrix (e.g., spatial_utar_marker_correlations.csv) for validation against reported colocalization patterns.

## Related tools

- **Seurat** (Loads Visium count matrices and log-e-normalizes the spatial feature expression matrices prior to correlation) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model segmentation of Space-Ranger-derived alignments to call spatial transcriptionally active regions (TARs)) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **Space Ranger** (Produces the Visium alignment files and count matrices used as correlation inputs)
- **rtracklayer** (Loads groHMM-generated BED track files of spatial TAR annotations) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Orchestrates the TAR-scRNA-seq workflow that generates uTARs and expression matrices feeding the correlation analysis) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
seu <- Load10X_Spatial('day4_spaceranger/'); seu <- SCTransform(seu, assay='Spatial'); cor(t(GetAssayData(seu)['SH3BGR_uTAR',]), t(GetAssayData(seu)['TNNT2',]), method='pearson')
```

## Evaluation signals

- SH3BGR-related uTAR shows positive Pearson correlation with the cardiomyocyte marker TNNT2 at both day 4 and day 14 sections
- RUNX1T1 uTAR correlates with COL1A1+ epicardial spots at day 4 and shows no/absent association at day 14 (verified via sign/magnitude of the correlation entries)
- Hierarchical clustering of the correlation matrix yields a dendrogram pattern consistent with the published figure (analogous to Fig. 4b)
- Correlations are computed on log-e-normalized spot expression (not raw counts) from Seurat-normalized matrices
- Every retained spatial uTAR overlaps a differentially expressed scRNA-seq uTAR in chromosome position and strandedness; ties resolved by highest total expression

## Limitations

- groHMM R scripts may fail silently and leave missing chr*_HMM.bed files, producing 'no lines available in input' errors in rule calcHMMrefFlat — check SingleCellHMM_Run_combined_bam_HMM_features.log and manually install groHMM and rtracklayer
- Colocalization is qualitative/correlational: validation relies on sign and magnitude of Pearson entries rather than formal significance testing
- uTAR relevance is strongly dependent on organism, tissue, cell type, and developmental stage, so colocalization patterns may not transfer between sections or time points

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [intro] uTARs differentially expressed across clusters retrieved and annotated: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [readme] README FAQ on groHMM/rtracklayer install failure: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
- [readme] Workflow from_fastqs vs from_cellranger selection: "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)."
- [intro] Degree of relevant information in uTARs context-dependent: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
