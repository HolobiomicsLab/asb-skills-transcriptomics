---
name: gene-set-outlier-thresholding
description: Use when computing PC1/SVD-based gene set (module) activity from a normalized
  expression matrix and you must remove aberrant genes that would distort the percentage
  of variance explained by PC1 (L1) before scoring module activity.
when_to_use_negative:
- Input data is not yet normalized — rRoma requires normalized data (e.g. cpm from
  edgeR or vst from DESeq2) before filtering
- The expression matrix contains missing values that have not been imputed — imputation
  via the mice package / FillNAMethod should happen first, or use datasets with no
  NAs
- The analysis goal is sample-level (not gene-level) quality control — sample outlier
  detection is a separate two-step procedure based on number of detected genes and
  PCA projection in gene space
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: rROMA
  role: R package implementing the leave-one-out outlier-gene detection (DetectOutliers)
    and PC1-based module activity computation
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: R package used to compute only the first principal component (first column
    of V and first value of S) for the leave-one-out L1 recomputation
- name: PlotOutliers
  role: Diagnostic visualization (Plot_L1, Plot_Weights, Plot_Projection) used as
    a baseline to adapt filtering hyperparameters
  repo: https://github.com/sysbio-curie/rROMA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:e0bd5de123040ecdb2cf980586e310325a6407b79463d33575c14630ec7f1572
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gene-set-outlier-thresholding

## Summary

A leave-one-out outlier-gene filtering procedure from rROMA that protects PC1-based gene set activity scores: each gene is temporarily removed, L1 (variance explained by PC1) is recomputed, and genes whose removal abnormally shifts the L1 distribution are flagged via z-score thresholding and excluded with Fisher-test and rarity safeguards.

## When to use

Use when computing PC1/SVD-based gene set (module) activity from a normalized expression matrix and you must remove aberrant genes that would distort the percentage of variance explained by PC1 (L1) before scoring module activity.

## When NOT to use

- Input data is not yet normalized — rRoma requires normalized data (e.g. cpm from edgeR or vst from DESeq2) before filtering
- The expression matrix contains missing values that have not been imputed — imputation via the mice package / FillNAMethod should happen first, or use datasets with no NAs
- The analysis goal is sample-level (not gene-level) quality control — sample outlier detection is a separate two-step procedure based on number of detected genes and PCA projection in gene space

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns; e.g. cpm-normalized RNA-seq or proteomic data)
- Module/gene set list loaded from a GMT file or via SelectFromMSIGdb (e.g. MSigDB Hallmark sets)
- Filtering hyperparameters: GeneOutDetection ('L1OutExpOut'), GeneOutThr (L1 z-score threshold), OutlierFisherThr, OutlierRarelyFoundThr, OutliersPerc

## Outputs

- Per-gene outlier statistics (leave-one-out L1 value, z-score, outlier status, module membership) exportable as outlier_gene_report.csv
- Cleaned gene sets with outlier genes excluded, used for the final PC1/L1 activity computation
- Diagnostic plots from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection) for visual validation of flagged/retained genes

## How to apply

Start from a normalized gene expression matrix (genes as rows, samples as columns, e.g. cpm from edgeR) and a module list such as MSigDB Hallmark GMT sets. For each gene in a gene set, recompute L1 (percentage of variance explained by PC1, computed via SVD with the R irlba package) with that gene removed — the leave-one-out mechanism exposed by DetectOutliers with GeneOutDetection = 'L1OutExpOut'. Z-score normalize the leave-one-out L1 distribution across the module's genes and flag genes whose z-score exceeds the GeneOutThr threshold as outliers. Apply safeguards: a Fisher exact test (OutlierFisherThr) comparing each gene's outlier proportion across module collections to the average proportion of all genes, exemption of genes present in fewer than OutlierRarelyFoundThr gene sets (e.g. ~5% of modules tested), and an OutliersPerc cap on the maximum fraction of genes removable per module. Rationale: genes that dominate or corrupt PC1 inflate or deflate L1, so their exclusion makes the activity score reflect coordinated expression of the whole module rather than single-gene artifacts.

## Related tools

- **rROMA** (R package implementing the leave-one-out outlier-gene detection (DetectOutliers) and PC1-based module activity computation) — https://github.com/sysbio-curie/rROMA
- **irlba** (R package used to compute only the first principal component (first column of V and first value of S) for the leave-one-out L1 recomputation)
- **PlotOutliers** (Diagnostic visualization (Plot_L1, Plot_Weights, Plot_Projection) used as a baseline to adapt filtering hyperparameters) — https://github.com/sysbio-curie/rROMA

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- The per-gene report table is produced with all fields populated (leave-one-out L1, z-score, outlier status, module membership)
- Flagged/retained genes are consistent with the visual diagnostics from PlotOutliers (Plot_L1, Plot_Weights, Plot_Projection)
- The fraction of genes removed as outliers in any single module does not exceed the OutliersPerc cap
- Genes exempted by the rarity rule (present in fewer than OutlierRarelyFoundThr gene sets) are not counted as outliers
- Recomputing L1 after exclusion yields a PC1 variance value consistent with the non-outlier leave-one-out distribution

## Limitations

- Default values of hyperparameters (e.g. outlier z-score threshold GeneOutThr, extreme-weight percentage) are not reported in the article and must be chosen by the user, guided by PlotOutliers diagnostics
- Filtering is conservative by design: genes present in fewer than a defined number of gene sets are never considered outliers, and a maximum proportion of genes removable per gene set is enforced, so genuinely aberrant rare genes may be retained
- Filtering hyperparameters may need to be adapted per dataset; the article states plots 'can be visualized on plots and used as a baseline to potentially adapt the filtering hyperparameters'

## Evidence

- [discussion] Leave-one-out L1 recomputation: "for each gene in the selected gene set, the method calculates the percentage of variance explained by the first principal component (PC1) when that gene is removed from the dataset."
- [discussion] Rarity exemption safeguard: "if a gene is present in less than a defined number of gene sets (set by a modifiable hyperparameter), then it is not considered an outlier"
- [discussion] Fisher test safeguard: "a Fisher test is performed. This consists of comparing the average proportion of collections in which the genes in the analysis are considered as outliers to this same proportion for a particular"
- [discussion] OutliersPerc cap: "A final filter therefore exists to limit the maximum proportion of genes that can be considered as aberrant for a given gene set and removed from the analysis"
- [readme] Expected input format: "rRoma requires a gene expression matrix, with column names indicating samples and row names indicating gene names."
