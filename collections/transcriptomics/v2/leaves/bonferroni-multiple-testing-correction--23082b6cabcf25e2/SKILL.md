---
name: bonferroni-multiple-testing-correction
description: Use when thousands of genes are tested in parallel by CTMM (e.g., REML
  JK and HE Wald p-values per gene) and a strict family-wise-error-controlled gene
  hit list is required before ranking or cross-method intersection.
when_to_use_negative:
- When only a handful of pre-specified genes are tested and a family-wise correction
  across 11,231 genes would be overly conservative and unnecessary
- When the goal is FDR control at a desired discovery rate rather than strict family-wise
  error control, since Bonferroni is the most conservative choice
- When sample sizes are below ~50 individuals, where the article notes CTMM tests
  are not reliable and corrected p-values would be untrustworthy
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: ctmm
  role: Produces the per-gene REML (jackknife) and HE Wald p-values (p['V'], p['ct_beta'])
    that form the family over which the Bonferroni correction is applied
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: Python
  role: Implements the correction arithmetic (p * 11,231 or alpha/11,231 threshold),
    the REML-HE intersection, the p > 0.01 mean filter, and writes ipsc_ctp_hit_table.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:0b8695f0b7eff357add3c620eb56edabc07cd1f59159659dcf71d519e419e094
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - ctmm
  - mvtnorm
  - numDeriv
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bonferroni-multiple-testing-correction

## Summary

Apply Bonferroni correction across all 11,231 tested genes in the iPSC-to-endoderm CTMM analysis to control the family-wise error rate when calling cell type-specific variance differentiation hits from REML (jackknife) and Haseman-Elston Wald p-values.

## When to use

Use when thousands of genes are tested in parallel by CTMM (e.g., REML JK and HE Wald p-values per gene) and a strict family-wise-error-controlled gene hit list is required before ranking or cross-method intersection.

## When NOT to use

- When only a handful of pre-specified genes are tested and a family-wise correction across 11,231 genes would be overly conservative and unnecessary
- When the goal is FDR control at a desired discovery rate rather than strict family-wise error control, since Bonferroni is the most conservative choice
- When sample sizes are below ~50 individuals, where the article notes CTMM tests are not reliable and corrected p-values would be untrustworthy

## Inputs

- Per-gene Wald p-value vectors for variance differentiation (V) from ctp.free_REML with jack_knife=True
- Per-gene Wald p-value vectors for variance differentiation (V) from ctp.free_HE
- Per-gene Wald p-value vectors for mean differentiation (ct_beta) from the same fits
- Total number of tested genes (11,231) to define the correction family

## Outputs

- Set of Bonferroni-significant candidate genes per method
- Intersection list of genes significant in both REML (JK) and HE
- Flagged subset of variance hits lacking significant mean differentiation (p > 0.01 in REML; 85 genes)
- Ranked hit table written to ipsc_ctp_hit_table.csv with gene, REML (JK) p-values, HE p-values, and variance-only flag

## How to apply

For each fitting method (ctp.free_REML with jack_knife=True and ctp.free_HE on the CTP/ctnu/P pseudobulk inputs), collect the per-gene Wald p-values for variance differentiation (V) and mean expression differentiation (ct_beta). Multiply each p-value by the number of tested genes (11,231 in this study) or equivalently compare to alpha/11,231, applying the correction within each method separately. Select candidate genes that remain significant after Bonferroni correction in BOTH the REML (JK) and HE results, then among the variance hits flag those whose mean-expression Wald p-value exceeds 0.01 in REML (85 such genes in this dataset). Rank survivors by p-value and cross-check top hits (e.g., POU5F1, HSPA8, NDUFB4) against the reported Table 1 p-values before writing the hit table.

## Related tools

- **ctmm** (Produces the per-gene REML (jackknife) and HE Wald p-values (p['V'], p['ct_beta']) that form the family over which the Bonferroni correction is applied) — https://github.com/Minhui-Chen/ctmm_python
- **Python** (Implements the correction arithmetic (p * 11,231 or alpha/11,231 threshold), the REML-HE intersection, the p > 0.01 mean filter, and writes ipsc_ctp_hit_table.csv)

## Examples

```
from ctmm import ctp; free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True); sig = p_jk['V'] * 11231 < 0.05
```

## Evaluation signals

- The correction family size must equal the number of tested genes (11,231) for each method before any selection
- No selected candidate gene has a raw p-value above the alpha/11,231 Bonferroni threshold in either method
- Every gene in the final hit list is significant in BOTH REML (JK) and HE; genes significant in only one method are excluded
- The variance-only subset (significant V, ct_beta p > 0.01 in REML) must contain exactly 85 genes as reported
- Top-ranked genes reproduce the reported Table 1 entries: POU5F1, HSPA8, and NDUFB4 appear at the top with matching p-values

## Limitations

- Bonferroni across 11,231 genes is very conservative and may miss true variance-differentiation signals, especially given modest scRNA-seq sample sizes
- The Free test is slightly inflated, so even corrected hits may require biological validation and replication before being trusted
- CTMM is not reliable for sample sizes below ~50, which undermines any p-value thresholding at the corrected level

## Evidence

- [abstract] Top hits and count verification: "including 85 genes that do not have significant stage-specific mean expression"
- [methods] Wald test output structure: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [discussion] Jackknife-based test recommendation: "propose a jackknife-based test using REML as the most powerful and robust method"
- [readme] Jackknife invocation in README: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [discussion] Reliability caveat: "CTMM is not reliable for sample sizes below ~50"
