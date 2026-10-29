---
name: single-cell-expression-subsampling
description: Use when ICELLNET communication scores are computed from scRNAseq cluster
  mean profiles and you must quantify how score variability scales with the number
  of cells subsampled from a cluster (e.g. rare clusters, low cell counts).
when_to_use_negative:
- Input is bulk RNAseq or microarray data from sorted populations where no single-cell
  subsampling is possible
- The goal is to compute a single ICELLNET score on a well-populated cluster rather
  than characterize variability across cell sample sizes
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: ICELLNET
  role: Provides the communication score implementation and curated ligand-receptor
    database used to recompute scores on each subsample
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Runtime environment for the subsampling loop and icellnet package (ICELLNET
    v2.0.0 requires R < 4.1; use v2.2.x for R > 4.1.x)
- name: devtools
  role: Used to install the icellnet package from GitHub via install_github
- name: ggplot2
  role: Plots SD of communication score versus subsampling percentage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:97b52f79367de965824a4d4b8b9b05847bc4bfa7f1fdf83a4a2ba21d98eef330
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-expression-subsampling

## Summary

Assesses the robustness of ICELLNET communication scores computed from single-cell cluster mean expression profiles by randomly subsampling cluster cells at decreasing percentages (10%–90%) and measuring the standard deviation of repeated score draws, e.g. for lupus nephritis dendritic cell cluster CM3 versus T-cell clusters CT0a and CT3b.

## When to use

Use when ICELLNET communication scores are computed from scRNAseq cluster mean profiles and you must quantify how score variability scales with the number of cells subsampled from a cluster (e.g. rare clusters, low cell counts).

## When NOT to use

- Input is bulk RNAseq or microarray data from sorted populations where no single-cell subsampling is possible
- The goal is to compute a single ICELLNET score on a well-populated cluster rather than characterize variability across cell sample sizes

## Inputs

- Single-cell cluster mean expression profiles (e.g. CM3 dendritic cells, CT0a and CT3b T cells from lupus nephritis)
- ICELLNET ligand-receptor interactions database (1669 curated interactions)
- Cell-by-gene expression matrix with cluster labels for subsampling

## Outputs

- Table of communication score standard deviations per cluster pair and subsampling percentage (e.g. subsampling_sd_results.csv)
- ggplot2 plot of score SD as a function of subsampling percentage per cluster

## How to apply

Load cluster mean expression profiles (e.g. CM3, CT0a, CT3b) and the expert-curated ICELLNET ligand-receptor database (1669 interactions) in R. For each subsampling percentage from 10% to 90% in 10% increments, randomly draw 20 independent subsamples of the cluster's cells using a fixed random seed for reproducibility. Recompute the ICELLNET communication score for each of the 20 draws at every percentage with the icellnet score implementation, then compute the standard deviation across the 20 draws per percentage and per cluster pair. The rationale is that smaller subsamples give noisier mean-expression profiles, so the SD of the score reveals how many cells are needed for a stable communication estimate. Plot SD versus subsampling percentage (ggplot2) to expose the expected inverse relationship.

## Related tools

- **ICELLNET** (Provides the communication score implementation and curated ligand-receptor database used to recompute scores on each subsample) — github.com/soumelis-lab/ICELLNET
- **R** (Runtime environment for the subsampling loop and icellnet package (ICELLNET v2.0.0 requires R < 4.1; use v2.2.x for R > 4.1.x))
- **devtools** (Used to install the icellnet package from GitHub via install_github)
- **ggplot2** (Plots SD of communication score versus subsampling percentage)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"); library(icellnet)  # then loop 20 draws per percentage 10-90%
```

## Evaluation signals

- SD at 10% subsampling is markedly higher than at 90% (e.g. ~146 vs ~37 for CT0a; ~215 vs ~59 for CT3b), reproducing the inverse relationship
- Each subsampling percentage (10%–90% in 10% steps) yields exactly 20 independent draws with a fixed random seed, giving reproducible SDs
- SD values decrease monotonically (approximately) as subsampling percentage increases for every cluster pair tested
- Output table contains one SD value per cluster pair per percentage and saves cleanly to CSV

## Limitations

- Communication scores derived from very small clusters (low subsampling percentages) are highly unstable, with SDs several-fold larger than at high percentages
- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x, v2.2.x must be used, otherwise installation/behavior may break
- ICELLNET database is human-curated and human-specific (1669 interactions); chemokine and checkpoint interactions were not yet included at publication time

## Evidence

- [results] Communication score variability (standard deviation) was anti-correlated with the percentage of cells subsampled from the cluster: [evidence span withheld — anchor status: not_found]
- [results] SDs of 37 and 59 at 90% versus 146 and 215 at 10%: [evidence span withheld — anchor status: not_found]
- [methods] New structure and extension of ICELLNET database up to 1669 interactions: "New structure and extension of ICELLNET database up to 1669 interactions."
- [readme] ICELLNET v2.0.0 requires R version < 4.1: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [methods] install_github installation call: "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
