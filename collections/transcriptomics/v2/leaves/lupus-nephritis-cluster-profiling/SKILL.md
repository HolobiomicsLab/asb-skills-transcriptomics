---
name: lupus-nephritis-cluster-profiling
description: Use when measuring how robust ICELLNET communication scores between lupus
  nephritis clusters (CM3 dendritic cells vs CT0a/CT3b T cells) are to random cell
  subsampling, i.e. when cluster size may inflate or destabilize inferred communication.
when_to_use_negative:
- Input is bulk or sorted-population transcriptomic data with no per-cell data, since
  there is nothing to randomly subsample to build a variability distribution
- Goal is a single point estimate of the communication score between two clusters
  without uncertainty estimation
- 'Non-human datasets: the ICELLNET database is curated exclusively from human studies'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ICELLNET
  role: Computes the communication score between cluster mean expression profiles
    (CM3 vs CT0a/CT3b) against the manually curated ligand-receptor database (1669
    interactions) for every subsample
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Runtime for the icellnet package and the subsampling loop (fixed-seed random
    draws, sd aggregation across 20 draws per percentage)
- name: devtools
  role: Installs the icellnet package from GitHub with install_github('soumelis-lab/ICELLNET',
    ref='master', subdir='icellnet')
- name: ggplot2
  role: Plots the standard deviation of the communication score as a function of subsampling
    percentage for CM3/CT0a/CT3b
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:ed0f1f7ee38715603ad7798ca448eb46c66a4a4cb9d86128f2118d04572de198
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lupus-nephritis-cluster-profiling

## Summary

Subsampling-based robustness profiling of ICELLNET communication scores between lupus nephritis scRNA-seq clusters (CM3 dendritic cells vs CT0a/CT3b T cells): repeated random draws of cluster cells quantify how the standard deviation of the score scales with the fraction of cells used.

## When to use

Use when measuring how robust ICELLNET communication scores between lupus nephritis clusters (CM3 dendritic cells vs CT0a/CT3b T cells) are to random cell subsampling, i.e. when cluster size may inflate or destabilize inferred communication.

## When NOT to use

- Input is bulk or sorted-population transcriptomic data with no per-cell data, since there is nothing to randomly subsample to build a variability distribution
- Goal is a single point estimate of the communication score between two clusters without uncertainty estimation
- Non-human datasets: the ICELLNET database is curated exclusively from human studies

## Inputs

- Per-cell expression matrix with cluster assignments for lupus nephritis clusters CM3 (dendritic cells), CT0a and CT3b (T cells) from scRNA-seq
- Cluster mean expression profiles for CM3, CT0a, CT3b
- ICELLNET curated ligand-receptor interactions database (1669 interactions)

## Outputs

- ICELLNET communication scores for each of 20 independent draws at each subsampling percentage (10%-90%, 10% increments)
- Table of score standard deviations per cluster and percentage (subsampling_sd_results.csv)
- ggplot2 plot of SD of the communication score as a function of subsampling percentage for CM3/CT0a/CT3b

## How to apply

Load the cluster mean expression profiles (CM3, CT0a, CT3b) together with the curated ICELLNET ligand-receptor database (1669 interactions) in R, installing the package via devtools::install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet'). For each subsampling percentage from 10% to 90% in 10% increments, draw 20 independent random subsamples of the cluster's cells using a fixed random seed for reproducibility. Recompute the ICELLNET communication score with the icellnet score implementation for each of the 20 draws at every percentage. Compute the standard deviation of the 20 scores at each subsampling percentage per cluster, plot SD versus subsampling percentage for CM3/CT0a/CT3b with ggplot2, and write the SD table to subsampling_sd_results.csv. Rationale: the reported SD is anti-correlated with the percentage of cells subsampled (SD 37/59 at 90% vs 146/215 at 10% for CT0a/CT3b), so a steep SD rise at low percentages flags communication scores that are unstable due to small effective cluster size.

## Related tools

- **ICELLNET** (Computes the communication score between cluster mean expression profiles (CM3 vs CT0a/CT3b) against the manually curated ligand-receptor database (1669 interactions) for every subsample) — github.com/soumelis-lab/ICELLNET
- **R** (Runtime for the icellnet package and the subsampling loop (fixed-seed random draws, sd aggregation across 20 draws per percentage))
- **devtools** (Installs the icellnet package from GitHub with install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet'))
- **ggplot2** (Plots the standard deviation of the communication score as a function of subsampling percentage for CM3/CT0a/CT3b)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")
```

## Evaluation signals

- SD at 10% subsampling is markedly higher than at 90%: approximately 146 (CT0a) and 215 (CT3b) versus approximately 37 and 59, reproducing the reported inverse relationship
- SD of the communication score decreases monotonically as the subsampling percentage increases from 10% to 90% for both CM3-CT0a and CM3-CT3b pairs
- Using a fixed random seed, the 20 independent draws per percentage reproduce the same SD values across runs
- subsampling_sd_results.csv contains one SD value per cluster per percentage across all nine subsampling levels (10%-90%)
- Scores are recomputed with the icellnet score implementation against the 1669-interaction curated database, matching the version noted in the article

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x
- At the time of the paper, the ICELLNET database is very specific and exhaustive for cytokine interactions but excludes chemokine and checkpoint interactions
- Score variability depends on effective cluster size: low-percentage subsamples of small clusters yield highly unstable scores (SD up to 146/215 at 10%)
- ICELLNET does not integrate downstream effectors, so scores capture ligand-receptor co-expression rather than signaling activity

## Evidence

- [abstract] We develop ICELLNET, a transcriptomic-based framework integrating: 1) an original expert-curated database of ligand-receptor interactions: "We develop ICELLNET, a transcriptomic-based framework integrating: 1) an original expert-curated database of ligand-receptor interactions"
- [methods] ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x.: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [methods] New structure and extension of ICELLNET database up to 1669 interactions.: "New structure and extension of ICELLNET database up to 1669 interactions."
- [readme] **Versatile** tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics): "**Versatile** tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics)"
