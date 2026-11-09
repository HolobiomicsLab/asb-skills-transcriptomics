---
name: fdr-thresholded-significance-calling
description: Use when Normalisr association outputs (gRNA-gene TSS-repression, trans-association,
  differential expression, or co-expression P values) must be filtered into significant
  hits using stated Q-value and logFC thresholds.
when_to_use_negative:
- Hypothesis testing of arbitrary nonlinear associations — Normalisr is explicitly
  not designed for this
- Causal network reconstruction via v-structures, regularized regression, or Granger
  causality with longitudinal data — outside Normalisr's scope
- Integrative studies spanning multiple datasets requiring batch-effect correction,
  which Normalisr does not address
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Computes exact linear-model association P values on normalized single-cell
    data; its Q-value/logFC outputs are the object of thresholded significance calling
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:8c6f2dd33d1d27386a9a1b7111a911626f0aa0f47e04ed1f7a0051c1d96d0956
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# fdr-thresholded-significance-calling

## Summary

Call significant gene regulations, differential expression, or co-expression links from Normalisr's linear-model association results by applying explicit FDR (Q-value) and effect-size (logFC) cutoffs, e.g. Q ≤ 0.05 with logFC ≤ −0.2 for TSS-repression gRNA efficacy and Q ≤ 0.2 with |logFC| ≥ 0.05 for trans-associations in the CROP-seq CRISPRi screens.

## When to use

Use when Normalisr association outputs (gRNA-gene TSS-repression, trans-association, differential expression, or co-expression P values) must be filtered into significant hits using stated Q-value and logFC thresholds.

## When NOT to use

- Hypothesis testing of arbitrary nonlinear associations — Normalisr is explicitly not designed for this
- Causal network reconstruction via v-structures, regularized regression, or Granger causality with longitudinal data — outside Normalisr's scope
- Integrative studies spanning multiple datasets requiring batch-effect correction, which Normalisr does not address

## Inputs

- Normalisr association output tables (gRNA-gene TSS-repression and trans-association results) with P values, Q-values, and logFC
- UMI read count matrix (tsv or sparse mtx, Cell Ranger output) and gRNA assignment matrix
- Gene and cell name text files, one per line

## Outputs

- Significant-hit tables filtered by Q-value and logFC thresholds (e.g. supplementary_data_3.tsv of 833 high-confidence gene regulations)
- Reconstructed putative gene regulatory network of high-confidence TSS-targeting gRNA -> targeted gene -> trans-gene regulations
- Significant co-expression or differential expression gene lists under the chosen Q cutoff

## How to apply

Run Normalisr's association modules on the UMI read-count and gRNA-assignment matrices, then apply paper-specific FDR cutoffs to the resulting Q-values: for CRISPRi TSS-repression (gRNA targeting efficacy) use Q ≤ 0.05 combined with logFC ≤ −0.2; for trans-association regulations use Q ≤ 0.2 with |logFC| ≥ 0.05; for co-expression networks in dysfunctional T cells a strong cutoff of Q ≤ 10−15 was used to prioritize direct gene interactions. In causal inference mode, require all four Q-values ≤ 0.2 for a high-confidence gRNA -> targeted gene -> trans-gene regulation. Decision points include excluding nearby-gene (local/cis) repression signals, requiring reproducibility across distinct gRNAs targeting the same regulator, and requiring reproducibility between the full-scale and small-scale screens. Normalisr is non-parametric, so these final cutoffs (together with QC parameters) are the only adjustable parameters; because P values are estimated exactly without permutation, FDR estimation is well-behaved and fast.

## Related tools

- **Normalisr** (Computes exact linear-model association P values on normalized single-cell data; its Q-value/logFC outputs are the object of thresholded significance calling) — github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h  # see submodule help; thresholds (e.g. Q <= 0.05, logFC <= -0.2) are then applied to the output tsv of Q-values and logFC
```

## Evaluation signals

- The final filtered CRISPRi regulation table contains exactly the 833 high-confidence regulations reported in Supplementary Data 3 when full-scale and small-scale CROP-seq screens are used
- Every called TSS-repression regulation satisfies Q ≤ 0.05 and logFC ≤ −0.2
- Every called trans-association regulation satisfies Q ≤ 0.2 and |logFC| ≥ 0.05, with all four causal-inference Q-values ≤ 0.2 for high-confidence calls
- Regulations with nearby-gene (local/cis) repression or irreproducibility across distinct gRNAs targeting the same regulator, or between full-scale and small-scale screens, are excluded
- Q-values show the expected FDR behavior given exact (non-permutation) P values from the linear models

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms were not validated and may produce a different read-count distribution requiring revalidation of thresholds
- Q-value significance calling does not reconstruct causal networks beyond the gRNA-instrumented trans-effect framework, and batch effects in multi-dataset integrations are not handled
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations

## Evidence

- [results] Call TSS-repression regulations using the significance thresholds Q ≤ 0.05 and logFC ≤ −0.2 to confirm gRNA targeting efficacy.: [evidence span withheld — anchor status: not_found]
- [discussion] For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions: "For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions"
- [readme] Since Normalisr is non-parametric, the only adjustable parameters are for quality control and final cutoffs of differential or co-expression.: [evidence span withheld — anchor status: external_doc]
- [results] Applying the causal inference criteria to the full-scale and small-scale CROP-seq screens yielded 833 high-confidence putative gene regulations (all four Q ≤ 0.2): [evidence span withheld — anchor status: not_found]
- [discussion] Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations54: "Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations"
