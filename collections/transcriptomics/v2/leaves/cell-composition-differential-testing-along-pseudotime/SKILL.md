---
name: cell-composition-differential-testing-along-pseudotime
description: Decision support for detecting condition- or covariate-associated changes
  in cell abundance/density along a multi-sample scRNA-seq pseudotime trajectory (Lamian
  TCD/XCD testing).
when_to_use_negative:
- The goal is differential gene expression along pseudotime (TDE/XDE) rather than
  cell abundance — use lamian_test instead of cellPropTest.
- The data are single-sample or the question is trajectory topology/branching differences
  — use Lamian's trajectory inference/uncertainty or differential topology module
  instead.
- The input lacks per-sample annotation or a sample-level design matrix, so cross-sample
  variability cannot be modeled.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: Lamian
  role: Provides cellPropTest for TCD (test.type='Time') and XCD (test.type='Variable')
    cell-composition testing along pseudotime
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default method to construct the pseudotime trajectory (cluster-based minimum
    spanning tree) that the composition test is run along
- name: Seurat
  role: Sample harmonization by embedding cells from multiple samples into a common
    low-dimensional space before pseudotime construction
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:9fd4d2db9473606f660c2e79ac2076ba4db5faba3d60c8b0ce86125d7916d97a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Seurat
  - TSCAN
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cell-composition-differential-testing-along-pseudotime

## Summary

Lamian's cellPropTest workflow tests whether cell composition/density along pseudotime differs across sample covariates (TCD: overall change; XCD: covariate-dependent change) by modeling cell counts in 100 equal-length pseudotime intervals while accounting for sample-to-sample variability.

## When to use

Decision support for detecting condition- or covariate-associated changes in cell abundance/density along a multi-sample scRNA-seq pseudotime trajectory (Lamian TCD/XCD testing).

## When NOT to use

- The goal is differential gene expression along pseudotime (TDE/XDE) rather than cell abundance — use lamian_test instead of cellPropTest.
- The data are single-sample or the question is trajectory topology/branching differences — use Lamian's trajectory inference/uncertainty or differential topology module instead.
- The input lacks per-sample annotation or a sample-level design matrix, so cross-sample variability cannot be modeled.

## Inputs

- Cell-to-sample annotation table (cellanno) mapping cells to biological samples
- Per-cell pseudotime values from TSCAN on harmonized (e.g., Seurat-embedded) cells
- Sample-level design matrix with intercept column and covariate columns (e.g., disease severity, sex, batch indicators)

## Outputs

- TCD/XCD test statistics with pval.overall and z.overall per test
- Covariate-associated cell-composition change significance (e.g., XCD p-value)

## How to apply

After samples are harmonized (e.g., Seurat embedding) and pseudotime is constructed with TSCAN's cluster-based minimum spanning tree, divide pseudotime from 0 to its maximum into 100 consecutive intervals of equal length and count cells per sample per interval. Run Lamian's cellPropTest with test.type='Time' and testvar for the overall cell-density change (TCD), or test.type='Variable' to test how a covariate (e.g., COVID severity or sex in the design matrix) modifies cell composition along pseudotime (XCD), passing the cell-to-sample annotation (cellanno), pseudotime vector, and sample-level design matrix whose first column is the intercept. The test returns pval.overall and z.overall; significance is judged at conventional thresholds such as p < 0.05. Rationale: modeling counts in intervals at the sample level accounts for cross-sample variability, reducing sample-specific false discoveries that non-sample-aware approaches cannot control.

## Related tools

- **Lamian** (Provides cellPropTest for TCD (test.type='Time') and XCD (test.type='Variable') cell-composition testing along pseudotime) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default method to construct the pseudotime trajectory (cluster-based minimum spanning tree) that the composition test is run along)
- **Seurat** (Sample harmonization by embedding cells from multiple samples into a common low-dimensional space before pseudotime construction)

## Examples

```
cellPropTest(cellanno, pseudotime, design, test.type='Variable', testvar=2)
```

## Evaluation signals

- COVID-19 severity XCD analysis should reproduce the reported p-value of 1.38e-61.
- TB atlas sex-associated XCD should reproduce pval.overall = 8.9e-6.
- TCD p-value in the COVID severity analysis should reach the reported < 2.22e-308.
- Cell counts must be tabulated over exactly 100 equal-length pseudotime intervals from 0 to the pseudotime maximum.
- The design matrix must include an intercept as its first column, with covariates (and optional batch indicators) in subsequent columns.

## Limitations

- Simulations for evaluating TCD and XCD detection are deferred to Supplementary Notes rather than the main text.
- Lamian modules do not seamlessly support Monocle2, Monocle3, or Slingshot trajectories; interoperability is deferred to future work.
- Simultaneously accounting for pseudotime inference uncertainty and cell abundance uncertainty remains an open research topic.

## Evidence

- [discussion] we divide the pseudotime from 0 to its maximum into 100 consecutive intervals of equal lengths.: "we divide the pseudotime from 0 to its maximum into 100 consecutive intervals of equal lengths."
- [other] Run Lamian cellPropTest with test.type = 'Time' (TCD) and test.type = 'Variable' (XCD) on the pseudotime divided into 100 equal-length intervals.: [evidence span withheld — anchor status: not_found]
- [abstract] Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [discussion] For sample harmonization, we used Seurat32 to embed cells into a common low-dimensional space.: "For sample harmonization, we used Seurat32 to embed cells into a common low-dimensional space."
- [other] Run cellPropTest(cellanno, pseudotime, design, test.type='Variable', testvar=2) to obtain the XCD result (pval.overall, z.overall): [evidence span withheld — anchor status: not_found]
