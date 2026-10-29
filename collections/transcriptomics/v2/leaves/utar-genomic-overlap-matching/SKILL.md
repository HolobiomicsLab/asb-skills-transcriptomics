---
name: utar-genomic-overlap-matching
description: Use when spatial Visium count matrices and a TAR+reference genome annotation
  are available and you must confirm that scRNA-seq-discovered uTARs are spatially
  detectable and colocalize with known marker genes.
when_to_use_negative:
- Input is a single-cell (non-spatial) expression matrix — this skill requires per-spot
  Visium data with spatial coordinates
- Only raw FASTQs without Space Ranger count matrices or alignments are available
  and no alignment step is planned
- The organism lacks any genome annotation or TAR track, so uTAR coordinates/strandedness
  cannot be compared across modalities
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: groHMM
  role: Runs a hidden Markov model over Space-Ranger alignments to call spatially
    transcribed active regions (TARs) used for uTAR overlap matching
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: Loads and manipulates groHMM-generated TAR BED/GTF annotation tracks in R
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Seurat
  role: Loads Visium spatial matrices, log-e-normalizes spot expression, and provides
    the correlation computation between uTARs and marker genes
- name: Space Ranger
  role: Produces the Visium count matrices and alignment files that groHMM is run
    on
- name: Snakemake
  role: Orchestrates the from_fastqs/from_cellranger TAR-scRNA-seq workflow rules,
    including the groHMM HMM step
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:3482bb89386b93a4e4345534444595bb5e3faa7d3dd2092ee8f60c547d8e13db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - Snakemake
  - Space Ranger
  - groHMM
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# utar-genomic-overlap-matching

## Summary

Matches spatial-transcriptomics uTARs to scRNA-seq uTARs by chromosome-position and strandedness overlap, then tests whether these uTARs colocalize with canonical cell-type markers by Pearson correlation of log-e-normalized spot expression. Extends annotation-independent TAR analysis from single-cell to 10X Visium spatial data.

## When to use

Use when spatial Visium count matrices and a TAR+reference genome annotation are available and you must confirm that scRNA-seq-discovered uTARs are spatially detectable and colocalize with known marker genes.

## When NOT to use

- Input is a single-cell (non-spatial) expression matrix — this skill requires per-spot Visium data with spatial coordinates
- Only raw FASTQs without Space Ranger count matrices or alignments are available and no alignment step is planned
- The organism lacks any genome annotation or TAR track, so uTAR coordinates/strandedness cannot be compared across modalities

## Inputs

- Space Ranger-processed 10X Visium count matrices (per-spot spot-by-feature matrices for each time point/section)
- Combined TAR + GRCg6a Ensembl v96 genome annotation (GTF/BED)
- Space-Ranger-derived BAM/alignment files for groHMM TAR discovery
- List of differentially expressed scRNA-seq uTARs with chromosome coordinates and strand
- Canonical cell-type marker gene list (e.g. HBZ, TNNT2, COL1A1)

## Outputs

- Spatial uTAR annotation tracks (groHMM BED/GTF from Visium alignments)
- Matched set of spatial uTARs with chromosome-position and strandedness assignments to scRNA-seq uTARs
- uTAR-by-marker Pearson correlation matrix (e.g. spatial_utar_marker_correlations.csv)
- Hierarchical-clustering dendrogram of uTAR-marker correlations (Fig. 4b analogue)

## How to apply

Load Space Ranger-processed 10X Visium count matrices (e.g. 747 day-4 spots from 5 chicken hearts; 1995 day-14 spots from 1 heart) together with a combined TAR + GRCg6a Ensembl v96 annotation using Seurat's spatial transcriptome tools, then log-e-normalize the feature matrices with Seurat. Run groHMM on the combined Space-Ranger alignment files across all time points to derive spatial TAR annotations; identify spatial uTARs that overlap differentially expressed scRNA-seq uTARs in both chromosome position and strandedness, and when several spatial uTARs map to one scRNA-seq uTAR keep the one with the highest total expression. Compute Pearson correlations of log-e-normalized spot expression between the day-4 spatial uTARs (e.g. 11 uTARs) and canonical markers such as HBZ (red blood cells), TNNT2 (myocytes), and COL1A1 (epicardial progenitors) for both day-4 and day-14 sections, then hierarchically cluster the correlation matrix into a dendrogram analogous to Fig. 4b.

## Related tools

- **groHMM** (Runs a hidden Markov model over Space-Ranger alignments to call spatially transcribed active regions (TARs) used for uTAR overlap matching) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (Loads and manipulates groHMM-generated TAR BED/GTF annotation tracks in R) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Seurat** (Loads Visium spatial matrices, log-e-normalizes spot expression, and provides the correlation computation between uTARs and marker genes)
- **Space Ranger** (Produces the Visium count matrices and alignment files that groHMM is run on)
- **Snakemake** (Orchestrates the from_fastqs/from_cellranger TAR-scRNA-seq workflow rules, including the groHMM HMM step) — github.com/fw262/TAR-scRNA-seq

## Examples

```
seu <- NormalizeData(seu, normalization.method = "LogNormalize", scale.factor = exp(1)); cor_mat <- cor(t(GetAssayData(seu)[utars, spots]), t(GetAssayData(seu)[markers, spots]), method = "pearson")
```

## Evaluation signals

- Every spatial uTAR accepted into the match set overlaps its paired scRNA-seq uTAR in both chromosome position and strandedness; mismatches are excluded
- When multiple spatial uTARs map to one scRNA-seq uTAR, exactly one (the highest-total-expression one) is retained
- The correlation CSV contains entries for all 11 day-4 spatial uTARs against HBZ, TNNT2, and COL1A1 for both day-4 and day-14 sections
- SH3BGR uTAR shows positive correlation/colocalization with TNNT2 at BOTH day 4 and day 14, and RUNX1T1 uTAR correlates with COL1A1+ spots at day 4 but is absent at day 14 (sign/magnitude check on Pearson entries)
- Hierarchical clustering of the correlation matrix yields a dendrogram structure analogous to Fig. 4b of the paper

## Limitations

- The README FAQ documents a known failure mode: rule calcHMMrefFlat errors with 'no lines available in input' / 'chr*_HMM.bed: No such file or directory' when groHMM is not correctly installed and its R script fails to emit groHMM bed files
- Day-14 spatial data covered only 1 heart (1995 spots) versus 5 hearts (747 spots) at day 4, so day-14 inferences rest on a single section
- uTAR overlap matching relies on chromosome position and strandedness, so spatial uTARs with shifted boundaries but the same origin may be missed or ambiguously assigned

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] groHMM and rtracklayer must be installed and loadable in R for the HMM rule to generate groHMM bed files: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
- [abstract] The tool generates a single-cell expression matrix for all transcriptionally active regions (TARs): "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs"
- [readme] Workflows support starting from raw fastqs or from cellranger count output: "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)."
