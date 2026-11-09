---
name: random-geneset-null-distribution-sampling
description: Use when quantifying pathway/module activity from a normalized gene expression
  matrix and you need empirical p-values for PC1-based activity statistics (L1, Median
  Exp) that distinguish shifted from overdispersed gene sets.
when_to_use_negative:
- The expression matrix is raw/unnormalized — rRoma works with normalized data (e.g.
  cpm from edgeR or vst from DESeq2)
- The dataset contains many missing values that have not been imputed (imputation
  via the mice low-rank PCA procedure is recommended beforehand)
- The analysis goal is gene-level differential expression rather than gene-set/module
  activity scoring
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: rROMA
  role: R package implementing the random gene set null distribution procedure for
    L1 and Median Exp significance testing
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Computes only the first principal component (first column of V and first value
    of S) for the observed and random gene sets
- name: mice
  role: Imputes missing values via iterative low-rank PCA on the global expression
    matrix before significance computation
- name: R
  role: Runtime environment for the rROMA package
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:798c55ddecac173451f37c3c056c5849a809425c8eb9dffeb540ef3de41590fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Random gene-set null distribution sampling

## Summary

Assess statistical significance of gene set activity (as implemented in rROMA) by generating null distributions of L1 (% variance explained by PC1) and Median Exp (median sample projection on PC1) statistics from randomly generated gene sets of matched size, then classifying modules as shifted or overdispersed by empirical ppv p-values (BH-corrected) with a 0.05 threshold.

## When to use

Use when quantifying pathway/module activity from a normalized gene expression matrix and you need empirical p-values for PC1-based activity statistics (L1, Median Exp) that distinguish shifted from overdispersed gene sets.

## When NOT to use

- The expression matrix is raw/unnormalized — rRoma works with normalized data (e.g. cpm from edgeR or vst from DESeq2)
- The dataset contains many missing values that have not been imputed (imputation via the mice low-rank PCA procedure is recommended beforehand)
- The analysis goal is gene-level differential expression rather than gene-set/module activity scoring

## Inputs

- Normalized, globally centered gene expression matrix (genes as rows, samples as columns; e.g. cpm-normalized RNA-seq from GSE176121 or the 9771-protein x 122-sample breast cancer proteomic matrix)
- Module list of gene sets to test (e.g. 50 MSigDB v6.2 Hallmark sets loaded via SelectFromMSIGdb or ReadGMTFile)
- Parameters: nSamples (number of random gene sets, default 100), ApproxSamples (percentage governing shared background sets across similarly sized modules), PCADims, fixed random seed

## Outputs

- ModuleMatrix table with per-module L1, Median Exp, ppv L1, ppv Median Exp, and BH-adjusted q L1 and q Median Exp
- Module classification: shifted (ppv Median Exp <= 0.05) vs overdispersed (ppv L1 <= 0.05)
- Empirical null distributions of L1 and Median Exp for representative gene set sizes on a log-spaced grid

## How to apply

After centering the global expression matrix so every gene has mean zero across samples, compute PC1 per gene set via SVD (rROMA uses the R irlba package to compute only the first principal component, with PCADims = 2), deriving L1 (% variance explained by PC1) and Median Exp (median of sample projections on PC1). Build null distributions by sampling nSamples = 100 random gene sets matched in size, sharing the same background sets across modules whose sizes differ by less than the ApproxSamples percentage; rROMA constructs these null distributions for a representative list of gene set sizes uniformly distributed in the logarithmic scale between the minimum and maximum module size. Compute empirical ppv p-values by comparing each module's observed L1 and Median Exp against its matched null distribution, then Benjamini-Hochberg correct them (q L1, q Median Exp). Classify a module as shifted if 'ppv Median Exp' <= 0.05 and as overdispersed if 'ppv L1' <= 0.05, assembling all statistics into the ModuleMatrix table.

## Related tools

- **rROMA** (R package implementing the random gene set null distribution procedure for L1 and Median Exp significance testing) — https://github.com/sysbio-curie/rROMA
- **irlba** (Computes only the first principal component (first column of V and first value of S) for the observed and random gene sets)
- **mice** (Imputes missing values via iterative low-rank PCA on the global expression matrix before significance computation)
- **R** (Runtime environment for the rROMA package)

## Examples

```
library(rRoma); rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- ModuleMatrix contains finite ppv and q values (ppv L1, ppv Median Exp, q L1, q Median Exp) for every tested module
- Classification threshold consistency: modules labeled shifted have 'ppv Median Exp' <= 0.05; modules labeled overdispersed have 'ppv L1' <= 0.05
- Reproduction check: running rRoma.R on MatData with the 50 MSigDB v6.2 Hallmark sets calls 3 of 50 pathways significant (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION)
- Breast cancer proteomic reproduction: 34 shifted and 6 overdispersed pathways at p < 0.05
- Null sampling uses a fixed seed so results are reproducible across runs

## Limitations

- Default values of hyperparameters (e.g. extreme-weight percentage, ApproxSample, number of PCs) are not reported in the article and must be chosen by the practitioner
- The CF case study lacked statistical power; the authors note it would require additional transcriptional datasets and testing of several gene set reference databases
- rRoma recommends using datasets for which missing values have already been imputed rather than relying on the mice imputation
- Random gene set background sharing via the ApproxSamples percentage is an approximation: null distributions are built for representative gene set sizes on a logarithmic grid, not for every exact size

## Evidence

- [discussion] uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution.: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [discussion] rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale between the minimum and maximum size of the: "rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale between the minimum and maximum size of the"
- [results] a gene set is classified as shifted if ppv Median Exp < 0.05 and as overdispersed if ppv L1 < 0.05, with ppv values corrected for multiple testing by the Benjamini and Hochberg method.: [evidence span withheld — anchor status: not_found]
- [readme] rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation: "rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [readme] You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
