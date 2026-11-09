---
name: summary-statistics-zscore-to-pvalue-conversion
description: Use when OTTERS Stage II per-gene TWAS Z-scores from five training models
  must be rescaled via genomic control and converted to p values for downstream ACAT-O
  omnibus aggregation.
when_to_use_negative:
- Input is already ACAT-O omnibus p values or single-method TWAS p values that do
  not require cross-method genomic-control harmonization.
- Reference GWAS summary statistics for a single gene need conversion to p values
  inside Stage I weight training (e.g. PRS-CS input) rather than Stage II TWAS omnibus
  preparation.
- Z-scores come from individual-level reference methods (e.g. FUSION/PrediXcan) whose
  association testing pipeline has its own calibration rather than OTTERS per-method
  genomic control.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: OTTERS
  role: Produces the Stage II per-gene TWAS Z-scores (testing.py) that this skill
    genomic-controls and converts to adjusted p values
  repo: https://github.com/daiqile96/OTTERS
- name: scipy
  role: Two-sided p value computation from Z-scores via scipy.stats.norm survival
    function
- name: numpy
  role: Rescaling Z-scores by the genomic-control inflation factor and recomputing
    adjusted p values
- name: pandas
  role: Reading/writing the tab-separated per-gene, per-method adjusted p value table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:64ee9e7621d4b3054059f284f22970db0131308bca333b0844eadaff300d63ba
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
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# summary-statistics-zscore-to-pvalue-conversion

## Summary

Converts per-gene TWAS Z-scores from OTTERS Stage II (testing.py) into two-sided normal p values and applies genomic control per training model (P0.001, P0.05, lassosum, SDPR, PRS-CS) so the median p value equals 0.5 before ACAT-O omnibus aggregation.

## When to use

Use when OTTERS Stage II per-gene TWAS Z-scores from five training models must be rescaled via genomic control and converted to p values for downstream ACAT-O omnibus aggregation.

## When NOT to use

- Input is already ACAT-O omnibus p values or single-method TWAS p values that do not require cross-method genomic-control harmonization.
- Reference GWAS summary statistics for a single gene need conversion to p values inside Stage I weight training (e.g. PRS-CS input) rather than Stage II TWAS omnibus preparation.
- Z-scores come from individual-level reference methods (e.g. FUSION/PrediXcan) whose association testing pipeline has its own calibration rather than OTTERS per-method genomic control.

## Inputs

- OTTERS Stage II output from testing.py: per-gene TWAS Z-scores for each of the five training models (P0.001, P0.05, lassosum, SDPR, PRS-CS)
- List of tested genes with CHROM and TargetID identifiers

## Outputs

- Tab-separated per-gene, per-method table with columns CHROM, TargetID and one genomic-control-adjusted p value column per training model (P0.001, P0.05, lassosum, SDPR, PRScs), ready for ACAT-O aggregation

## How to apply

Load the Stage II output from testing.py containing per-gene TWAS Z-scores computed with eQTL weights trained by P+T (P0.001, P0.05), lassosum, SDPR and PRS-CS, and convert each Z-score to a two-sided p value with the standard normal survival function via scipy.stats.norm. For each training model independently, estimate the genomic-control inflation factor from the median of the squared Z-scores across all tested genes and divide every Z-score by its square root, forcing the median p value to 0.5; this stabilizes per-method test statistics before the ACAT-O omnibus test combines the five models. Recompute the adjusted two-sided p values from the rescaled Z-scores with numpy/scipy and emit a tab-separated per-gene, per-method table with CHROM, TargetID and one adjusted p value column per model matching the STAGE II 'models=P0.001,P0.05,lassosum,SDPR,PRScs' specification. Note that because ACAT-O consumes p values only, direction of association must be recovered separately from the sign of the original Z-scores per method.

## Related tools

- **OTTERS** (Produces the Stage II per-gene TWAS Z-scores (testing.py) that this skill genomic-controls and converts to adjusted p values) — https://github.com/daiqile96/OTTERS
- **scipy** (Two-sided p value computation from Z-scores via scipy.stats.norm survival function)
- **numpy** (Rescaling Z-scores by the genomic-control inflation factor and recomputing adjusted p values)
- **pandas** (Reading/writing the tab-separated per-gene, per-method adjusted p value table)

## Examples

```
from scipy.stats import norm; import numpy as p; pvals = 2*norm.sf(np.abs(z)/np.sqrt(np.median(z**2)))
```

## Evaluation signals

- For every training method, the median of the adjusted p values in the emitted table equals exactly 0.5.
- The output table contains one row per tested gene and all five model columns (P0.001, P0.05, lassosum, SDPR, PRScs) are present.
- Column headers match the STAGE II 'models=P0.001,P0.05,lassosum,SDPR,PRScs' specification with CHROM and TargetID identifiers.
- Adjusted p values are recomputed from rescaled Z-scores, so unadjusted p values recalculated from raw Z-scores differ by the per-method genomic-control factor.
- All adjusted p values lie in [0,1] and two-sided p values are consistent with the rescaled normal test statistics.

## Limitations

- OTTERS cannot provide the direction of identified gene-phenotype associations; users must refer to the sign of the TWAS Z-score per training method, so the p value table alone loses directionality.
- The p value conversion step does not propagate training model uncertainty; it operates only on Stage II Z-scores after Stage I weight training.
- p values are generated per model for ACAT-O aggregation, so per-method genome-wide significance decisions should not be made on unadjusted single-model p values.

## Evidence

- [discussion] TWAS test statistics are scaled so that the median test p value equals 0.5: [evidence span withheld — anchor status: not_found]
- [intro] OTTERS omnibus test via ACAT-O: "OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value using the aggregated"
- [intro] Stage II association testing: "Stage II of TWAS proceeds by using these trained eQTL effect sizes to impute GReX within an independent GWAS of a complex human disease or trait"
- [discussion] Direction not provided by OTTERS: "our OTTERS cannot provide the direction of the identified gene-phenotype associations"
- [readme] OTTERS Stage II workflow: "perform TWAS Stage II: gene-based association test using GWAS summary statistics."
