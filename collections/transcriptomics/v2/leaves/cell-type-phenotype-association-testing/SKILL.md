---
name: cell-type-phenotype-association-testing
description: Use when cell-type proportions have been estimated from bulk RNA-seq
  (via BisqueRNA bulk.props or BisqueMarker) and you need to test associations with
  phenotypes such as BMI, Matsuda index, or Braak stage, replicating reported cell-composition
  associations.
when_to_use_negative:
- Comparing absolute abundance magnitudes across cell types from BisqueMarker output,
  since marker-based estimates are relative abundances that cannot be compared between
  cell types
- Decomposition itself is needed — this skill assumes bulk.props estimates already
  exist
- Reference profile or gene-specific transformations are unreliable because too few
  individuals have single-cell data (a stated Bisque limitation)
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BisqueRNA
  role: Provides ReferenceBasedDecomposition and MarkerBasedDecomposition whose bulk.props
    slot supplies the per-sample cell-type proportions fed into the phenotype association
    regressions
  repo: github.com/cozygene/bisque
- name: Bisque
  role: Reference-based decomposition mode whose proportion estimates replicate reported
    cell-type–phenotype associations in adipose and DLPFC datasets
  repo: github.com/cozygene/bisque
- name: BisqueMarker
  role: Semi-supervised marker-based mode used to confirm the same phenotype associations
    (e.g. adipocyte-BMI, macrophage-BMI, T-cell-Matsuda) from marker genes alone
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet format required by BisqueRNA for bulk and single-cell
    inputs to decomposition
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:7a47afe128de0d3e9e8489d653d900a64ae70e59229e2a9fefa4844c9c5b88e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-type-phenotype-association-testing

## Summary

Tests whether cell-type proportions estimated by Bisque (reference-based) or BisqueMarker (marker-based) decomposition of bulk RNA-seq are significantly associated with measured phenotypes, via linear (mixed) regression with demographic covariates. It is the validation layer that turns decomposition outputs into replicable biological findings, e.g. adipocyte-BMI, macrophage-BMI, T-cell-Matsuda, and microglia-Braak associations.

## When to use

Use when cell-type proportions have been estimated from bulk RNA-seq (via BisqueRNA bulk.props or BisqueMarker) and you need to test associations with phenotypes such as BMI, Matsuda index, or Braak stage, replicating reported cell-composition associations.

## When NOT to use

- Comparing absolute abundance magnitudes across cell types from BisqueMarker output, since marker-based estimates are relative abundances that cannot be compared between cell types
- Decomposition itself is needed — this skill assumes bulk.props estimates already exist
- Reference profile or gene-specific transformations are unreliable because too few individuals have single-cell data (a stated Bisque limitation)

## Inputs

- Cell-type proportion estimates from BisqueRNA ReferenceBasedDecomposition (res$bulk.props) for bulk samples (e.g. 100 adipose or 628 DLPFC bulk RNA-seq samples)
- Relative abundance estimates from BisqueRNA MarkerBasedDecomposition (res$bulk.props) when no single-cell reference is available
- Sample phenotype table (BMI, Matsuda index, Braak stage, cognitive diagnosis)
- Covariates per sample: age, sex, age-squared, and kinship/relatedness matrix where applicable

## Outputs

- Per-cell-type regression coefficients and p-values for each phenotype and each decomposition method
- Comparison table of association p-values across methods (e.g. adipose_association_pvalues.csv, bisque_marker_associations.csv)

## How to apply

First obtain per-sample cell-type proportion estimates by running BisqueRNA::ReferenceBasedDecomposition (extract res$bulk.props) or BisqueRNA::MarkerBasedDecomposition for marker-based relative abundances. For each cell type and method, fit a linear regression (or linear-mixed model when relatedness/kinship is available) of the estimated proportion on the phenotype of interest, adjusting for covariates such as age, sex, and age-squared (DLPFC) or age, sex, and kinship (adipose). Compile coefficients and p-values from both decomposition modes into a comparison table (e.g. adipose_association_pvalues.csv, bisque_marker_associations.csv). Validate replication qualitatively: the same associations significant under reference-based Bisque should reproduce under the tested mode — e.g. adipocyte-BMI p = 0.030, macrophage-BMI p < 0.001, T-cell-Matsuda p = 0.002 in adipose, and Bisque microglia-Braak p = 0.001 (positive, covariate-corrected) in the 628 DLPFC bulk samples. Note that BisqueMarker estimates are relative within each cell type, so cross-cell-type comparisons of its estimates are not meaningful and only within-cell-type association patterns should be compared.

## Related tools

- **BisqueRNA** (Provides ReferenceBasedDecomposition and MarkerBasedDecomposition whose bulk.props slot supplies the per-sample cell-type proportions fed into the phenotype association regressions) — github.com/cozygene/bisque
- **Bisque** (Reference-based decomposition mode whose proportion estimates replicate reported cell-type–phenotype associations in adipose and DLPFC datasets) — github.com/cozygene/bisque
- **BisqueMarker** (Semi-supervised marker-based mode used to confirm the same phenotype associations (e.g. adipocyte-BMI, macrophage-BMI, T-cell-Matsuda) from marker genes alone) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet format required by BisqueRNA for bulk and single-cell inputs to decomposition)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); props <- t(res$bulk.props); fit <- lm(props[,"microglia"] ~ Braak + age + I(age^2) + sex)
```

## Evaluation signals

- Adipose replication: adipocyte-BMI p = 0.030, macrophage-BMI p < 0.001, and T-cell-Matsuda p = 0.002 are significant under reference-based Bisque on the 100 remaining bulk samples
- DLPFC replication: Bisque microglia proportion shows a significant positive association with Braak stage (p = 0.001) after correcting for sex, age, and age-squared, while MuSiC follows the trend without reaching significance
- BisqueMarker qualitative check: the same three adipose phenotype associations (adipocyte-BMI, macrophage-BMI, T-cell-Matsuda) are significant under marker-based mode as under reference-based Bisque (per Supplementary Table 1)
- BisqueMarker estimates reproduce within-cell-type association patterns but not cross-cell-type associations, consistent with their relative-abundance nature
- Regression tables include coefficients and p-values for every cell type and both methods, with covariates (age, sex, age-squared and/or kinship) specified

## Limitations

- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types, so only within-cell-type association trends are interpretable
- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable, degrading the proportion estimates being tested
- Bisque provides estimates of cell proportions reported by the single-cell technology used to generate the reference data; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions

## Evidence

- [abstract] Applied to subcutaneous adipose and dorsolateral prefrontal cortex datasets, Bisque replicates previously reported associations between cell type proportions and measured phenotypes across abundant and rare cell types: [evidence span withheld — anchor status: not_found]
- [methods] reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] marker-based decomposition call: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [discussion] BisqueMarker relative abundance limitation: "it should be noted that this method estimates relative differences in abundances that cannot be compared across cell types."
- [readme] README on marker-based mode interpretation: [evidence span withheld — anchor status: external_doc]
