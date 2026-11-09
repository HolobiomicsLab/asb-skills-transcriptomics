---
name: gene-based-association-result-processing
description: Use when Stage II TWAS output from testing.py (per-gene Z-scores from
  five eQTL-weight training models) must be calibrated via genomic control so median
  test p values equal 0.5 before ACAT-O aggregation.
when_to_use_negative:
- Input is individual-level GWAS data for a gene-based test rather than already-computed
  Stage II per-gene TWAS statistics — run OTTERS testing.py first
- Only one training model is available, so ACAT-O omnibus aggregation across the five
  models (and hence this pre-aggregation calibration schema with five p value columns)
  does not apply
- Statistics are already genomic-control adjusted and median p values are approximately
  0.5 — re-scaling would distort well-calibrated results
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: OTTERS
  role: Provides the Stage II gene-based association test output (per-gene TWAS Z-scores
    from the five training models) that this skill calibrates via genomic control
  repo: https://github.com/daiqile96/OTTERS
- name: scipy
  role: Standard normal survival function (scipy.stats.norm) to convert rescaled Z-scores
    into two-sided adjusted p values
- name: numpy
  role: Computes the median of squared Z-scores for the genomic-control inflation
    factor and rescales Z-scores
- name: pandas
  role: Reads the Stage II output and writes the tab-separated per-gene, per-method
    adjusted p value table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:dcc2f67763607d9bcfbd5fb1f4dec8af27d449a3d8a9647439faaba6b784423e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-based-association-result-processing

## Summary

Genomic-control adjustment of per-method TWAS test statistics from the OTTERS Stage II gene-based association tests, producing calibrated per-gene, per-method p values suitable for ACAT-O omnibus aggregation across the five training models (P0.001, P0.05, lassosum, SDPR, PRS-CS).

## When to use

Use when Stage II TWAS output from testing.py (per-gene Z-scores from five eQTL-weight training models) must be calibrated via genomic control so median test p values equal 0.5 before ACAT-O aggregation.

## When NOT to use

- Input is individual-level GWAS data for a gene-based test rather than already-computed Stage II per-gene TWAS statistics — run OTTERS testing.py first
- Only one training model is available, so ACAT-O omnibus aggregation across the five models (and hence this pre-aggregation calibration schema with five p value columns) does not apply
- Statistics are already genomic-control adjusted and median p values are approximately 0.5 — re-scaling would distort well-calibrated results

## Inputs

- OTTERS Stage II output from testing.py: per-gene TWAS Z-scores for five training models (P0.001, P0.05, lassosum, SDPR, PRS-CS)
- Gene identifiers (TargetID) and chromosome (CHROM) per tested gene

## Outputs

- Tab-separated per-gene, per-method table with CHROM, TargetID, and five genomic-control-adjusted two-sided p value columns (P0.001, P0.05, lassosum, SDPR, PRScs)
- Calibrated p value table validated so the median adjusted p value per model equals 0.5, ready for ACAT-O omnibus aggregation

## How to apply

Load the OTTERS Stage II output from testing.py, which contains per-gene TWAS Z-scores from gene-based association tests using eQTL weights trained by P+T (P0.001, P0.05), lassosum, SDPR and PRS-CS. For each training model independently, estimate the genomic-control inflation factor from the median of the squared Z-scores across all tested genes, and divide every Z-score by the square root of that factor so that the median test p value is forced to 0.5. Convert Z-scores to two-sided p values with the standard normal survival function (scipy.stats.norm, numpy) before and after rescaling. Write a tab-separated per-gene, per-method table (CHROM, TargetID, plus one adjusted p value column per model matching the STAGE II 'models=P0.001,P0.05,lassosum,SDPR,PRScs' specification) for downstream ACAT-O omnibus aggregation.

## Related tools

- **OTTERS** (Provides the Stage II gene-based association test output (per-gene TWAS Z-scores from the five training models) that this skill calibrates via genomic control) — https://github.com/daiqile96/OTTERS
- **scipy** (Standard normal survival function (scipy.stats.norm) to convert rescaled Z-scores into two-sided adjusted p values)
- **numpy** (Computes the median of squared Z-scores for the genomic-control inflation factor and rescales Z-scores)
- **pandas** (Reads the Stage II output and writes the tab-separated per-gene, per-method adjusted p value table)

## Examples

```
from scipy.stats import norm; import numpy as np; z_gc = z / np.sqrt(np.median(z**2)/0.6744897501960817**2); p_gc = 2*norm.sf(np.abs(z_gc))
```

## Evaluation signals

- For every training model, the median of the adjusted p values in the emitted table equals exactly 0.5
- The output table contains one row per tested gene with all five model columns (P0.001, P0.05, lassosum, SDPR, PRScs) present
- Each model's genomic-control factor is estimated independently from the median of squared Z-scores across all tested genes, not pooled across models
- Adjusted p values are valid two-sided probabilities (0 < p <= 1) recomputed via the standard normal survival function after rescaling

## Limitations

- OTTERS cannot provide the direction of identified gene-phenotype associations; users must refer to the sign of the TWAS Z-score per training method
- OTTERS currently considers only cis-eQTL effects; trans-eQTL effects are not incorporated
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability of the resulting TWAS statistics

## Evidence

- [intro] OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value using the aggregated: "OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value using the aggregated"
- [discussion] Apply ACAT-O to aggregate the TWAS p values from individual training methods: "(3) Apply ACAT-O to aggregate the TWAS p values from individual training methods"
- [readme] perform TWAS Stage II: gene-based association test using GWAS summary statistics.: "perform TWAS Stage II: gene-based association test using GWAS summary statistics."
- [full_text] for each specific training model, all TWAS test statistics are scaled so that the median test p value equals 0.5: [evidence span withheld — anchor status: not_found]
