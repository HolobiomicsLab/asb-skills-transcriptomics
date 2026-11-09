---
name: pc1-module-activity-quantification
description: Use when you need sample-wise pathway/module activity scores and significance
  from a normalized gene expression matrix (e.g. edgeR cpm) and a collection of gene
  sets (GMT/MSigDB Hallmark), to find shifted or over-dispersed modules between conditions.
when_to_use_negative:
- Input is raw, unnormalized count data — rRoma explicitly requires normalized matrices
  (cpm, vst, etc.)
- Expression matrix still contains many missing values that were not imputed — the
  README recommends using datasets with missing values already imputed
- The goal is differential gene expression per gene rather than coordinated module/sample-wise
  activity — a DE tool (edgeR/DESeq2) is the wrong lens here
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3518
tools:
- name: rROMA
  role: Core R package that computes PC1-based module activity, tests shifted/over-dispersed
    modules, and filters outlier genes
  repo: https://github.com/sysbio-curie/rROMA
- name: edgeR
  role: Used to normalize the RNA-seq count matrix to cpm prior to running rRoma.R
- name: mice
  role: Provides the iterative PCA method rROMA uses to impute missing values in the
    global expression matrix
- name: irlba
  role: Efficient partial SVD used to compute only the first principal component (PC1)
    of each gene set
- name: GSVA
  role: Companion package for benchmarking rROMA scores against GSVA, PLAGE, ssGSEA
    and z-score sample-wise methods
- name: ROMA (Java)
  role: Original Java implementation of the ROMA algorithm behind the rROMA R package
  repo: https://github.com/sysbio-curie/Roma
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:df3c133fcb834ada59bdd149290d627742007285ba03284e74c4bcb5dd061418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - GSVA
  - R
  - ROMA (Java)
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pc1-module-activity-quantification

## Summary

Quantify gene set (module) activity from normalized omics matrices by computing the first principal component (PC1) of the genes in each set with rROMA, an R implementation of ROMA, and testing shifted vs over-dispersed modules against a random gene set null.

## When to use

Use when you need sample-wise pathway/module activity scores and significance from a normalized gene expression matrix (e.g. edgeR cpm) and a collection of gene sets (GMT/MSigDB Hallmark), to find shifted or over-dispersed modules between conditions.

## When NOT to use

- Input is raw, unnormalized count data — rRoma explicitly requires normalized matrices (cpm, vst, etc.)
- Expression matrix still contains many missing values that were not imputed — the README recommends using datasets with missing values already imputed
- The goal is differential gene expression per gene rather than coordinated module/sample-wise activity — a DE tool (edgeR/DESeq2) is the wrong lens here

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns), e.g. edgeR cpm or DESeq2 vst
- Gene set collection (GMT file or MSigDB selection, e.g. 50 HALLMARK gene sets from MSigDB v6.2)
- Sample labels/metadata defining comparison groups (e.g. CF vs non-CF, IL17+TNFa treated vs control)

## Outputs

- rRoma.R output object containing ModuleMatrix with L1 (variance explained by PC1), Median Exp / L1 p-values and q-values per module
- Sample-wise module activity scores (module eigengene/PC1 scores per sample)
- Per-module classification as shifted vs over-dispersed (significant at 'ppv Median Exp' <= 0.05)
- Outlier gene diagnostics per module (e.g. via PlotOutliers, GetTopContrib)

## How to apply

Start from a normalized expression matrix (genes as rows, samples as columns; e.g. cpm from edgeR — rRoma does not work on raw counts), imputing missing values with the mice-based low-rank PCA approximation. Load gene sets from MSigDB with SelectFromMSIGdb('HALLMARK') (v6.2 in the paper) or from a GMT via ReadGMTFile, optionally stripping prefixes for plotting. Run rRoma.R with a fixed random seed, recommended settings PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, GeneOutDetection='L1OutExpOut', nSamples>=100, obtaining a ModuleMatrix with L1 and 'Median Exp' p/q values; modules are called shifted when 'ppv Median Exp' <= 0.05. The algorithm centers the global matrix, computes PC1 via irlba, orients PC1 to resolve sign ambiguity, filters outlier genes by leave-one-out L1 z-scores (L1OutExpOut), and compares observed L1 against a log-spaced ApproxSample null distribution for gene set sizes.

## Related tools

- **rROMA** (Core R package that computes PC1-based module activity, tests shifted/over-dispersed modules, and filters outlier genes) — https://github.com/sysbio-curie/rROMA
- **edgeR** (Used to normalize the RNA-seq count matrix to cpm prior to running rRoma.R)
- **mice** (Provides the iterative PCA method rROMA uses to impute missing values in the global expression matrix)
- **irlba** (Efficient partial SVD used to compute only the first principal component (PC1) of each gene set)
- **GSVA** (Companion package for benchmarking rROMA scores against GSVA, PLAGE, ssGSEA and z-score sample-wise methods)
- **ROMA (Java)** (Original Java implementation of the ROMA algorithm behind the rROMA R package) — https://github.com/sysbio-curie/Roma

## Examples

```
library(rRoma); AllHall <- SelectFromMSIGdb("HALLMARK"); rRoma.output <- rRoma.R(MatData, AllHall, PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, GeneOutDetection='L1OutExpOut', NReps=100)
```

## Evaluation signals

- Benchmark reproduction: on GSE176121 scenario 1 (CF vs non-CF), rROMA with 50 Hallmark sets yields 3 significantly shifted modules (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION) vs 0 for GSVA/ssGSEA/z-score and 45/50 for PLAGE
- Scenario 2 (IL17+TNFa vs control): TNFA_SIGNALING_VIA_NFKB must rank top by rROMA significance, vs ranks 5/24, 5/27, 7/17 and 18/45 for GSVA/PLAGE/ssGSEA/z-score
- All analyzed modules should have sufficiently high PC1 variance (L1) significance versus the random gene set null; shifted modules meet 'ppv Median Exp' <= 0.05
- Outlier gene filtering stays bounded: the maximum proportion of genes removable as outliers per gene set is limited by a hyperparameter
- Results should be reproducible under a fixed random seed (rRoma.R permutations are stochastic)

## Limitations

- Results depend on stochastic gene set permutations; a fixed seed is needed for reproducibility
- PC1 orientation is ambiguous — several PCSignMode options exist and the choice (e.g. UseMeanExpressionAllWeights with PCSignThr=0.9) affects score direction
- The CF case study in the article was acknowledged as lacking statistical power and would require additional transcriptional datasets and multiple gene set databases
- Hyperparameter defaults (extreme-weight percentage, outlier z-score threshold, ApproxSample grid, distance threshold) are not fully documented; filtering settings may need adaptation per dataset
- rROMA requires normalized input and datasets with missing values preferably already imputed

## Evidence

- [readme] rROMA works with **normalized** data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [discussion] PC1-based activity quantification: "this algorithm implements a linear model of gene regulation and efficiently and reliably quantifies the activity of gene sets by computing the first principal component (PC1)"
- [discussion] Null distribution p-values from L1: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1"
- [methods] Hallmark selection from MSigDB: "we need to define the pathway we want to test in our analysis. We can do that by loading a GMT file"
- [discussion] Three shifted Hallmark modules in CF dataset: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [methods] Running rRoma.R with fixed seed: "You can now run rRoma on our dataset by simply specifying the expression dataset and the modules you want to test. Here we are fixing the seed"
