---
name: per-gene-algorithm-dispatch-thresholding
description: Use when fitting NBGMMs with the nebula R package on multi-subject single-cell
  counts and you must decide, per gene, whether NEBULA-LN estimates suffice or NEBULA-HL
  re-estimation (full or partial, fixing phi) is required.
when_to_use_negative:
- Data with fewer than 30 cells per subject across the board — every gene routes to
  NEBULA-HL regardless, so the threshold dispatch is trivially unnecessary
- Genes whose NEBULA-LN fit has not yet been obtained (no sigma2_hat/phi_hat/c_hat
  available) — run nebula first; the dispatch operates on fitted LN outputs, not raw
  counts alone
- Genes with convergence code <= -20 or estimated cell-level overdispersion > 100
  — these should be removed or interpreted cautiously rather than mechanically refit
  with NEBULA-HL
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nebula
  role: R package providing the nebula() fitting function and the method='HL' option
    used to implement the dispatch branches
  repo: https://github.com/lhe17/nebula
- name: NEBULA-LN
  role: Default fast algorithm (approximated likelihood, law of large numbers) whose
    per-gene estimates seed the dispatch decisions
  repo: https://github.com/lhe17/nebula
- name: NEBULA-HL
  role: H-likelihood refit engine invoked for full refits (CPS < 30 or CPS*phi_hat
    < 20) and partial refits (fix phi_hat, re-estimate sigma2)
  repo: https://github.com/lhe17/nebula
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:35f1750b871ebfbeda08ca4079bb8067023cf59ac72e4d538a11567b628a70c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA-HL
  - NEBULA-LN
  - R
  - nebula
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-gene-algorithm-dispatch-thresholding

## Summary

Routes each gene between NEBULA-LN (fast large-sample approximation) and NEBULA-HL (h-likelihood refit) using cells-per-subject (CPS) and estimated overdispersion thresholds, reserving the slower NEBULA-HL refit only for genes whose LN estimates are unreliable.

## When to use

Use when fitting NBGMMs with the nebula R package on multi-subject single-cell counts and you must decide, per gene, whether NEBULA-LN estimates suffice or NEBULA-HL re-estimation (full or partial, fixing phi) is required.

## When NOT to use

- Data with fewer than 30 cells per subject across the board — every gene routes to NEBULA-HL regardless, so the threshold dispatch is trivially unnecessary
- Genes whose NEBULA-LN fit has not yet been obtained (no sigma2_hat/phi_hat/c_hat available) — run nebula first; the dispatch operates on fitted LN outputs, not raw counts alone
- Genes with convergence code <= -20 or estimated cell-level overdispersion > 100 — these should be removed or interpreted cautiously rather than mechanically refit with NEBULA-HL

## Inputs

- Per-gene NEBULA-LN fitted estimates: subject-level sigma2_hat, cell-level phi_hat (overdispersion), and c_hat, from the upstream overdispersion decomposition
- Cells-per-subject (CPS) value for the analyzed cell type (e.g., CPS ~700 excitatory neurons, CPS ~380 oligodendrocytes in the frontal cortex snRNA-seq data)
- Predictor type (subject-level vs cell-level) determining kappa (800 vs 200)

## Outputs

- dispatch_table.csv with columns: gene ID, selected method (NEBULA-LN or NEBULA-HL), and dispatch branch (full-refit / partial-refit / LN-only)

## How to apply

Load per-gene NEBULA-LN fitted estimates (subject-level sigma2_hat, cell-level phi_hat, c_hat) from the upstream overdispersion-decomposition output, plus the cells-per-subject (CPS) for the analyzed cell type. First branch: if CPS*phi_hat < 20 or CPS < 30, refit both overdispersions with NEBULA-HL, consistent with nebula forcing method='HL' when CPS < 30. Second branch: if phi_hat < kappa*(1+c_hat^2)/CPS and sigma2_hat < 8*(1+c_hat^2)/(CPS*phi_hat), fix phi_hat and re-estimate only sigma2 with NEBULA-HL, using kappa=200 when testing cell-level predictors and kappa=800 (default) for subject-level predictors. All remaining genes keep their NEBULA-LN estimates, which the paper shows resolves ~90% of genes (CPC > 0.1%) at CPS ~700 (excitatory neurons) and ~80% at CPS ~380 (oligodendrocytes), avoiding the computationally expensive h-likelihood iteration for most of the genome.

## Related tools

- **nebula** (R package providing the nebula() fitting function and the method='HL' option used to implement the dispatch branches) — https://github.com/lhe17/nebula
- **NEBULA-LN** (Default fast algorithm (approximated likelihood, law of large numbers) whose per-gene estimates seed the dispatch decisions) — https://github.com/lhe17/nebula
- **NEBULA-HL** (H-likelihood refit engine invoked for full refits (CPS < 30 or CPS*phi_hat < 20) and partial refits (fix phi_hat, re-estimate sigma2)) — https://github.com/lhe17/nebula

## Examples

```
re = nebula(sample_data$count, sample_data$sid, pred=df, ncore=1)  # LN estimates seed per-gene dispatch to NEBULA-HL via method='HL' refits
```

## Evaluation signals

- Every gene in dispatch_table.csv receives exactly one method assignment (row count equals number of analyzed genes)
- All genes with CPS < 30 are routed to NEBULA-HL
- Fraction of genes resolved by NEBULA-LN alone matches paper rates: ~90% for excitatory neurons (CPS ~700) and ~80% for oligodendrocytes (CPS ~380) among genes with CPC > 0.1%
- kappa is set to 800 for subject-level predictors and 200 for cell-level predictors before evaluating the partial-refit branch
- Genes flagged with convergence code <= -20 are excluded or flagged for caution rather than passed through as resolved

## Limitations

- NEBULA-HL shows a downward bias in estimating the subject-level overdispersion, so refit estimates should be interpreted with that bias in mind
- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion, which is the motivation for selective HL re-estimation
- The dispatch inherits the sensitivity of subject-level variance-component estimation to the number of subjects and random-effects distribution assumptions
- Convergence code <= -20 indicates the algorithm did not converge well; results for such genes should be interpreted with caution even after refitting

## Evidence

- [methods] NEBULA-LN vs NEBULA-HL algorithm choice: "In *nebula*, a user can choose one of the two algorithms to fit an NBGMM. NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood."
- [results] Expected LN-only resolution rates: [evidence span withheld — anchor status: not_found]
- [readme] Convergence QC gating dispatch outputs: "Generally, a convergence code $\leq$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [methods] Overdispersion filtering thresholds adjacent to dispatch: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
