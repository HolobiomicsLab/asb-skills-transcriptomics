---
name: permutation-based-likelihood-ratio-testing
description: Use when multi-sample scRNA-seq data with pseudotime and a design matrix
  require testing whether gene expression dynamics differ by a sample covariate (e.g.,
  sex, disease severity) with controlled FDR.
when_to_use_negative:
- Data are single-sample or per-sample pseudotime analysis without sample replicates,
  where Lamian's cross-sample variability model does not apply (as with condiments'
  one-sample-per-condition assumption)
- Goal is testing topological/trajectory branch differences (differential topology
  via binomial logistic regression) or cell density changes rather than gene expression
  along pseudotime
- Trajectories were inferred with Monocle2, Monocle3, or Slingshot for Lamian modules
  1 and 2, which do not support them (though Slingshot pseudotime feeds the module
  3-4 tests shown here)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: Implements the permutation-based likelihood ratio test for XDE (test.type='variable',
    testvar=2) and TDE (test.type='time') via lamian_test()
  repo: https://github.com/Winnie09/Lamian
- name: Slingshot
  role: One of the pseudotime inference methods whose pseudotime values serve as input
    to the Lamian tests
- name: TSCAN
  role: Default cluster-based minimum spanning tree method used by Lamian to construct
    the pseudotime input for testing
  repo: zji90/TSCAN (installed via devtools::install_github("zji90/TSCAN"))
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:28e12611cb7ff58a15809cf25ac4b79f61153bbbbd30c1ef1b6de60bb8651097
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Slingshot
  - TSCAN
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# permutation-based-likelihood-ratio-testing

## Summary

Lamian's hypothesis-testing core: a permutation-based likelihood ratio test comparing nested models to detect sample-covariate-associated changes in gene expression along pseudotime (XDE) and time-dependent expression (TDE), while accounting for cross-sample variability.

## When to use

Use when multi-sample scRNA-seq data with pseudotime and a design matrix require testing whether gene expression dynamics differ by a sample covariate (e.g., sex, disease severity) with controlled FDR.

## When NOT to use

- Data are single-sample or per-sample pseudotime analysis without sample replicates, where Lamian's cross-sample variability model does not apply (as with condiments' one-sample-per-condition assumption)
- Goal is testing topological/trajectory branch differences (differential topology via binomial logistic regression) or cell density changes rather than gene expression along pseudotime
- Trajectories were inferred with Monocle2, Monocle3, or Slingshot for Lamian modules 1 and 2, which do not support them (though Slingshot pseudotime feeds the module 3-4 tests shown here)

## Inputs

- Gene expression count matrix (expr) for HCA-BM cells
- Cell-to-sample annotation (cellanno)
- Pseudotime values per cell (from Slingshot/TSCAN trajectory)
- Design matrix with first column as intercept (e.g., intercept + sex)
- Number of permutations (permuiter, recommended 100)

## Outputs

- Res$statistics table with per-gene fdr.overall columns for TDE and XDE
- Counts of significant genes per lineage at FDR < 0.05 (e.g., tde_xde_lineage_results.csv)
- Lists of TDE and XDE genes per lineage for marker recovery (CD14, HBB, GATA1, CD3D, CD19, CD27)

## How to apply

For each lineage (myeloid, erythroid, lymphoid in the HCA-BM example), run lamian_test() with test.type='time' on the expr, cellanno, pseudotime, and design objects, then with test.type='variable' and testvar=2 to detect sample-covariate-dependent expression dynamics using a permutation-based likelihood ratio test comparing nested models. Use the recommended permuiter=100; drop cells with NA pseudotime and exclude samples with fewer than 50 cells first to ensure adequate signal-to-noise ratio. In each permutation Lamian bootstraps cells to account for pseudotime variability and permutes pseudotime within each sample, so inference respects cross-sample variability and controls FDR appropriately in multi-sample settings. Extract Res$statistics and call TDE genes at fdr.overall < 0.05 and XDE genes at the fdr.*overall* column < 0.05, counting per lineage (expected XDE: 43/32/29 for myeloid/erythroid/lymphoid).

## Related tools

- **Lamian** (Implements the permutation-based likelihood ratio test for XDE (test.type='variable', testvar=2) and TDE (test.type='time') via lamian_test()) — https://github.com/Winnie09/Lamian
- **Slingshot** (One of the pseudotime inference methods whose pseudotime values serve as input to the Lamian tests)
- **TSCAN** (Default cluster-based minimum spanning tree method used by Lamian to construct the pseudotime input for testing) — zji90/TSCAN (installed via devtools::install_github("zji90/TSCAN"))

## Examples

```
Res <- lamian_test(expr=expr, cellanno=cellanno, pseudotime=pseudotime, design=design, test.type='variable', testvar=2, permuiter=100)
```

## Evaluation signals

- XDE counts reproduce Fig. 3 values at FDR < 0.05: 43/32/29 genes for myeloid/erythroid/lymphoid lineages in HCA-BM
- TDE counts reproduce 8475/7454/8953 genes for myeloid/erythroid/lymphoid lineages
- ~33% (N=14) of myeloid XDE genes are significantly enriched on sex chromosomes, consistent with a sex-associated covariate
- Known lineage markers (CD14, HBB, GATA1, CD3D, CD19, CD27) are recovered among significant genes
- Output table tde_xde_lineage_results.csv contains per-lineage counts and gene lists matching reported values

## Limitations

- Simulations for TDE, TCD, and XCD detection are deferred to Supplementary Notes, so main-text validation details are limited
- Permutation-based testing is computationally driven (permuiter default/recommendation of 100), trading compute for p-value calibration
- Simultaneous accounting of pseudotime inference uncertainty, gene expression and cell abundance uncertainty, and cross-sample/cross-cell variability remains a future research topic

## Evidence

- [discussion] XDE detection using permutation-based likelihood ratio test: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] TDE permutation scheme bootstraps cells and permutes pseudotime within samples: "In each permutation, we first bootstrap the cells to account for pseudotime variability, and we then permute the pseudotime of the cells within each sample."
- [readme] Cross-sample variability is accounted for, controlling FDR in multi-sample tests: "when identifying cell abundance or gene expression changes associated with pseudotime and sample covariates, Lamian accounts for variability across biological samples"
- [abstract] Lamian detects gene expression and cell density changes associated with sample covariates: "Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects"
