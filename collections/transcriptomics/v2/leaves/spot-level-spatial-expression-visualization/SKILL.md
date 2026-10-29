---
name: spot-level-spatial-expression-visualization
description: Use when validating that uTAR signals recapitulate tissue biology in
  10X Visium data — i.e., when spot-level expression of annotation-independent TARs
  must be compared against marker genes (e.g., TNNT2, COL1A1, HBZ) via spatial colocalization
  or correlation.
when_to_use_negative:
- The input is already a per-cell or per-spot feature expression table with no spatial
  coordinates and no paired marker genes to compare against
- The goal is de novo TAR discovery from raw scRNA-seq FASTQs rather than validating
  existing uTARs in spatial sections
- The Visium section derives from a tissue/organism for which no scRNA-seq uTARs or
  marker gene sets have been established
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: Seurat
  role: Loads the Visium spatial count matrices and log-e-normalizes spot feature
    expression for spatial analysis and plotting
- name: groHMM
  role: Hidden-Markov-model calling of transcriptionally active regions (TARs) from
    Space-Ranger-derived alignments to generate spatial uTAR annotations
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: Space Ranger
  role: Produces the processed Visium count matrices and alignment files that feed
    the spatial analysis
- name: rtracklayer
  role: Loads groHMM-generated TAR BED/GTF annotation files in R
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: Wraps the TAR-scRNA-seq pipeline rules (e.g., groHMM HMM rule calcHMMrefFlat)
    that generate the TAR expression matrices used in the spatial analysis
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:30c2f760fbccc2779d23aa6af6af02f81580d1bf9bd8592d27b1e306b508de5e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Spot-level spatial expression visualization

## Summary

Visualize and statistically validate spot-level spatial co-expression of transcriptionally active regions (TARs/uTARs) with canonical cell-type marker genes on 10X Visium sections, using Seurat-normalized spot matrices, groHMM-derived spatial TAR annotations, Pearson correlation, and hierarchical clustering.

## When to use

Use when validating that uTAR signals recapitulate tissue biology in 10X Visium data — i.e., when spot-level expression of annotation-independent TARs must be compared against marker genes (e.g., TNNT2, COL1A1, HBZ) via spatial colocalization or correlation.

## When NOT to use

- The input is already a per-cell or per-spot feature expression table with no spatial coordinates and no paired marker genes to compare against
- The goal is de novo TAR discovery from raw scRNA-seq FASTQs rather than validating existing uTARs in spatial sections
- The Visium section derives from a tissue/organism for which no scRNA-seq uTARs or marker gene sets have been established

## Inputs

- Space Ranger-processed 10X Visium spot-by-feature count matrices (day-4: 747 spots from 5 hearts; day-14: 1995 spots from 1 heart)
- Space Ranger alignment files (BAM) for groHMM spatial TAR calling
- Combined TAR + GRCg6a Ensembl v96 genome annotation (GTF/BED)
- Differentially expressed scRNA-seq uTAR list with chromosome positions and strandedness
- Canonical marker gene sets: HBZ (red blood cells), TNNT2 (myocytes), COL1A1 (epicardial progenitors)

## Outputs

- Full uTAR-by-marker Pearson correlation matrix (e.g., spatial_utar_marker_correlations.csv)
- Hierarchical-clustering dendrogram of uTAR-marker correlation values (analogous to Fig. 4b)
- Spot-level spatial feature plots showing uTAR/marker colocalization per section and time point

## How to apply

Load Space Ranger-processed Visium count matrices (spot x feature, e.g., 747 day-4 spots from 5 chicken hearts; 1995 day-14 spots from 1 heart) into Seurat's spatial transcriptome tools together with the combined TAR + GRCg6a Ensembl v96 annotation, and log-e-normalize the feature matrices. Run groHMM on the Space-Ranger alignment files to call spatial TARs, then map each differentially expressed scRNA-seq uTAR to an overlapping spatial uTAR by chromosome position and strandedness, resolving one-to-many overlaps by keeping the spatial uTAR with the highest total expression. Compute Pearson correlations of log-e-normalized spot expression between the spatial uTARs (e.g., 11 day-4 uTARs) and canonical markers such as HBZ (red blood cells), TNNT2 (myocytes), and COL1A1 (epicardial progenitors), for each time-point section. Perform hierarchical clustering on the correlation matrix to build a dendrogram analogous to Fig. 4b, and export the full uTAR-by-marker matrix (e.g., spatial_utar_marker_correlations.csv) for inspection and downstream spot-level feature plots.

## Related tools

- **Seurat** (Loads the Visium spatial count matrices and log-e-normalizes spot feature expression for spatial analysis and plotting)
- **groHMM** (Hidden-Markov-model calling of transcriptionally active regions (TARs) from Space-Ranger-derived alignments to generate spatial uTAR annotations) — https://github.com/fw262/TAR-scRNA-seq
- **Space Ranger** (Produces the processed Visium count matrices and alignment files that feed the spatial analysis)
- **rtracklayer** (Loads groHMM-generated TAR BED/GTF annotation files in R) — https://github.com/fw262/TAR-scRNA-seq
- **Snakemake** (Wraps the TAR-scRNA-seq pipeline rules (e.g., groHMM HMM rule calcHMMrefFlat) that generate the TAR expression matrices used in the spatial analysis) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
Rscript -e 'library(Seurat); vis <- Load10X_Spatial("day4/filtered_feature_bc_matrix/", filter.matrix=TRUE); vis <- SCTransform/NormalizeData(vis, normalization.method="LogNormalize", margin=2); SpatialFeaturePlot(vis, features=c("TNNT2","uTAR_SH3BGR"))'
```

## Evaluation signals

- The SH3BGR-related uTAR shows positive Pearson correlation with TNNT2 (cardiomyocyte marker) in both day-4 and day-14 correlation matrices
- The RUNX1T1 uTAR shows positive correlation with COL1A1+ epicardial spots at day 4 and is absent (no comparable signal) at day 14
- The output CSV has dimensions equal to the number of spatial uTARs (e.g., 11 day-4 uTARs) by the number of markers tested (HBZ, TNNT2, COL1A1) for each time point
- Each scRNA-seq uTAR maps to at most one spatial uTAR, resolved by highest total expression among overlapping candidates matched on chromosome position and strandedness
- Hierarchical clustering of the correlation matrix yields a dendrogram in which marker-correlated uTARs group consistently with the reported Fig. 4b-like structure

## Limitations

- The day-14 Visium dataset derives from only 1 heart (1995 spots) versus 5 hearts at day 4 (747 spots), limiting replicate power at day 14
- groHMM rule failures (e.g., 'chr*_HMM.bed: No such file or directory') leave the pipeline without TAR annotations; groHMM and rtracklayer must be manually installed and loadable in R
- Spatial uTAR detection is restricted to uTARs overlapping differentially expressed scRNA-seq uTARs; uTARs found only in spatial data are not assessed

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs: "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs), performs single-cell TAR expression analysis to identify biologically significant TARs"
- [readme] We have written Snakemake workflows to perform this analysis.: "We have written Snakemake workflows to perform this analysis."
- [abstract] This procedure uses single-cell expression analyses as a filter to direct annotation efforts to biologically significant transcripts: "This procedure uses single-cell expression analyses as a filter to direct annotation efforts to biologically significant transcripts"
