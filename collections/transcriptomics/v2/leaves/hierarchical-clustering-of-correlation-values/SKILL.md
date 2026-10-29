---
name: hierarchical-clustering-of-correlation-values
description: Use when you have log-e-normalized expression of candidate uTARs plus
  canonical cell-type marker genes (e.g., HBZ, TNNT2, COL1A1) over matching spots/cells
  and need to test which uTARs are cell-type specific or spatially colocalized, and
  to visualize groupings as a dendrogram.
when_to_use_negative:
- The expression matrix is raw (not log-e-normalized) or derives from platforms/sections
  that were not matched to the scRNA-seq uTARs by position and strandedness — correlations
  would reflect normalization and annotation artifacts rather than colocalization.
- You need de novo discovery of cell-type markers rather than testing candidate uTARs
  against known markers; this skill assumes markers such as HBZ, TNNT2, and COL1A1
  are already established.
- Only one time point or section is available with no matched uTAR set, so no day-4
  vs day-14 colocalization validation (presence/absence of RUNX1T1 uTAR signal) is
  possible.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Spatial transcriptome tool used to load Space Ranger Visium count matrices
    and log-e-normalize the feature expression matrix prior to correlation computation
  repo: github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model run on Space Ranger-derived alignment files across the
    four time points to generate spatial TAR annotations from which uTARs are selected
  repo: github.com/fw262/TAR-scRNA-seq
- name: Space Ranger
  role: Produces the 10X Visium count matrices and alignment files that are the raw
    inputs to uTAR calling and normalization
- name: rtracklayer
  role: Used alongside groHMM to load/interconvert the groHMM BED track files; README
    warns this must be manually installed for the HMM rules to run
  repo: github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: Orchestrates the TAR-scRNA-seq workflow (from_fastqs or from_cellranger entry
    points) that generates the TAR expression matrices feeding this analysis
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:d85ba5bc5a82714e8ac1e55fb5050e2954a7773bd856cc74e0ca872106428a82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - Snakemake
  - Space Ranger
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# hierarchical-clustering-of-correlation-values

## Summary

Quantifies uTAR–marker colocalization by computing Pearson correlations of log-e-normalized expression (spot-level for Visium, cell-level for scRNA-seq) between transcriptionally active regions and canonical marker genes, then hierarchically clusters the correlation matrix into a dendrogram. This mirrors Fig. 4b of the TAR-scRNA-seq paper, where spatially co-expressed uTARs group with the cell-type markers they colocate with (e.g., SH3BGR uTAR with TNNT2+ myocytes, RUNX1T1 uTAR with COL1A1+ epicardial spots).

## When to use

Use when you have log-e-normalized expression of candidate uTARs plus canonical cell-type marker genes (e.g., HBZ, TNNT2, COL1A1) over matching spots/cells and need to test which uTARs are cell-type specific or spatially colocalized, and to visualize groupings as a dendrogram.

## When NOT to use

- The expression matrix is raw (not log-e-normalized) or derives from platforms/sections that were not matched to the scRNA-seq uTARs by position and strandedness — correlations would reflect normalization and annotation artifacts rather than colocalization.
- You need de novo discovery of cell-type markers rather than testing candidate uTARs against known markers; this skill assumes markers such as HBZ, TNNT2, and COL1A1 are already established.
- Only one time point or section is available with no matched uTAR set, so no day-4 vs day-14 colocalization validation (presence/absence of RUNX1T1 uTAR signal) is possible.

## Inputs

- Log-e-normalized (Seurat LogNormalize) spot-level expression matrix from Space Ranger-processed 10X Visium counts (e.g., 747 day-4 spots, 1995 day-14 spots)
- Set of spatial uTARs matched to differential scRNA-seq uTARs by chromosome position and strandedness (one-to-one by highest total expression)
- List of canonical cell-type marker genes (HBZ, TNNT2, COL1A1) with their associated cell types

## Outputs

- uTAR-by-marker Pearson correlation matrix (spatial_utar_marker_correlations.csv), computed per time point
- Hierarchically clustered dendrogram of the correlation values, analogous to Fig. 4b
- Qualitative colocalization calls (e.g., SH3BGR uTAR–TNNT2 at day 4 and day 14; RUNX1T1 uTAR–COL1A1 at day 4 only)

## How to apply

