---
name: bootstrap-significance-testing
description: Use when you must decide which principal components of a log-transformed,
  gene-centered scRNA-Seq matrix represent the cell-cycle effect, by testing loading
  differences between cell-cycle genes (XA) and control genes (XB) via resampling.
when_to_use_negative:
- The cell-cycle effect influences only a few control genes (below ~8% affected),
  where it may not be captured by the leading principal components and the test will
  miss it
- The goal is to remove non-cell-cycle confounders such as mitochondrial burden or
  batch effects — ccRemover is designed for removing the cell-cycle effect exclusively
- Cell-cycle activity is known to be very low (e.g. the glioblastoma dataset), where
  the default cutoff should be lowered (CΔ = 2) rather than the test relied on at
  CΔ = 3
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: R package that wraps the two-class bootstrap test on Δj to flag significant
    cell-cycle components and subtract them
  repo: cran/ccRemover
- name: bootstrap_diff
  role: Generates bootstrap replicates of Δj by sampling with replacement from XA
    and XB
- name: get_diff
  role: Computes the difference in loading strength Δj between cell-cycle and control
    genes feeding the bootstrap t-statistic
- name: stats
  role: Supplies the t-statistic computation used against cutoff CΔ
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:ac21e1deb1612375884cb7a38de979f7587165bd2264ea91fa4062483e2541d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Python
  - bootstrap_diff
  - ccRemover
  - get_diff
  - org.Mm.eg.db
  - stats
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bootstrap-significance-testing

## Summary

A two-class bootstrap t-test, embedded in ccRemover, that tests whether cell-cycle-annotated genes load more strongly on a principal component than control genes, declaring components with t > CΔ as cell-cycle effects.

## When to use

Use when you must decide which leading principal components of a log-transformed, gene-centered scRNA-Seq matrix represent the cell-cycle effect, by testing loading differences between cell-cycle genes (XA) and control genes (XB) via resampling.

## When NOT to use

- The cell-cycle effect influences only a few control genes (below ~8% affected), where it may not be captured by the leading principal components and the test will miss it
- The goal is to remove non-cell-cycle confounders such as mitochondrial burden or batch effects — ccRemover is designed for removing the cell-cycle effect exclusively
- Cell-cycle activity is known to be very low (e.g. the glioblastoma dataset), where the default cutoff should be lowered (CΔ = 2) rather than the test relied on at CΔ = 3

## Inputs

- Normalized, log-transformed, gene-by-gene centered expression matrix X (genes × cells)
- if_cc indicator vector splitting genes into cell-cycle set A (XA) and control set B (XB)
- Bootstrap-test parameters: cutoff CΔ (default 3), nboot (100–200), ntop (10), max_it (4)

## Outputs

- Bootstrap replicates of Δj and bootstrap-based t-statistics per principal component
- Set of significant components declared as cell-cycle effects
- Corrected cell-cycle-free matrix xhat after subtracting projections onto significant directions

## How to apply

After PCA on the control-gene matrix XB (ntop = 10 components tested per iteration), project the cell-cycle gene matrix XA onto each loading vj and compute the difference in loading strength Δj between cell-cycle and control genes. Take random samples with replacement of |A| rows from XA and |B| rows from XB to build bootstrap replicates of Δj (nboot = 200 for real data, 100 in simulations), then compute the bootstrap-based t-statistic and declare components significant when t exceeds the cutoff CΔ = 3, roughly p = 0.01. Project the full matrix X onto significant directions, subtract those projections, and repeat until no significant component is found or max_it = 4. This formal test is what distinguishes ccRemover from scLVM, which removes leading factors without inspecting whether they represent the cell cycle.

## Related tools

- **ccRemover** (R package that wraps the two-class bootstrap test on Δj to flag significant cell-cycle components and subtract them) — cran/ccRemover
- **bootstrap_diff** (Generates bootstrap replicates of Δj by sampling with replacement from XA and XB)
- **get_diff** (Computes the difference in loading strength Δj between cell-cycle and control genes feeding the bootstrap t-statistic)
- **stats** (Supplies the t-statistic computation used against cutoff CΔ)

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- On the T-helper dataset (81 cells × 14,147 genes), PC1 must be flagged as a cell-cycle effect on iteration 1 with t.load.boot = 7.13, and no further effects found on iteration 2
- On the 50-cell, 2,000-gene simulation with 400 cell-cycle genes, the test must identify the 2nd and 3rd principal components as significant
- All declared components must satisfy t > CΔ = 3 (roughly p = 0.01); components with t ≤ 3 must be retained
- The loop must terminate within max_it = 4 iterations, returning xhat with stored gene means re-added
- After removal, downstream clustering should preserve cell-type separation with cell-cycle stages mixed within clusters (odds ratio 3.439 for TH2 signature vs 2.382 for scLVM)

## Limitations

- Annotation databases are always incomplete and inaccurate, so the if_cc gene set driving the test may be imperfect
- If the cell-cycle effect influences fewer than ~8% of control genes it may not be captured by the principal components and will be missed for detection
- The test removes only the cell-cycle effect; other confounding effects that may dominate after removal are left in place
- Very low cell-cycle activity (glioblastoma data) requires a smaller cutoff CΔ = 2; no version numbers for ccRemover or annotation packages are documented

## Evidence

- [methods] Take a random sample with replacement of |A| rows from XA and another |B| rows from XB: [evidence span withheld — anchor status: not_found]
- [methods] ccRemover call with parameters: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [results] Correct component identification on simulation: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [discussion] Sparsity caveat on control genes: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
