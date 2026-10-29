---
name: single-cell-overdispersion-decomposition
description: Use when fitting per-gene mixed models to multi-subject scRNA-seq/snRNA-seq
  counts and you need to separate between-subject from within-subject overdispersion,
  or test whether covariates (e.g., age, sex, AD status) explain subject-level variation.
when_to_use_negative:
- The data are bulk RNA-seq or have no multi-subject (multi-donor) structure — the
  subject-level/cell-level overdispersion decomposition requires hierarchical multi-subject
  sampling.
- You only need differential expression p-values, not the variance-component estimates
  (σ², ϕ⁻¹); standard DE tools may suffice.
- Genes fail QC outright (convergence code ≤ -20, cell-level overdispersion >100,
  or subject-level overdispersion >1 when testing subject-level variables) — these
  estimates are unreliable and should be excluded rather than interpreted.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: nebula (R package v1.5.8)
  role: Fits the NBGMM per gene via nebula(), reorders cells by subject with group_cell(),
    reports σ² and ϕ⁻¹ and convergence codes
  repo: https://github.com/lhe17/nebula
- name: NEBULA-HL
  role: H-likelihood algorithm automatically invoked by nebula when average counts
    per subject are <3; used for lowly expressed genes
  repo: https://github.com/lhe17/nebula
- name: NEBULA-LN
  role: Large-sample-approximation algorithm (default) for fitting the NBGMM per gene
  repo: https://github.com/lhe17/nebula
- name: R (R-4.4.0)
  role: Runtime environment; nebula v1.5.7+ requires R-4.4.0+
- name: devtools
  role: Used to install the nebula package from GitHub via install_github("lhe17/nebula")
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:a5e8ea2aca14452c76a917e4caf4e965313c3464ced0aec072d5c257898297b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA
  - NEBULA-HL
  - NEBULA-LN
  - R (R-4.4.0)
  - devtools
  - nebula (R package v1.5.8)
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-overdispersion-decomposition

## Summary

Decomposes gene-specific overdispersion in multi-subject single-cell data into subject-level (σ²) and cell-level (ϕ⁻¹) components using NEBULA's negative binomial gamma mixed model (NBGMM), then quantifies how much covariate adjustment explains each component. Applied here to ROSMAP frontal cortex snRNA-seq across six major brain cell types.

## When to use

Use when fitting per-gene mixed models to multi-subject scRNA-seq/snRNA-seq counts and you need to separate between-subject from within-subject overdispersion, or test whether covariates (e.g., age, sex, AD status) explain subject-level variation.

## When NOT to use

- The data are bulk RNA-seq or have no multi-subject (multi-donor) structure — the subject-level/cell-level overdispersion decomposition requires hierarchical multi-subject sampling.
- You only need differential expression p-values, not the variance-component estimates (σ², ϕ⁻¹); standard DE tools may suffice.
- Genes fail QC outright (convergence code ≤ -20, cell-level overdispersion >100, or subject-level overdispersion >1 when testing subject-level variables) — these estimates are unreliable and should be excluded rather than interpreted.

## Inputs

- ROSMAP frontal cortex snRNA-seq count matrix (sparse dgCMatrix, genes x cells) from Synapse syn21261143
- Cell metadata with major cell type labels and per-cell library sizes (offset)
- Subject-level covariate table (age, sex, race, AD status) from Synapse syn3157322
- Cell-level covariates (total features, ribosomal-gene percentage)

## Outputs

- overdispersion_by_celltype.csv with per-gene sigma^2 and phi^-1 for unadjusted and adjusted fits across six cell types
- Per-gene convergence codes retained for QC

## How to apply

