---
name: median-pvalue-inflation-calibration
description: Use when OTTERS Stage II output (testing.py) has produced per-gene TWAS
  Z-scores from multiple eQTL weight training methods that must be calibrated per
  method before ACAT-O aggregation, e.g. UKBB cardiovascular disease GWAS with eQTLGen-derived
  weights.
when_to_use_negative:
- The p values are already the ACAT-O omnibus output — genomic control must be applied
  per training method BEFORE aggregation, not to the combined omnibus p value.
- 'The goal is to obtain the direction of the gene–phenotype association: OTTERS cannot
  provide direction, and users must refer to the sign of the per-method TWAS Z-score
  rather than the calibrated p value.'
- Only a small subset of genes is available per method, making the median-of-squared-Z
  inflation estimate unstable or unrepresentative.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: OTTERS
  role: produces the Stage II per-gene TWAS Z-scores per training method that are
    the calibration input; the ACAT-O omnibus test consumes the calibrated p values
  repo: https://github.com/daiqile96/OTTERS
- name: scipy
  role: converts Z-scores to two-sided p values via scipy.stats.norm survival function
    and back after rescaling
- name: numpy
  role: computes the median of squared Z-scores per method and performs the vectorized
    genomic-control rescaling
- name: pandas
  role: assembles and writes the tab-separated per-gene, per-model adjusted p value
    table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:a9cfdc1b5c023a32b2451bc99b67bba4f6dad82fce1b4c56721337083e63b80b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# median-pvalue-inflation-calibration

## Summary

Per-method genomic control of TWAS test statistics in the OTTERS pipeline: within each of the five Stage II training models (P0.001, P0.05, lassosum, SDPR, PRS-CS), gene-level Z-scores are rescaled so the median two-sided p value equals 0.5 before the per-gene, per-method p values are aggregated by the ACAT-O omnibus test.

## When to use

Use when OTTERS Stage II output (testing.py) has produced per-gene TWAS Z-scores from multiple eQTL weight training methods that must be calibrated per method before ACAT-O aggregation, e.g. UKBB cardiovascular disease GWAS with eQTLGen-derived weights.

## When NOT to use

- The p values are already the ACAT-O omnibus output — genomic control must be applied per training method BEFORE aggregation, not to the combined omnibus p value.
- The goal is to obtain the direction of the gene–phenotype association: OTTERS cannot provide direction, and users must refer to the sign of the per-method TWAS Z-score rather than the calibrated p value.
- Only a small subset of genes is available per method, making the median-of-squared-Z inflation estimate unstable or unrepresentative.

## Inputs

- OTTERS Stage II output from testing.py: per-gene TWAS Z-scores for each of the five training models (P0.001, P0.05, lassosum, SDPR, PRS-CS)
- gene-level test statistic table keyed by CHROM and TargetID for all tested genes

## Outputs

- tab-separated per-gene, per-method table with columns CHROM, TargetID, and five genomic-control-adjusted p value columns (P0.001, P0.05, lassosum, SDPR, PRScs)
- per-method genomic-control inflation factors (median-based lambda) used for rescaling

## How to apply

Load the Stage II output containing per-gene TWAS Z-scores from the gene-based association tests with eQTL weights trained by P+T (P0.001, P0.05), lassosum, SDPR and PRS-CS, and convert each Z-score to a two-sided p value with the standard normal survival function (scipy.stats.norm). For each training model independently, estimate the genomic-control inflation factor as the median of the squared Z-scores across all tested genes (the null median of chi-squared(1) is ~0.4549), and divide every Z-score by the square root of that factor; this forces the median test p value to 0.5 without changing the ranking of genes. Recompute the adjusted two-sided p values from the rescaled Z-scores using numpy/scipy, performing the scaling separately per method so that heterogeneous per-method inflation does not dominate the omnibus statistic. Emit a tab-separated per-gene, per-method table with CHROM, TargetID and one adjusted p value column per model, matching the STAGE II 'models=P0.001,P0.05,lassosum,SDPR,PRScs' specification, as the direct input to ACAT-O. The rationale is that each PRS-trained weight set can carry different residual inflation, and ACAT-O assumes well-calibrated input p values; the omnibus test had near-optimal performance across all simulation scenarios considered only after this per-method calibration.

## Related tools

- **OTTERS** (produces the Stage II per-gene TWAS Z-scores per training method that are the calibration input; the ACAT-O omnibus test consumes the calibrated p values) — https://github.com/daiqile96/OTTERS
- **scipy** (converts Z-scores to two-sided p values via scipy.stats.norm survival function and back after rescaling)
- **numpy** (computes the median of squared Z-scores per method and performs the vectorized genomic-control rescaling)
- **pandas** (assembles and writes the tab-separated per-gene, per-model adjusted p value table)

## Examples

```
from scipy.stats import norm; import numpy as np; lam = np.median(Z**2)/0.45493642; p_adj = 2*norm.sf(np.abs(Z/np.sqrt(lam)))
```

## Evaluation signals

- For every training method, the median of the adjusted p values in the emitted table equals exactly 0.5.
- The table contains one row per tested gene with all five model columns (P0.001, P0.05, lassosum, SDPR, PRScs) present, plus CHROM and TargetID.
- All adjusted p values lie in (0, 1] and the per-method gene ranking is preserved (scaling is monotone within each method).
- Recomputing two-sided p values from the rescaled Z-scores via scipy.stats.norm reproduces the emitted columns bit-for-bit.
- Each method's inflation factor is derived from the median of its squared Z-scores over the same gene set that is rescaled.

## Limitations

- Calibrated p values are directionless; the sign must be recovered from the per-method TWAS Z-score, and OTTERS cannot provide the direction of the identified gene–phenotype associations.
- The median-based inflation estimate is only as good as the gene set tested; it assumes a majority of genes are null.
- OTTERS currently considers only cis-eQTL effects, and available eQTL summary statistics are mainly from individuals of European descent, limiting transferability of the calibrated results.

## Evidence

- [full_text] per-method genomic control before ACAT-O: [evidence span withheld — anchor status: not_found]
- [discussion] ACAT-O aggregation of per-method p values: "Apply ACAT-O to aggregate the TWAS p values from individual training methods"
- [discussion] omnibus test performance after calibration: "this test had near-optimal performance across all simulation scenarios considered"
- [readme] Stage II gene-based test producing the Z-scores being calibrated: "perform TWAS Stage II: gene-based association test using GWAS summary statistics."
