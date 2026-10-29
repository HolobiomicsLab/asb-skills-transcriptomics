---
name: gene-module-membership-outlier-exemption
description: Use when applying leave-one-out outlier detection (L1OutExpOut) to gene
  sets and needing to prevent over-removal of genes, e.g. genes flagged as outliers
  in many collections or belonging to only a few gene sets.
when_to_use_negative:
- The analysis goal is to remove outlier samples rather than outlier genes — rROMA
  handles sample filtering separately via detected-gene counts and PCA projection
  with a distance threshold.
- The gene set is so small that the maximum removable-outlier proportion cap would
  invalidate the PC1 computation, or the expression matrix is not normalized (rRoma
  requires normalized data, e.g. edgeR cpm or DESeq2 vst).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_2269
tools:
- name: rROMA
  role: R package implementing leave-one-out outlier detection with Fisher-test, rare-membership
    exemption, and proportion-cap safeguards; provides DetectOutliers and PlotOutliers
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Computes only the first principal component (first column of V, first value
    of S) used for the leave-one-out L1 recomputation
- name: mice
  role: Iterative PCA imputation of missing values in the global expression matrix
    before PC1/outlier computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:92272ff72fe59c879b048588334cccab4df132b87901f5c9939d3e58a5948cf1
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gene-module-membership-outlier-exemption

## Summary

rROMA's safeguard layer for leave-one-out outlier gene detection: exemptions and caps that prevent legitimate genes from being wrongly excluded when recomputing PC1-explained variance (L1) per gene set. It combines a Fisher test on outlier proportions across gene set collections, a rare-membership exemption, and a per-module cap on removable genes.

## When to use

Use when applying leave-one-out outlier detection (L1OutExpOut) to gene sets and needing to prevent over-removal of genes, e.g. genes flagged as outliers in many collections or belonging to only a few gene sets.

## When NOT to use

- The analysis goal is to remove outlier samples rather than outlier genes — rROMA handles sample filtering separately via detected-gene counts and PCA projection with a distance threshold.
- The gene set is so small that the maximum removable-outlier proportion cap would invalidate the PC1 computation, or the expression matrix is not normalized (rRoma requires normalized data, e.g. edgeR cpm or DESeq2 vst).

## Inputs

- Normalized gene expression matrix (MatData: genes as rows, samples as columns)
- Module/gene set list (e.g. MSigDB Hallmark GMT file)
- Per-gene leave-one-out L1 values and outlier flags per gene set
- Hyperparameters: GeneOutDetection, GeneOutThr, OutlierFisherThr, OutlierRarelyFoundThr, OutliersPerc

## Outputs

- Per-gene outlier report (leave-one-out L1, z-score, outlier status, module membership)
- Filtered gene sets with exempted genes retained for PC1 activity computation
- Diagnostic plots from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection)

## How to apply

After the leave-one-out step — where rROMA recomputes L1 (percentage of variance explained by PC1 via the irlba SVD) with each gene removed and flags genes by z-score of the L1 distribution against GeneOutThr — apply the safeguards. First, run a Fisher test comparing the average proportion of collections in which a gene is considered an outlier to the same proportion for that particular gene; suspicious genes can be exempted. Second, exempt any gene present in fewer than a defined number of gene sets (a modifiable hyperparameter; the source workflow suggests ~5% of tested modules, e.g. 2 of the 50 MSigDB Hallmark sets). Third, enforce a final cap on the maximum proportion of genes removable as outliers per gene set (OutliersPerc). The rationale is that leave-one-out L1 shifts can be spurious for genes with narrow module membership, so membership-level statistics correct the per-module signal.

## Related tools

- **rROMA** (R package implementing leave-one-out outlier detection with Fisher-test, rare-membership exemption, and proportion-cap safeguards; provides DetectOutliers and PlotOutliers) — https://github.com/sysbio-curie/rROMA
- **irlba** (Computes only the first principal component (first column of V, first value of S) used for the leave-one-out L1 recomputation)
- **mice** (Iterative PCA imputation of missing values in the global expression matrix before PC1/outlier computation)

## Examples

```
rRoma.R(MatData, AllHall, GeneOutDetection = 'L1OutExpOut')
```

## Evaluation signals

- Per-gene report contains leave-one-out L1, z-score, outlier status, and module membership for every gene in every module with no missing fields.
- No gene set loses more genes than the OutliersPerc cap allows; genes present in fewer than the rare-membership threshold of gene sets are retained.
- Fisher test of each gene's outlier proportion against the average proportion across collections does not flag genes exempted by rare membership.
- Flagged/retained genes are consistent with the visual diagnostics from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection).
- Re-running with a fixed seed reproduces the same outlier flags for the same module list (e.g. the 50 MSigDB Hallmark sets).

## Limitations

- Default values of the filtering hyperparameters (outlier z-score threshold, extreme-weight percentage, OutliersPerc cap) are not reported in the article and must be set by the user; PlotOutliers plots are suggested as a baseline to adapt them.
- No software versions for R, irlba, or mice are stated, so exact numerical reproducibility of leave-one-out L1 values may vary.
- Exempting rarely-found genes can mask true outliers that simply occur in few modules; the Fisher test safeguard only compares proportions across collections, not biological plausibility.

## Evidence

- [discussion] Leave-one-out L1 recomputation basis: "for each gene in the selected gene set, the method calculates the percentage of variance explained by the first principal component (PC1) when that gene is removed from the dataset."
- [discussion] Rare-membership exemption: "if a gene is present in less than a defined number of gene sets (set by a modifiable hyperparameter), then it is not considered an outlier"
- [discussion] Fisher test safeguard: "a Fisher test is performed. This consists of comparing the average proportion of collections in which the genes in the analysis are considered as outliers to this same proportion for a particular"
- [discussion] Maximum removable proportion cap: "A final filter therefore exists to limit the maximum proportion of genes that can be considered as aberrant for a given gene set and removed from the analysis"
- [methods] Tuning filtering hyperparameters via plots: "can be visualized on plots and used as a baseline to potentially adapt the filtering hyperparameters"
