---
name: fdr-multiple-testing-correction
description: Use when a per-gene test across many genes (e.g. 100-gene Lamian XDE/TDE
  permutation likelihood-ratio tests) yields raw p-values that must be corrected for
  multiplicity before calling genes at FDR.overall < 0.05.
when_to_use_negative:
- The analysis already produces adjusted values (e.g. the table already contains FDR.overall)
  — re-correcting would double-adjust
- Only a single hypothesis is tested (no multiplicity), so no correction is needed
- The p-values come from cell-level tests that ignore sample-to-sample variation;
  Lamian's rationale is that such p-values do not generalize, so correction would
  not fix the underlying calibration problem
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Lamian
  role: Produces the permutation-based likelihood-ratio p-values (TDE/XDE tests) that
    are subsequently converted to FDR and thresholded at 0.05
  repo: https://github.com/Winnie09/Lamian
- name: Monocle
  role: Its GAM is tailored into monocle2TrajTestCorr whose null-vs-full model LRT
    p-values feed the FDR correction step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:ef51634d7b95cc20074bb8669f6b30c5b52e8779861f3fccfd1a2a635da95be7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fdr-multiple-testing-correction

## Summary

Converts per-gene permutation-based likelihood-ratio p-values from Lamian-style pseudotime tests (TDE/XDE) into FDR values via Benjamini-Hochberg correction, then calls significant genes at an FDR.overall < 0.05 cutoff.

## When to use

Use when a per-gene test across many genes (e.g. 100-gene Lamian XDE/TDE permutation likelihood-ratio tests) yields raw p-values that must be corrected for multiplicity before calling genes at FDR.overall < 0.05.

## When NOT to use

- The analysis already produces adjusted values (e.g. the table already contains FDR.overall) — re-correcting would double-adjust
- Only a single hypothesis is tested (no multiplicity), so no correction is needed
- The p-values come from cell-level tests that ignore sample-to-sample variation; Lamian's rationale is that such p-values do not generalize, so correction would not fix the underlying calibration problem

## Inputs

- Numeric vector or column of raw per-gene p-values from the permutation-based likelihood ratio test (permuiter >= 100 for real applications)
- Gene identifiers matching the rows of monocle2_xde_results.csv (100 genes in the expdata example)

## Outputs

- Per-gene FDR values appended to the results table (monocle2_xde_results.csv columns: statistic, p-value, FDR)
- XDE gene set defined by FDR.overall < 0.05

## How to apply

Collect the raw p-value vector from the per-gene permutation-based likelihood ratio test that compares the nested null model g(E(Y)) ~ beta0 + f2(pseudotime) against the full model g(E(Y)) ~ beta0 + f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group). Apply Benjamini-Hochberg multiple testing correction (e.g. R p.adjust with method='BH') across all tested genes to obtain FDR values. Threshold the corrected values at FDR.overall < 0.05 to define the XDE gene set. The rationale is that Lamian's inference first accounts for cross-sample variability, so the resulting p-values are appropriately calibrated before correction; correction then controls sample-specific and gene-wise false discoveries across the 100-gene (or genome-wide) test panel.

## Related tools

- **Lamian** (Produces the permutation-based likelihood-ratio p-values (TDE/XDE tests) that are subsequently converted to FDR and thresholded at 0.05) — https://github.com/Winnie09/Lamian
- **Monocle** (Its GAM is tailored into monocle2TrajTestCorr whose null-vs-full model LRT p-values feed the FDR correction step)

## Examples

```
fdr <- p.adjust(pval, method="BH"); xde_genes <- genes[fdr < 0.05]
```

## Evaluation signals

- Output table contains one likelihood-ratio statistic, p-value, and FDR per tested gene (e.g. 100 rows for the expdata example)
- All FDR values fall in [0,1] and are monotonically non-decreasing with the raw p-values (BH invariant)
- Minimum FDR is <= the corresponding minimum raw p-value after BH adjustment
- The called XDE gene set exactly equals {genes with FDR.overall < 0.05}
- FDR is computed over all tested genes jointly, not per gene or per sample

## Limitations

- FDR correction does not compensate for miscalibrated p-values from methods that ignore cross-sample variability; Lamian argues such methods inflate sample-specific false discoveries
- The LRT here relies on permutation testing; permuiter >= 100 is recommended for real applications, so small permutation counts yield unstable p-values that propagate into the FDR
- Lamian modules 1 and 2 do not natively support Monocle2, Monocle3, or slingshot trajectories — the monocle2TrajTestCorr route is a custom adaptation

## Evidence

- [discussion] XDE detection using permutation-based likelihood ratio test comparing nested models: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [abstract] Lamian reduces false discoveries by accounting for cross-sample variability: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [readme] FDR control as a distinguishing property of the framework: "Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
- [other] Monocle2 adaptation generating the p-values that are FDR-corrected: [evidence span withheld — anchor status: not_found]
