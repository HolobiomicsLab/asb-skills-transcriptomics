---
name: fisher-exact-test-outlier-filtering
description: Use when running rROMA/Roma PC1-based gene set activity analysis on normalized
  expression matrices and needing to protect outlier-gene removal (leave-one-out L1
  z-scores) from incorrectly excluding genes whose apparent outlier status is not
  specific to individual gene sets.
when_to_use_negative:
- When genes present in fewer than the defined number of gene sets are at stake —
  these are exempt and must not be flagged as outliers
- When data are not normalized — rRoma requires normalized input (e.g. cpm from edgeR
  or vst from DESeq2)
- When you need sample-level filtering rather than gene-level outlier handling — the
  two-step sample filter (number of detected genes plus PCA projection) is a separate
  procedure
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: rROMA
  role: R package implementing the outlier-gene detection with Fisher test safeguard
    (OutlierFisherThr, OutlierRarelyFoundThr, OutliersPerc parameters)
  repo: https://github.com/sysbio-curie/rROMA
- name: PlotOutliers
  role: Visual diagnostics to validate flagged/retained genes and adapt filtering
    hyperparameters
  repo: https://github.com/sysbio-curie/rROMA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:bed94d7fec3c48375beffb61c9cc60b25b90fc793dedd659928b6703a62cb9a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - DetectOutliers
  - PlotOutliers
  - R
  - irlba
  - rROMA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fisher-exact-test-outlier-filtering

## Summary

A safeguard step of rROMA's outlier-gene filtering that uses a Fisher exact test to compare a gene's outlier proportion across gene set collections to the average proportion of all genes, combined with an exemption for genes present in few gene sets and a cap on the removable fraction per gene set.

## When to use

Use when running rROMA/Roma PC1-based gene set activity analysis on normalized expression matrices and needing to protect outlier-gene removal (leave-one-out L1 z-scores) from incorrectly excluding genes whose apparent outlier status is not specific to individual gene sets.

## When NOT to use

- When genes present in fewer than the defined number of gene sets are at stake — these are exempt and must not be flagged as outliers
- When data are not normalized — rRoma requires normalized input (e.g. cpm from edgeR or vst from DESeq2)
- When you need sample-level filtering rather than gene-level outlier handling — the two-step sample filter (number of detected genes plus PCA projection) is a separate procedure

## Inputs

- Normalized gene expression matrix (MatData, genes as rows, samples as columns)
- Module list / GMT file of gene sets (e.g. MSigDB Hallmark sets via SelectFromMSIGdb or ReadGMTFile)
- Per-gene outlier statistics from leave-one-out L1OutExpOut detection (outlier proportions across collections)

## Outputs

- Filtered gene sets with safeguarded outlier assignments used for PC1/L1 activity computation
- Outlier diagnostics from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection) supporting hyperparameter choice

## How to apply

After the leave-one-out step—where for each gene in a gene set the percentage of variance explained by PC1 is recomputed with that gene removed and genes are flagged via z-score normalization of the L1 distribution—apply the Fisher exact test safeguard: compare the average proportion of collections in which genes are outliers to the same proportion for a particular gene (OutlierFisherThr hyperparameter). Exempt any gene present in fewer than a defined number of gene sets (OutlierRarelyFoundThr, set to ~5% of modules tested, e.g. 2 of the 50 MSigDB Hallmark sets) so it is not considered an outlier. Enforce a final filter capping the maximum proportion of genes removable as outliers per gene set (OutliersPerc). Adjust hyperparameters using PlotOutliers diagnostics (Plot_L1, Plot_Weights, Plot_Projection) as visual baselines.

## Related tools

- **rROMA** (R package implementing the outlier-gene detection with Fisher test safeguard (OutlierFisherThr, OutlierRarelyFoundThr, OutliersPerc parameters)) — https://github.com/sysbio-curie/rROMA
- **PlotOutliers** (Visual diagnostics to validate flagged/retained genes and adapt filtering hyperparameters) — https://github.com/sysbio-curie/rROMA

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Genes present in fewer than OutlierRarelyFoundThr gene sets (e.g. < 2 of 50 Hallmark sets) are retained, not flagged as outliers
- The fraction of genes removed per gene set never exceeds the OutliersPerc cap
- Fisher test outcomes agree with the comparison of each gene's cross-collection outlier proportion to the average proportion of all genes
- Final gene weights and L1 values are consistent with the leave-one-out L1 profiles shown in Plot_L1 / Plot_Weights diagnostics
- Analysis runs on a normalized matrix (genes as row names, samples as column names) with no missing values, or values imputed by mice

## Limitations

- Default values of hyperparameters (outlier z-score threshold, Fisher threshold, rare-gene-set exemption count) are modifiable and not fixed a priori; they may need tuning per dataset
- Outlier filtering alters PC1-based activity scores, so decisions should be validated against visual diagnostics rather than treated as fully automatic ground truth
- The rRoma authors note that using a single database (e.g. Hallmark alone) gives a limited view; the Fisher-test cross-collection comparison is most meaningful with multiple combined databases

## Evidence

- [discussion] Fisher test cross-collection comparison: "a Fisher test is performed. This consists of comparing the average proportion of collections in which the genes in the analysis are considered as outliers to this same proportion for a particular"
- [discussion] Rare gene set exemption: "if a gene is present in less than a defined number of gene sets (set by a modifiable hyperparameter), then it is not considered an outlier"
- [discussion] Cap on removable genes: "A final filter therefore exists to limit the maximum proportion of genes that can be considered as aberrant for a given gene set and removed from the analysis"
- [discussion] Leave-one-out L1 basis: "for each gene in the selected gene set, the method calculates the percentage of variance explained by the first principal component (PC1) when that gene is removed from the dataset."
- [methods] Visual hyperparameter validation: "can be visualized on plots and used as a baseline to potentially adapt the filtering hyperparameters"
- [readme] Input requirements: "rRoma requires a gene expression matrix, with column names indicating samples and row names indicating gene names."
