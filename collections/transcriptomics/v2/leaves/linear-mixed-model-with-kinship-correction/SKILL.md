---
name: linear-mixed-model-with-kinship-correction
description: Use when you have per-sample cell-type proportion estimates (e.g., Bisque
  bulk.props) for a bulk RNA-seq cohort plus measured phenotypes and sample relatedness,
  and you need to test cell-proportion/phenotype associations while correcting for
  kinship, age, and sex.
when_to_use_negative:
- No measured phenotypes or covariates (age, sex, kinship) are available for the bulk
  samples, so the association model cannot be fit
- BisqueMarker relative abundances are used — these capture relative differences in
  abundance that cannot be compared across cell types, complicating cross-cell-type
  association interpretation
- Few individuals have single-cell data, making the reference profile and gene-specific
  transformations unreliable and the resulting proportions untrustworthy for association
  testing
tools:
- name: BisqueRNA
  role: Produces the bulk.props cell-type proportion estimates that serve as the response
    variables in the linear-mixed association models
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet format required by Bisque for bulk and single-cell
    inputs prior to decomposition
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:eaf48584d311fa1d0873ea1b7fd541c0a6f8199663ab5e8d5cd64970f37c445d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# linear-mixed-model-with-kinship-correction

## Summary

Fit a linear-mixed model of each Bisque-estimated adipose cell-type proportion (adipocyte, macrophage, T-cell) against measured phenotypes (BMI, Matsuda index), including age, sex, and relatedness (kinship) as covariates, to test whether decomposition estimates replicate previously reported cell-composition phenotype associations.

## When to use

Use when you have per-sample cell-type proportion estimates (e.g., Bisque bulk.props) for a bulk RNA-seq cohort plus measured phenotypes and sample relatedness, and you need to test cell-proportion/phenotype associations while correcting for kinship, age, and sex.

## When NOT to use

- No measured phenotypes or covariates (age, sex, kinship) are available for the bulk samples, so the association model cannot be fit
- BisqueMarker relative abundances are used — these capture relative differences in abundance that cannot be compared across cell types, complicating cross-cell-type association interpretation
- Few individuals have single-cell data, making the reference profile and gene-specific transformations unreliable and the resulting proportions untrustworthy for association testing

## Inputs

- Bisque bulk.props cell-type proportion matrix for 100 adipose bulk RNA-seq samples
- Per-sample phenotype table with BMI and Matsuda index
- Per-sample covariate table with age, sex, and kinship/relatedness structure

## Outputs

- Per-cell-type, per-phenotype association p-values (e.g., adipocyte–BMI, macrophage–BMI, T-cell–Matsuda)
- adipose_association_pvalues.csv with p-values from reference-based and marker-based decompositions

## How to apply

Decompose the adipose bulk samples with BisqueRNA::ReferenceBasedDecomposition (use.overlap=TRUE on the overlapping single-cell/bulk subjects) or MarkerBasedDecomposition, and extract proportions from the bulk.props slot. For each cell type (adipocyte, macrophage, T-cell), fit a linear-mixed model of the proportion against BMI and Matsuda index with age, sex, and relatedness (kinship) as covariates, restricted to the 100 remaining bulk samples without matched snRNA-seq. Compile the association p-values per cell type and phenotype (target values: adipocyte–BMI p = 0.030, macrophage–BMI p < 0.001, T-cell–Matsuda p = 0.002) and write them to adipose_association_pvalues.csv. Correctness is judged by replication of the expected direction and significance of prior associations, notably a significant negative adipocyte-proportion vs BMI relationship.

## Related tools

- **BisqueRNA** (Produces the bulk.props cell-type proportion estimates that serve as the response variables in the linear-mixed association models) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet format required by Bisque for bulk and single-cell inputs prior to decomposition)

## Evaluation signals

- Adipocyte proportion shows a significant negative association with BMI at approximately p = 0.030, matching the expected negative relationship reported in prior work
- Macrophage–BMI and T-cell–Matsuda p-values reproduce the expected targets (p < 0.001 and p = 0.002, respectively)
- Model includes age, sex, and kinship as covariates and is restricted to the 100 non-overlapping bulk samples
- Output file adipose_association_pvalues.csv contains p-values from both ReferenceBasedDecomposition and MarkerBasedDecomposition

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption of the transformation framework is that single-cell-based estimates of cell proportions accurately reflect the true proportions
- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types
- Accuracy may decrease if the proportion of cell types captured by single-cell experiments differs significantly from the true physiological distributions

## Evidence

- [abstract] Bisque replicates previously reported associations between cell type proportions and measured phenotypes across abundant and rare cell types.: "Bisque replicates previously reported associations between cell type proportions and measured phenotypes across abundant and rare cell types."
- [discussion] Single-cell subsetting of bulk cohorts powers cell-specific association detection: "single-cell sequencing of a subset of samples from large-scale bulk expression cohorts can provide high power to detect cell-specific associations in complex phenotypes"
- [methods] Reference-based decomposition call used to generate proportions: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] Marker-based decomposition alternative when reference is unavailable: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [readme] README description of bulk.props output used as model input: [evidence span withheld — anchor status: external_doc]