First log-e-normalize the feature expression matrix (Seurat's LogNormalize on the Space Ranger count matrices, e.g., 747 day-4 spots from 5 chicken hearts and 1995 day-14 spots). Restrict to the spatial uTARs that overlap differentially expressed scRNA-seq uTARs in chromosome position and strandedness (keep the highest-total-expression uTAR when several overlap one scRNA-seq uTAR — 11 such day-4 spatial uTARs in the chicken heart study) and to canonical markers: HBZ (red blood cells), TNNT2 (myocytes), COL1A1 (epicardial progenitors). Compute pairwise Pearson correlations between log-e-normalized uTAR and marker expression profiles per time point, write the full uTAR-by-marker matrix (spatial_utar_marker_correlations.csv), and run hierarchical clustering on the correlation values to build a dendrogram analogous to Fig. 4b. Validate by checking the sign/magnitude of the expected entries: SH3BGR uTAR should correlate with TNNT2 at both day 4 and day 14, while the day-4-only RUNX1T1 uTAR should correlate with COL1A1 at day 4 but be absent at day 14.

## Related tools

- **Seurat** (Spatial transcriptome tool used to load Space Ranger Visium count matrices and log-e-normalize the feature expression matrix prior to correlation computation) — github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model run on Space Ranger-derived alignment files across the four time points to generate spatial TAR annotations from which uTARs are selected) — github.com/fw262/TAR-scRNA-seq
- **Space Ranger** (Produces the 10X Visium count matrices and alignment files that are the raw inputs to uTAR calling and normalization)
- **rtracklayer** (Used alongside groHMM to load/interconvert the groHMM BED track files; README warns this must be manually installed for the HMM rules to run) — github.com/fw262/TAR-scRNA-seq
- **Snakemake** (Orchestrates the TAR-scRNA-seq workflow (from_fastqs or from_cellranger entry points) that generates the TAR expression matrices feeding this analysis) — github.com/fw262/TAR-scRNA-seq

## Examples

```
cor_mat <- cor(t(lognorm[utar_ids, ]), t(lognorm[marker_ids, ]), method = "pearson"); write.csv(cor_mat, "spatial_utar_marker_correlations.csv"); hc <- hclust(as.dist(1 - cor_mat))
```

## Evaluation signals

- The SH3BGR uTAR × TNNT2 correlation entry is positive with substantial magnitude at BOTH day 4 and day 14 (matches cardiomyocyte colocalization).
- The RUNX1T1 uTAR shows a positive correlation with COL1A1 at day 4, but its expression (and hence the correlation signal) is absent at day 14.
- The full uTAR-by-marker correlation matrix is written out (spatial_utar_marker_correlations.csv) with rows = 11 day-4 spatial uTARs and columns = HBZ, TNNT2, COL1A1.
- Hierarchical clustering on the correlation values yields a dendrogram whose groupings are consistent with the qualitative colocalization pattern reported in Fig. 4b.
- uTAR inputs were matched to scRNA-seq differential uTARs by both chromosome position and strandedness, resolving multi-matches by highest total expression.

## Limitations

- Colocalization validation is qualitative — it relies on the sign/magnitude of Pearson correlation entries rather than formal statistical significance testing.
- The day-14 Visium dataset is small (1995 spots from a single heart vs 747 day-4 spots from 5 hearts), so day-14 correlation estimates may be noisier and less well replicated.
- groHMM bed files may fail to generate (e.g., 'cannot read: chr*_HMM.bed: No such file or directory') if groHMM/rtracklayer are not manually installed, which blocks the uTAR annotations needed for this correlation analysis.
- When several spatial uTARs overlap one scRNA-seq uTAR, only the highest-total-expression representative is kept, so alternative overlapping uTARs are not independently tested.

## Evidence

- [methods] Compute Pearson correlations of log-e-normalized spot expression between the 11 day-4 spatial uTARs and the canonical gene markers HBZ (red blood cells), TNNT2 (myocytes), and COL1A1 (epicardial progenitors): [evidence span withheld — anchor status: not_found]
- [methods] Perform hierarchical clustering on the resulting correlation values to build a dendrogram analogous to Fig. 4b.: [evidence span withheld — anchor status: not_found]
- [results] the SH3BGR-related uTAR colocalized with the cardiomyocyte marker TNNT2 at both day 4 and day 14, and the RUNX1T1 uTAR (expressed at day 4 only) colocalized with COL1A1+ epicardial spots, confirming scRNA-seq findings.: [evidence span withheld — anchor status: not_found]
- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [abstract] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs: "generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs"
