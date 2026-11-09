---
name: communication-score-robustness-quantification
description: Use when ICELLNET communication scores are computed between clusters
  of unequal or small cell counts (e.g. lupus nephritis CM3 dendritic cells vs CT0a/CT3b
  T cells) and you need to assess how score variability depends on cluster cell number
  before interpreting score differences.
when_to_use_negative:
- The goal is to compare communication scores between clusters that each contain ample,
  comparable cell numbers — in that case compute scores directly rather than running
  a subsampling sensitivity analysis.
- The communication framework is not ICELLNET and its score is not recomputable from
  mean cluster expression profiles plus the ligand-receptor database, since the draw-and-recompute
  scheme relies on the icellnet score implementation.
- The question is about differential communication between conditions rather than
  the numerical stability of a score with respect to cell sampling.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ICELLNET
  role: Provides the curated ligand-receptor database (1669 interactions) and the
    icellnet score implementation used to recompute communication scores on each subsample
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Analysis environment in which subsampling, score recomputation, and SD computation
    are performed (ICELLNET v2.0.0 requires R < 4.1; use ICELLNET v2.2.x for R > 4.1.x)
- name: devtools
  role: Used to install the ICELLNET package from GitHub via install_github
- name: ggplot2
  role: Used to plot the standard deviation of the communication score as a function
    of subsampling percentage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:52c8d28c90b1e14f9b6d0665ad73e1ed42d87eb50b075ab3f2a8c7b77bc77fe5
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# communication-score-robustness-quantification

## Summary

Quantifies the robustness of ICELLNET communication scores between cell clusters by randomly subsampling cluster cells at percentages from 10% to 90%, recomputing scores across 20 independent draws, and reporting the standard deviation of the score at each subsampling level.

## When to use

Use when ICELLNET communication scores are computed between clusters of unequal or small cell counts (e.g. lupus nephritis CM3 dendritic cells vs CT0a/CT3b T cells) and you need to assess how score variability depends on cluster cell number before interpreting score differences.

## When NOT to use

- The goal is to compare communication scores between clusters that each contain ample, comparable cell numbers — in that case compute scores directly rather than running a subsampling sensitivity analysis.
- The communication framework is not ICELLNET and its score is not recomputable from mean cluster expression profiles plus the ligand-receptor database, since the draw-and-recompute scheme relies on the icellnet score implementation.
- The question is about differential communication between conditions rather than the numerical stability of a score with respect to cell sampling.

## Inputs

- Cluster mean expression profiles for the cell clusters of interest (e.g. CM3 dendritic cell, CT0a and CT3b T-cell clusters from lupus nephritis)
- Cell-level expression matrix or cluster assignments enabling random cell subsampling from each cluster
- ICELLNET curated ligand-receptor interactions database (1669 interactions)

## Outputs

- Table of standard deviations of the communication score per cluster and per subsampling percentage (e.g. subsampling_sd_results.csv)
- ggplot2 plot of communication score SD as a function of subsampling percentage (10%-90%) for each cluster pair (e.g. CM3-CT0a, CM3-CT3b)

## How to apply

Load the cluster mean expression profiles (e.g. CM3, CT0a, CT3b from lupus nephritis) together with the curated ICELLNET ligand-receptor database (1669 interactions) in R. For each subsampling percentage from 10% to 90% in 10% increments, randomly draw 20 independent subsamples of the cluster's cells using a fixed random seed for reproducibility. Recompute the ICELLNET communication score for each of the 20 draws at every percentage using the icellnet score implementation. Compute the standard deviation of the 20 score draws at each subsampling percentage per cluster, plot the SD as a function of subsampling percentage with ggplot2, and export the SD table (e.g. subsampling_sd_results.csv). The rationale is that scores from sparsely sampled clusters are unstable, so score variability (SD) should be interpreted jointly with the fraction of cells subsampled.

## Related tools

- **ICELLNET** (Provides the curated ligand-receptor database (1669 interactions) and the icellnet score implementation used to recompute communication scores on each subsample) — github.com/soumelis-lab/ICELLNET
- **R** (Analysis environment in which subsampling, score recomputation, and SD computation are performed (ICELLNET v2.0.0 requires R < 4.1; use ICELLNET v2.2.x for R > 4.1.x))
- **devtools** (Used to install the ICELLNET package from GitHub via install_github)
- **ggplot2** (Used to plot the standard deviation of the communication score as a function of subsampling percentage)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"); # then for pct in seq(10,90,10): for(i in 1:20) score[i] <- icellnet(CC=mean_exp_subsample, LR=LRdb, family= fam_data)$total_CC; sd(score)
```

## Evaluation signals

- SD of the communication score is anti-correlated with the percentage of cells subsampled: SD at 10% subsampling is markedly higher than at 90%.
- Reported benchmark values are reproduced: SD ≈ 146 (CT0a) and ≈ 215 (CT3b) at 10% subsampling versus ≈ 37 and ≈ 59 at 90% subsampling.
- Exactly 20 independent draws are used per subsampling percentage, with 10 percentage points from 10% to 90% in 10% increments, and a fixed random seed guarantees reproducibility.
- The output CSV contains one SD value per cluster per subsampling percentage (e.g. subsampling_sd_results.csv) matching the plotted ggplot2 curves.

## Limitations

- Communication score variability is high when few cells are subsampled (SDs up to 146 and 215 at 10% for CT0a and CT3b), so scores derived from small clusters are unreliable and must be interpreted cautiously.
- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x, ICELLNET v2.2.x must be used instead.
- ICELLNET includes a relatively small number of interactions compared to other databases and chemokine and checkpoint interactions are not yet included, which bounds the interactions available for score recomputation.

## Evidence

- [results] Communication score variability (standard deviation) was anti-correlated with the percentage of cells subsampled from the cluster, with SDs of 37 and 59 at 90% of cells versus 146 and 215 at 10% for CT0a and CT3b communication.: [evidence span withheld — anchor status: not_found]
- [readme] ICELLNET database up to 1669 interactions.: "New structure and extension of ICELLNET database up to 1669 interactions."
- [methods] install.packages(c("devtools", "jetset", "readxl", "psych", "GGally", "gplots", "ggplot2", ...: "install.packages(c("devtools", "jetset", "readxl", "psych", "GGally", "gplots", "ggplot2", "RColorBrewer", "data.table", "grid", "gridExtra", "ggthemes", "scales","rlist"))"
- [methods] ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x.: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
