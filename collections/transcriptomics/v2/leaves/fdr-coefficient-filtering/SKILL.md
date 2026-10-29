---
name: fdr-coefficient-filtering
description: Use when a CPPLS regression model has produced a coefficient matrix W
  of neighboring-cell-type effects on HVGs and statistically significant communication
  edges must be retained before k-means clustering or MIMO graph construction.
when_to_use_negative:
- Before the CPPLS regression model has been fit — there are no factor loadings or
  coefficients to test yet
- For general differential expression analysis of genes rather than filtering regression
  coefficients linking neighboring cell types to HVG expression
- When the input is already a significant-edge list from the second filtering pass
  and the next step is HVG clustering or MIMO graph construction
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CPPLS-MLP
  role: Provides the CPPLS regression modeling and the two-step coefficient filtering
    implementation applied to spatial transcriptomics data
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: pls
  role: R package used to fit the constrained partial least squares (CPPLS) models
    whose factor loadings undergo the t-test/BH filtering
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R (version <= 4.2.3)
  role: Runtime environment in which the CPPLS-MLP filtering scripts (e.g., 2023_01_seqFISH+.RMD)
    are executed
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:41196a7dbaa6e138af9df68a5a9b7725fc25dcdc6f29bda8b310e81ed8b6d402
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - R
  - R (version <= 4.2.3)
  - R (version <=)
  - Seurat
  - circlize
  - pls
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# fdr-coefficient-filtering

## Summary

Two-step Benjamini-Hochberg FDR filtering of CPPLS regression coefficients (t-tests on factor loadings, delta = 0.05) to zero out non-significant neighboring-cell-type effects before downstream clustering of HVGs.

## When to use

Use when a CPPLS regression model has produced a coefficient matrix W of neighboring-cell-type effects on HVGs and statistically significant communication edges must be retained before k-means clustering or MIMO graph construction.

## When NOT to use

- Before the CPPLS regression model has been fit — there are no factor loadings or coefficients to test yet
- For general differential expression analysis of genes rather than filtering regression coefficients linking neighboring cell types to HVG expression
- When the input is already a significant-edge list from the second filtering pass and the next step is HVG clustering or MIMO graph construction

## Inputs

- Coefficient matrix W of neighboring-cell-type effects on HVGs from CPPLS regression (10-fold cross-validated)
- Factor loadings from each CPPLS block c
- HVG expression data from seqFISH+ mouse cortex or Seq-Scope mouse colon datasets (gene expression matrix U, coordinate matrix O, cell type label vector D)

## Outputs

- Filtered coefficient matrix with non-significant coefficients set to zero (delta = 0.05, BH-adjusted)
- List of communication-associated HVGs retained after removing zero-coefficient HVGs
- Cell-cell communication edge list (e.g., L5/L6 eNeuron edges for seqFISH+; immature B cell/IgA B cell-fibroblast edges for Seq-Scope)

## How to apply

After CPPLS regression modeling (10-fold cross-validation) yields coefficients w_b,h for each cell type block c, perform the first filtering pass: a t-test on the factor loadings from each block, with P-values adjusted by the Benjamini-Hochberg method at threshold delta = 0.05; non-significant coefficients are zeroed. Then run a second pass on genes that were not filtered out but still lack statistical significance, computing BH-adjusted P-values at the same delta = 0.05 and zeroing those coefficients. Rationale: raw regression coefficients confound neighboring-cell-type regulation with noise, and the two-pass scheme ensures only communication-associated HVG effects survive before clustering. Following filtering, HVGs with coefficients equal to 0 are dropped entirely so k-means (k from 2 to 15, chosen via Silhouette) operates only on significant signals.

## Related tools

- **CPPLS-MLP** (Provides the CPPLS regression modeling and the two-step coefficient filtering implementation applied to spatial transcriptomics data) — https://github.com/wuzhenao/CPPLS-MLP
- **pls** (R package used to fit the constrained partial least squares (CPPLS) models whose factor loadings undergo the t-test/BH filtering) — https://github.com/wuzhenao/CPPLS-MLP
- **R (version <= 4.2.3)** (Runtime environment in which the CPPLS-MLP filtering scripts (e.g., 2023_01_seqFISH+.RMD) are executed) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
Rscript 2023_01_seqFISH+.RMD  # runs CPPLS modeling and the two-pass BH (delta=0.05) coefficient filtering for the seqFISH+ network
```

## Evaluation signals

- Both filtering passes completed: first-pass t-tests on factor loadings and second-pass BH P-value test, each with Benjamini-Hochberg adjustment at delta = 0.05
- All HVGs with coefficients equal to 0 are removed before downstream clustering
- The retained edges still capture the reported communication structure, e.g., L5-L6 eNeuron edges in seqFISH+ and immature B cell/IgA B cell-fibroblast edges in Seq-Scope
- The output network file exists, is non-empty, and CPPLS-MLP identifies the expected ~354 (seqFISH+) and ~345 (Seq-Scope) communication-related genes
- Downstream k-means clustering with Silhouette-selected k (2-15) produces renderable clusters, indicating the filtered matrix is not degenerate

## Limitations

- Only two benchmark datasets (seqFISH+ and Seq-Scope) were used for performance assessment, so generalization of the delta = 0.05 threshold is untested
- The filtering is a t-test-based statistical step; the paper gives no standardized classification criterion beyond this, noting that 'these methods lack a standardized classification criterion to determine whether a gene is associated with cell communication'
- Genes filtered in the second pass are zeroed, which can over-prune the coefficient matrix and affect downstream Silhouette-based k selection

## Evidence

- [methods] First filtering pass uses t-test on factor loadings: [evidence span withheld — anchor status: not_found]
- [methods] Second filtering pass on genes lacking significance: [evidence span withheld — anchor status: not_found]
- [methods] Zero-coefficient HVGs removed before clustering: "Before clustering, all the HVGs with coefficients equal to 0 are filtered out."
- [results] FDR threshold delta = 0.05 and two passes: [evidence span withheld — anchor status: not_found]