Download the count matrices and cell metadata (ROSMAP syn21261143) and subject-level covariates (syn3157322), subset to one major cell type, and reorder cells so each subject's cells are consecutive with nebula::group_cell. Build the design matrix with model.matrix (intercept required), first without covariates and then with age, sex, race, AD status, total features, and ribosomal-gene percentage; pass the per-cell library size as the offset argument. Filter low-expression genes with cpc=0.005 and fit the NBGMM per gene with nebula, which automatically invokes NEBULA-HL when average counts per subject are <3; record σ², ϕ⁻¹, and convergence codes. Repeat for all six cell types and both designs, exclude genes with convergence code ≤ -20, flag cell-level overdispersion >100 and subject-level overdispersion >1, and assemble estimates into overdispersion_by_celltype.csv. Rationale: the hierarchical NBMM separates overdispersion into subject and cell components, letting you attribute reductions in σ² after adjustment to subject-level covariates while cell-level ϕ⁻¹ reflects subpopulation heterogeneity.

## Related tools

- **nebula (R package v1.5.8)** (Fits the NBGMM per gene via nebula(), reorders cells by subject with group_cell(), reports σ² and ϕ⁻¹ and convergence codes) — https://github.com/lhe17/nebula
- **NEBULA-HL** (H-likelihood algorithm automatically invoked by nebula when average counts per subject are <3; used for lowly expressed genes) — https://github.com/lhe17/nebula
- **NEBULA-LN** (Large-sample-approximation algorithm (default) for fitting the NBGMM per gene) — https://github.com/lhe17/nebula
- **R (R-4.4.0)** (Runtime environment; nebula v1.5.7+ requires R-4.4.0+)
- **devtools** (Used to install the nebula package from GitHub via install_github("lhe17/nebula"))

## Examples

```
re = nebula(sample_data$count, sample_data$sid, pred = model.matrix(~X1+X2+cc, data=sample_data$pred), cpc=0.005, offset=lib_sizes, ncore=1)
```

## Evaluation signals

- Every retained gene in overdispersion_by_celltype.csv has a reported convergence code, with codes ≤ -20 excluded
- The output table contains both sigma^2 and phi^-1 columns for the unadjusted and adjusted fits of each of the six cell types
- Adjusted models show substantially reduced subject-level overdispersion (σ²) in most cell types relative to unadjusted fits
- Cell-level overdispersion (ϕ⁻¹) remains almost unchanged between unadjusted and adjusted models
- Genes flagged as cell-level overdispersion >100 or subject-level overdispersion >1 are marked for cautious interpretation or removal

## Limitations

- NEBULA-HL shows downward bias in estimating the subject-level overdispersion, consistent with theoretical results
- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared to a standard h-likelihood method
- Genes with cell-level overdispersion >100 should be removed (≥50 cells per subject) and genes with subject-level overdispersion >1 interpreted cautiously for subject-level variables
- A convergence code ≤ -20 indicates the algorithm does not converge well; results should be interpreted with caution
- Misspecification of the random-effects distribution may inflate false positives when testing subject-level predictors with few subjects

## Evidence

- [methods] Filtering out low-expressed genes can be specified by cpc=0.005 (i.e., counts per cell<0.5%). The argument cpc is defined by the ratio between the total count of the gene and the number of cells.: "Filtering out low-expressed genes can be specified by cpc=0.005 (i.e., counts per cell<0.5%). The argument cpc is defined by the ratio between the total count of the gene and the number of cells."
- [methods] convergence code threshold: "Generally, a convergence code $\leq$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [methods] overdispersion filter thresholds: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
- [methods] grouping cells before nebula: "The cells in the count matrix need to be grouped by the subjects (that is, the cells of the same subject should be placed consecutively) before using as the input to the function"
- [discussion] NEBULA-HL bias: "The downward bias of NEBULA-HL in estimating the subject-level overdispersion is in line with theoretical results"
- [readme] package description: "The *nebula* package accounts for the hierarchical structure of the data by decomposing the total overdispersion into between-subject and within-subject components using a negative binomial mixed"
- [readme] algorithm choice: "NEBULA-LN uses an approximated likelihood based on the law of large numbers, and NEBULA-HL uses an h-likelihood."
