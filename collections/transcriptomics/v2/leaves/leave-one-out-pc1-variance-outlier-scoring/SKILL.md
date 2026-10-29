---
name: leave-one-out-pc1-variance-outlier-scoring
description: Use when computing PC1-based gene set activity scores (e.g. rROMA on
  MSigDB Hallmark sets) and genes within a set distort the one-dimensional variance,
  so aberrant genes must be excluded before scoring.
when_to_use_negative:
- Genes already belong to fewer than the OutlierRarelyFoundThr number of gene sets
  (e.g. <~2 of 50 Hallmark sets) — the article states such genes are exempt and not
  considered outliers
- The goal is outlier detection of samples, not genes — the article uses a separate
  two-step sample filtering based on detected-gene counts and PCA projection in gene
  space with a distance threshold
- Data are raw counts — rRoma requires normalized data (e.g. cpm from edgeR or vst
  from DESeq2) before any PC1 computation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: rROMA
  role: Implements the leave-one-out L1 outlier detection (DetectOutliers with GeneOutDetection='L1OutExpOut')
    and PlotOutliers diagnostics
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Efficiently computes only the first principal component (first column of V
    and first value of S) for each leave-one-out recomputation
- name: mice
  role: Imputes missing values in the global expression matrix via iterative PCA before
    PC1/outlier computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:8c9464b5a2d0ed8f5a0fa162e193d172f47bce6f42e78fc73af6bdb6f26bd299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - DetectOutliers
  - PlotOutliers
  - R
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# leave-one-out-pc1-variance-outlier-scoring

## Summary

rROMA's outlier gene detection procedure: for each gene in a gene set, recompute the percentage of variance explained by PC1 (L1) with that gene removed (leave-one-out), z-score normalize the resulting L1 distribution, and flag genes whose removal abnormally changes L1 as outliers, with Fisher-test and rare-gene safeguards.

## When to use

Use when computing PC1-based gene set activity scores (e.g. rROMA on MSigDB Hallmark sets) and genes within a set distort the one-dimensional variance, so aberrant genes must be excluded before scoring.

## When NOT to use

- Genes already belong to fewer than the OutlierRarelyFoundThr number of gene sets (e.g. <~2 of 50 Hallmark sets) — the article states such genes are exempt and not considered outliers
- The goal is outlier detection of samples, not genes — the article uses a separate two-step sample filtering based on detected-gene counts and PCA projection in gene space with a distance threshold
- Data are raw counts — rRoma requires normalized data (e.g. cpm from edgeR or vst from DESeq2) before any PC1 computation

## Inputs

- Normalized gene expression matrix (MatData: genes as rows, samples as columns; e.g. edgeR cpm or DESeq2 vst normalized)
- Module list of gene sets (e.g. MSigDB Hallmark v6.2 sets loaded via SelectFromMSIGdb or ReadGMTFile from a GMT file)

## Outputs

- Per-gene outlier statistics: leave-one-out L1, z-score, outlier status, module membership (e.g. outlier_gene_report.csv)
- Outlier diagnostic plots (Plot_L1, Plot_Weights, Plot_Projection) usable to adapt filtering hyperparameters
- Cleaned gene sets on which robust PC1-based activity scores are computed

## How to apply

For each gene in a selected gene set, calculate the percentage of variance explained by PC1 (L1) when that gene is removed from the dataset, recomputing PC1 via SVD with the R irlba package which computes only the first column of V and first value of S. Z-score normalize the leave-one-out L1 distribution across the module's genes and flag genes whose z-score exceeds the GeneOutThr threshold (GeneOutDetection = 'L1OutExpOut'). Apply safeguards: a Fisher exact test (OutlierFisherThr) comparing each gene's outlier proportion across gene set collections to the average proportion of all genes; genes present in fewer than a defined number of gene sets (OutlierRarelyFoundThr, e.g. ~5% of modules tested, about 2 of 50 Hallmark sets) are not considered outliers; and OutliersPerc caps the maximum fraction of genes removable per gene set. Visual diagnostics from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection) should confirm flagged/retained genes are consistent with the leave-one-out L1 profiles, and hyperparameters can be adapted based on these plots.

## Related tools

- **rROMA** (Implements the leave-one-out L1 outlier detection (DetectOutliers with GeneOutDetection='L1OutExpOut') and PlotOutliers diagnostics) — https://github.com/sysbio-curie/rROMA
- **irlba** (Efficiently computes only the first principal component (first column of V and first value of S) for each leave-one-out recomputation)
- **mice** (Imputes missing values in the global expression matrix via iterative PCA before PC1/outlier computation)

## Examples

```
library(rRoma); rRoma.output <- rRoma.R(MatData, AllHall, GeneOutDetection = 'L1OutExpOut')
```

## Evaluation signals

- Every leave-one-out L1 value, z-score, and outlier status in the per-gene report is populated (no missing fields)
- Genes flagged as outliers have z-scores exceeding GeneOutThr on the normalized leave-one-out L1 distribution
- The proportion of genes removed as outliers per gene set does not exceed the OutliersPerc cap
- Flagged/retained genes match the visual diagnostics from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection)
- Genes present in fewer than OutlierRarelyFoundThr gene sets are retained (not flagged), and Fisher-test safeguards were applied

## Limitations

- Default hyperparameter values (outlier z-score threshold GeneOutThr, OutlierFisherThr, OutlierRarelyFoundThr, OutliersPerc) are not reported in the article and must be tuned, guided by PlotOutliers diagnostics
- Outlier decisions are relative: a gene's outlier status is assessed against the L1 distribution of its own gene set and cross-collection Fisher tests, so results depend on the gene set collection tested
- Removing too many genes from a set is capped by OutliersPerc, so a genuinely aberrant majority of genes cannot be fully excluded

## Evidence

- [discussion] Leave-one-out L1 recomputation rationale: "for each gene in the selected gene set, the method calculates the percentage of variance explained by the first principal component (PC1) when that gene is removed from the dataset."
- [discussion] Fisher test safeguard: "a Fisher test is performed. This consists of comparing the average proportion of collections in which the genes in the analysis are considered as outliers to this same proportion for a particular"
- [discussion] Rare-gene exemption: "if a gene is present in less than a defined number of gene sets (set by a modifiable hyperparameter), then it is not considered an outlier"
- [discussion] Proportion cap: "A final filter therefore exists to limit the maximum proportion of genes that can be considered as aberrant for a given gene set and removed from the analysis"
- [discussion] irlba efficiency: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component, by computing only the first column of V and the first value of S."
- [methods] PlotOutliers as hyperparameter baseline: "can be visualized on plots and used as a baseline to potentially adapt the filtering hyperparameters"
- [readme] Normalized data requirement: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2"
