---
name: sensitivity-threshold-analysis
description: Use when sweeping a parameter (e.g. proportion of control genes influenced
  by a latent effect) to find the minimal proportion at which ccRemover's bootstrap
  Δj test reliably detects and removes that effect.
when_to_use_negative:
- The analysis goal is removing non-cell-cycle confounders — ccRemover 'is designed
  for removing the cell-cycle effect exclusively' and will not remove other dominant
  effects
- Input is real data with very low cell-cycle activity, e.g. the Patel et al. glioblastoma
  data, where the article switches to a smaller cutoff CΔ = 2 rather than the default
  3
- No control-gene set can be defined (all genes are annotated to the cell cycle),
  since the PCA on control genes XB that underpins the test cannot be performed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ccRemover
  role: R package that performs PCA on control genes, projects onto cell-cycle genes,
    and applies bootstrap significance testing (cutoff CΔ, nboot, ntop) to decide
    which components are cell-cycle effects
  repo: cran/ccRemover
- name: org.Hs.eg.db
  role: Provides human gene-to-cell-cycle annotations used to split genes into control
    genes XB and cell-cycle genes XA
- name: org.Mm.eg.db
  role: Provides mouse gene-to-cell-cycle annotations used to define control vs cell-cycle
    gene sets in simulated mouse-style data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:ce23ed05a410fc06a04a939773c01ece16671fe5c28a694cb9f5578e1ec640be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - ccRemover
  - org.Hs.eg.db
  - org.Mm.eg.db
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sensitivity-threshold-analysis

## Summary

Reproduces the Supplementary Section S2 sensitivity analysis of Barron & Li (2016) that determines how much of the cell-cycle effect must influence control genes before ccRemover can detect and remove it, establishing the ~8% robustness threshold.

## When to use

Use when sweeping a parameter (e.g. proportion of control genes influenced by a latent effect) to find the minimal proportion at which ccRemover's bootstrap Δj test reliably detects and removes that effect.

## When NOT to use

- The analysis goal is removing non-cell-cycle confounders — ccRemover 'is designed for removing the cell-cycle effect exclusively' and will not remove other dominant effects
- Input is real data with very low cell-cycle activity, e.g. the Patel et al. glioblastoma data, where the article switches to a smaller cutoff CΔ = 2 rather than the default 3
- No control-gene set can be defined (all genes are annotated to the cell cycle), since the PCA on control genes XB that underpins the test cannot be performed

## Inputs

- Simulated scRNA-Seq expression matrix X (50 cells × 2,000 genes, 400 annotated as cell-cycle genes)
- List of cell-cycle gene annotations (e.g. via Biomart / org.Mm.eg.db / org.Hs.eg.db)
- Sweep parameter: proportion of control genes influenced by the cell-cycle effect (e.g. 1%–20%)
- ccRemover parameters: cutoff CΔ = 3, nboot (100 per source card; 200 in the article's demo call), max_it = 4, ntop = 10

## Outputs

- Detection/removal success table: significant Δj outcome per proportion of affected control genes
- Minimal proportion of affected control genes at which ccRemover consistently captures the cell-cycle effect (reported as ≥ ~8%)
- Corrected data matrices Xhat for each sweep point

## How to apply

Generate simulated scRNA-Seq matrices following the modified Supplementary Section S2 model using the baseline design of 50 cells, 2,000 genes, and 400 cell-cycle genes, while varying the proportion of control genes influenced by the cell-cycle effect. For each simulated dataset, run the ccRemover algorithm — PCA on control genes XB, projection onto cell-cycle genes XA, then bootstrap significance testing — recording whether each iteration yields a significant Δj > 0. Per the source card, use cutoff CΔ = 3 with nboot = 100; the article's demonstration call uses nboot = 200, max_it = 4, ntop = 10. Tabulate detection/removal success against the proportion of affected control genes and identify the minimal proportion at which the leading principal components capture the effect. The rationale is that the cell-cycle signal must influence enough control genes to be captured by the principal components before the formal statistical test can flag it.

## Related tools

- **ccRemover** (R package that performs PCA on control genes, projects onto cell-cycle genes, and applies bootstrap significance testing (cutoff CΔ, nboot, ntop) to decide which components are cell-cycle effects) — cran/ccRemover
- **org.Hs.eg.db** (Provides human gene-to-cell-cycle annotations used to split genes into control genes XB and cell-cycle genes XA)
- **org.Mm.eg.db** (Provides mouse gene-to-cell-cycle annotations used to define control vs cell-cycle gene sets in simulated mouse-style data)

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Below ~8% of control genes affected, the sweep shows ccRemover failing to report significant Δj > 0 because the effect is not captured by the leading principal components
- At or above ~8% of control genes affected, ccRemover consistently identifies and removes the cell-cycle component across simulated replicates
- The baseline simulation matches the reported design exactly: 50 cells, 2,000 genes, of which 400 are cell-cycle genes
- After removal at suprathreshold proportions, cells cluster by cell type with cell-cycle stages mixed (as in the article's original simulation validation)
- Bootstrap settings (nboot, cutoff CΔ = 3, max_it, ntop) are recorded and held fixed across the sweep so detection differences are attributable to the sparsity parameter alone

## Limitations

- ccRemover may miss the cell-cycle effect if it influences too few control genes (below ~8% affected), since the effect is not captured by the leading principal components
- Annotation databases are always incomplete and inaccurate, so the control-gene / cell-cycle-gene split — the very variable swept here — depends on annotation quality
- No version numbers are reported for ccRemover or the R annotation packages, and the Supplementary Section S2 details are referenced but not included in the provided main text
- The reported ~8% threshold is specific to the S2 simulation design (50 cells, 2,000 genes, 400 cell-cycle genes) and may shift with different simulation dimensions

## Evidence

- [results] Simulation design baseline: "We simulate data matrix X that contains measurements for 50 cells and 2,000 genes, of these genes 400 are assigned as cell-cycle genes"
- [discussion] Detection mechanism: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
- [methods] Algorithm entry point: "The ccRemover algorithm follows these steps: 1. Perform a PCA on the data matrix of control genes XB."
- [methods] Reference invocation with cutoff: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
