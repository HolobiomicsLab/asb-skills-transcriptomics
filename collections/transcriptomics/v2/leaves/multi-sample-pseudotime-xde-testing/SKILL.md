---
name: multi-sample-pseudotime-xde-testing
description: Use when analyzing multi-sample scRNA-seq pseudotime trajectories to
  identify genes whose expression along pseudotime differs across a covariate (e.g.
  sex, disease severity) while adjusting for batch effects.
when_to_use_negative:
- Data are from a single biological sample or lack multiple replicates per condition
  — Lamian's cross-sample variability inference and XDE testing require multiple samples
  per condition
- The trajectory was inferred with Monocle2, Monocle3, or Slingshot and you need Lamian
  modules 1 and 2 (topology/branch analysis), which do not support those trajectories
- You only need pseudotime trajectory construction or topology testing, not differential
  expression along pseudotime across covariates
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Lamian
  role: 'Primary framework: lamian_test_h5 with test.type=''variable'' and permutation
    testing performs XDE detection with batch adjustment; saveh5, getPopulationFit,
    getCovariateGroupDiff, clusterGene, plotXDEHm, plotClusterMeanAndDiff, and cellPropTest
    support the workflow'
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default method used by Lamian to construct pseudotime trajectories prior to
    XDE testing
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:dd7dd67a4baf5acfdce0ae2ba8ba266d62719ad4b4910c88045100ae67741d0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multi-sample-pseudotime-xde-testing

## Summary

Run Lamian's differential pseudotime gene-expression (XDE) testing across multiple scRNA-seq samples while adjusting for batch effects, using permutation-based likelihood ratio tests and sample-level variability modeling to detect covariate-associated changes in expression along pseudotime.

## When to use

Use when analyzing multi-sample scRNA-seq pseudotime trajectories to identify genes whose expression along pseudotime differs across a covariate (e.g. sex, disease severity) while adjusting for batch effects.

## When NOT to use

- Data are from a single biological sample or lack multiple replicates per condition — Lamian's cross-sample variability inference and XDE testing require multiple samples per condition
- The trajectory was inferred with Monocle2, Monocle3, or Slingshot and you need Lamian modules 1 and 2 (topology/branch analysis), which do not support those trajectories
- You only need pseudotime trajectory construction or topology testing, not differential expression along pseudotime across covariates

## Inputs

- Pre-processed gene expression matrix (e.g. GSE158769_exprs_raw.tsv.gz) for cells from multiple samples
- Cell-to-sample annotation (cellanno)
- Per-cell pseudotime values (e.g. NAM-PC2 pseudotime)
- Sample-level design matrix with intercept, covariate column (e.g. sex), and batch indicator columns
- Expression stored as hdf5 via Lamian saveh5() for large datasets

## Outputs

- Per-gene XDE test statistics including fdr.overall
- List of XDE genes at FDR.overall < 0.05 with cluster assignments (e.g. k=14 clusters)
- Population-level fitted expression (getPopulationFit type='variable') and covariate group differences (getCovariateGroupDiff)
- Heatmaps of original, fitted, and difference patterns (plotXDEHm) and cluster mean/difference plots (plotClusterMeanAndDiff)
- XCD result via cellPropTest: pval.overall and z.overall for covariate-associated cell-composition changes along pseudotime

## How to apply

Prepare a sample-level design matrix (intercept, covariate of interest, plus batch indicators; retain only batches with at least one sample per covariate group and samples with at least 1000 cells). For atlas-scale data such as the 337,191-cell TB memory T cell dataset (GSE158769), store expression in hdf5 via Lamian's saveh5() and run lamian_test_h5 with test.type='variable', testvar=2, test.method='permutation', permuiter=100 to test the covariate while batch columns are adjusted for. Select XDE genes at FDR.overall < 0.05, then compute population-level fits with getPopulationFit(type='variable'), covariate group differences with getCovariateGroupDiff, and cluster genes with clusterGene(type='variable', k=14); visualize with plotXDEHm and plotClusterMeanAndDiff. Optionally run cellPropTest(test.type='Variable', testvar=2) to obtain the XCD (cell-composition change) result with pval.overall and z.overall. Including batch indicators in the regression improves cross-split reproducibility of XDE gene calls.

## Related tools

- **Lamian** (Primary framework: lamian_test_h5 with test.type='variable' and permutation testing performs XDE detection with batch adjustment; saveh5, getPopulationFit, getCovariateGroupDiff, clusterGene, plotXDEHm, plotClusterMeanAndDiff, and cellPropTest support the workflow) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default method used by Lamian to construct pseudotime trajectories prior to XDE testing) — https://github.com/zji90/TSCAN

## Examples

```
res <- lamian_test_h5(expr.h5, cellanno, pseudotime, design, test.type='variable', testvar=2, test.method='permutation', permuiter=100)
```

## Evaluation signals

- XCD pval.overall for the covariate reaches reported significance (e.g. p = 8.9e-6 for sex in the TB analysis)
- Number of XDE genes at FDR.overall < 0.05 approximates the reported result (e.g. ~1120 sex-associated genes across 14 clusters in the TB dataset)
- Batch-adjusted XDE calls show higher cross-split reproducibility than unadjusted calls across repeated sample partitions (e.g. 12 splits, Fig. 6h)
- Sex-chromosome and XCI-escape gene enrichment is observed in the XDE gene list
- Permutation-based LRT completes with the specified permuiter (default 100) and returns per-gene fdr.overall values

## Limitations

- Existing methods limitations apply: condiments assumes one sample per condition and ignores sample-to-sample variation; Phenopath assumes linear expression changes along pseudotime and cannot model arbitrary non-linear condition differences
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories
- Simultaneous accounting of pseudotime inference uncertainty, gene expression and cell abundance uncertainty, and cross-sample/cross-cell variability remains future work
- A systematic comparison of sample-harmonization methods is beyond the scope of the framework

## Evidence

- [discussion] XDE detection using permutation-based likelihood ratio test comparing nested models: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] TSCAN is the default pseudotime method in Lamian: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [discussion] TB dataset batch filtering criterion: "Batches with at least one male sample and at least one female sample were retained."
- [abstract] Lamian accounts for cross-sample variability reducing false discoveries: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [abstract] Lamian adjusts for batch effects when identifying covariate-associated changes: "Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects, and to detect changes in"
