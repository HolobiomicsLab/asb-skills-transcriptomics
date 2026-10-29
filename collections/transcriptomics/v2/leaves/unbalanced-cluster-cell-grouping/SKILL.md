---
name: unbalanced-cluster-cell-grouping
description: Use when scRNA-seq/snRNA-seq counts for a cell-type subcluster have highly
  unbalanced cells per subject (e.g., AST3 cells from only ~4 of 30+ subjects) and
  naive per-cell differential or marker testing would inflate false positives.
when_to_use_negative:
- Cells come from a single subject with no subject-level grouping — a plain NB regression
  (e.g., glm.nb) suffices since no between-subject variance exists to model
- Input is already aggregated per subject (e.g., pseudobulk counts) — the hierarchical
  cell-level mixed model and group_cell ordering are not applicable
- The variable of interest is purely cell-level and cell numbers across subjects are
  balanced — simpler cell-level models without subject random effects may be adequate
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: nebula (R package v1.5.8)
  role: Fits the negative binomial gamma mixed model (NBGMM) with subject IDs to account
    for unbalanced cells-per-subject grouping; nebula function with id argument, group_cell
    to reorder cells by subject
  repo: https://github.com/lhe17/nebula
- name: R
  role: Analysis environment; nebula v1.5.7+ requires R-4.4.0+, Rcpp-1.0.7
- name: devtools
  role: Installs the nebula package from GitHub via install_github
  repo: https://github.com/lhe17/nebula
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:2ae65955bba8ff99934ef990622d5cbce0f653eaf5e35c8ef2c68a946b3d9f81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - R
  - devtools
  - nebula (R package v1.5.8)
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# unbalanced-cluster-cell-grouping

## Summary

A method for testing cell-type/subcluster marker associations in multi-subject single-cell data whose cells-per-subject group sizes are highly unbalanced, by fitting a negative binomial mixed model (NEBULA NBGMM) that decomposes overdispersion into subject-level and cell-level components instead of treating cells as independent.

## When to use

Use when scRNA-seq/snRNA-seq counts for a cell-type subcluster have highly unbalanced cells per subject (e.g., AST3 cells from only ~4 of 30+ subjects) and naive per-cell differential or marker testing would inflate false positives.

## When NOT to use

- Cells come from a single subject with no subject-level grouping — a plain NB regression (e.g., glm.nb) suffices since no between-subject variance exists to model
- Input is already aggregated per subject (e.g., pseudobulk counts) — the hierarchical cell-level mixed model and group_cell ordering are not applicable
- The variable of interest is purely cell-level and cell numbers across subjects are balanced — simpler cell-level models without subject random effects may be adequate

## Inputs

- Gene-by-cell integer count matrix (dense matrix or sparse dgCMatrix, as in Seurat)
- Per-cell subject ID vector (character or numeric, length equal to number of cells)
- Per-cell cluster/subcluster labels (e.g., AST0-AST3) and predictor variables
- Per-cell scaling factors/offsets (library-size scaling, vector of length N)

## Outputs

- Per-gene NBGMM summary statistics: logFC estimates, raw p-values, and both subject-level (sigma^2) and cell-level (phi^-1) overdispersion estimates
- Empirical false-positive rates vs. subject-level overdispersion (e.g., sigma^2 in {0, 0.02, 0.05, 0.1}) comparing naive NB regression vs. NEBULA
- Convergence codes per gene for quality control

## How to apply

First group the count matrix cells by subject IDs so cells from the same subject are consecutive, using the group_cell helper, and supply the per-subject grouping as the id argument to nebula. Build a design matrix with model.matrix (intercept required) over the cluster/subcluster labels and predictors, and pass per-cell scaling factors via the offset argument. Fit the NBGMM with nebula (choosing NEBULA-LN or NEBULA-HL) so the model separates subject-level variance sigma^2 from cell-level overdispersion phi^-1; this hierarchy is what controls FPR when cells-per-subject sizes are unbalanced, unlike a naive glm.nb fit that treats cells as independent. For null-simulation validation, generate counts under a null NB mixed model using the empirical cells-per-subject and cells-per-subcluster group sizes (e.g., sigma^2 in {0, 0.02, 0.05, 0.1} with cell-level overdispersion fixed at 1), analyze each simulated gene with both glm.nb and nebula, and compute the fraction of raw p-values below the nominal 0.05 threshold per condition.

## Related tools

- **nebula (R package v1.5.8)** (Fits the negative binomial gamma mixed model (NBGMM) with subject IDs to account for unbalanced cells-per-subject grouping; nebula function with id argument, group_cell to reorder cells by subject) — https://github.com/lhe17/nebula
- **R** (Analysis environment; nebula v1.5.7+ requires R-4.4.0+, Rcpp-1.0.7)
- **devtools** (Installs the nebula package from GitHub via install_github) — https://github.com/lhe17/nebula

## Examples

```
df = model.matrix(~cluster, data=meta); re = nebula(count, sid, pred=df, ncore=1)
```

## Evaluation signals

- Cells in the input count matrix are ordered so that cells of the same subject appear consecutively before calling nebula
- Under the null with increasing subject-level overdispersion (sigma^2 from 0 to 0.1), NEBULA's empirical FPR stays near the nominal 0.05 while naive glm.nb's FPR inflates rapidly
- Each gene's convergence code is checked; codes <= -20 indicate non-convergence and such results should be excluded or interpreted with caution
- Estimated subject-level sigma^2 is reported per gene and can be compared against the simulated truth (0, 0.02, 0.05, 0.1) in null simulations
- Group-size table (table(sid)) reproduces the empirical cells-per-subject distribution used to seed the simulation

## Limitations

- Testing a subject-level variable is highly sensitive to estimation of the subject-level variance component and to random-effects distribution assumptions when the number of subjects is small; misspecification of the random effects may inflate FPR
- If the variable of interest is subject-level, genes with subject-level overdispersion >1 should be removed or interpreted cautiously; genes with estimated cell-level overdispersion >100 should be removed for datasets with >=50 cells per subject
- Convergence codes <= -20 indicate the algorithm did not converge well and summary statistics should be interpreted with caution

## Evidence

- [intro] NEBULA reduced false positives in selecting cell-type marker genes, especially when the numbers of cells across subjects were unbalanced: "We showed that NEBULA reduced false positives in selecting cell-type marker genes, especially when the numbers of cells across subjects were unbalanced"
- [methods] grouping cells by subject before running nebula: "The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function"
- [readme] NEBULA decomposes overdispersion into between-subject and within-subject components: "The package *nebula* accounts for the hierarchical structure of the data by decomposing the total overdispersion into between-subject and within-subject components"
- [discussion] random-effects misspecification inflates FPR for subject-level predictors with few subjects: "misspecification of the distribution of the random effects may also lead to inflated FPR in testing a subject-level predictor"
- [methods] convergence code QC threshold: "Generally, a convergence code $\leq$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
