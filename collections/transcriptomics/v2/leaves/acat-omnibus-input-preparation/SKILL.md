---
name: acat-omnibus-input-preparation
description: Use when OTTERS Stage II output from testing.py contains per-gene TWAS
  Z-scores for the five training models and they must be genomic-control adjusted
  before ACAT-O omnibus aggregation.
when_to_use_negative:
- Input is raw per-SNP GWAS summary statistics or Stage I eQTL weights rather than
  Stage II per-gene TWAS Z-scores — genomic control belongs to the gene-level TWAS
  statistics here
- P values were produced by an external TWAS pipeline with its own calibration; re-forcing
  the median p value to 0.5 would overwrite that pipeline's calibration
- Only a single training model is available, so ACAT-O omnibus aggregation is not
  applicable
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0625
tools:
- name: OTTERS
  role: Source of the Stage II per-gene TWAS Z-scores from testing.py that are genomic-control
    adjusted
  repo: https://github.com/daiqile96/OTTERS
- name: scipy
  role: Converts Z-scores to two-sided p values via the standard normal survival function
    (scipy.stats.norm)
- name: numpy
  role: Computes the median of squared Z-scores and the genomic-control rescaling
    of Z-scores
- name: pandas
  role: Writes the tab-separated per-gene, per-model adjusted p-value table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:d09bc78688b11fbc2ce7f840cc6a5df5475fbeed711766043bc6756d02ca1c25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# acat-omnibus-input-preparation

## Summary

Prepares the per-gene, per-model TWAS p-value table that feeds ACAT-O in OTTERS: each training model's Stage II Z-scores are genomic-control adjusted (median test p value forced to 0.5), re-converted to two-sided p values, and emitted as a TSV with one adjusted p column per model (P0.001, P0.05, lassosum, SDPR, PRScs).

## When to use

Use when OTTERS Stage II output from testing.py contains per-gene TWAS Z-scores for the five training models and they must be genomic-control adjusted before ACAT-O omnibus aggregation.

## When NOT to use

- Input is raw per-SNP GWAS summary statistics or Stage I eQTL weights rather than Stage II per-gene TWAS Z-scores — genomic control belongs to the gene-level TWAS statistics here
- P values were produced by an external TWAS pipeline with its own calibration; re-forcing the median p value to 0.5 would overwrite that pipeline's calibration
- Only a single training model is available, so ACAT-O omnibus aggregation is not applicable

## Inputs

- OTTERS Stage II output from testing.py (per-gene TWAS Z-scores, one set per training model)
- STAGE II model specification: models=P0.001,P0.05,lassosum,SDPR,PRScs

## Outputs

- Tab-separated per-gene, per-method table with columns CHROM, TargetID, and five genomic-control adjusted p value columns (P0.001, P0.05, lassosum, SDPR, PRScs)
- Adjusted p-value table ready for ACAT-O omnibus aggregation

## How to apply

Load the OTTERS Stage II output produced by testing.py, which holds per-gene TWAS Z-scores from gene-based association tests using eQTL weights trained by P+T (P0.001, P0.05), lassosum, SDPR, and PRS-CS. Convert each gene's Z-score to a two-sided p value with the standard normal survival function (scipy.stats.norm). For each training model independently, estimate the genomic-control inflation factor from the median of the squared Z-scores across all tested genes, and divide every Z-score by the square root of that factor so that the median test p value is forced to 0.5. Recompute the adjusted two-sided p values from the rescaled Z-scores with numpy/scipy and write a tab-separated per-gene, per-method table (CHROM, TargetID, and one adjusted p value column per model, matching the STAGE II 'models=P0.001,P0.05,lassosum,SDPR,PRScs' specification) for downstream ACAT-O omnibus aggregation. Rationale: forcing each model's median p to 0.5 removes per-method inflation before p values are combined, so no single training method dominates the omnibus result.

## Related tools

- **OTTERS** (Source of the Stage II per-gene TWAS Z-scores from testing.py that are genomic-control adjusted) — https://github.com/daiqile96/OTTERS
- **scipy** (Converts Z-scores to two-sided p values via the standard normal survival function (scipy.stats.norm))
- **numpy** (Computes the median of squared Z-scores and the genomic-control rescaling of Z-scores)
- **pandas** (Writes the tab-separated per-gene, per-model adjusted p-value table)

## Examples

```
from scipy.stats import norm; import numpy as np; p = 2*norm.sf(np.abs(Z)); Z_gc = Z/np.sqrt(np.median(Z**2)); p_adj = 2*norm.sf(np.abs(Z_gc))
```

## Evaluation signals

- For every training method, the median of the adjusted p values in the emitted table equals 0.5
- The table contains exactly one row per tested gene with all five model columns present (P0.001, P0.05, lassosum, SDPR, PRScs)
- Every adjusted p value lies in [0, 1] and equals 2 * norm.sf(abs(Z_rescaled)) for the rescaled Z-score
- The downstream ACAT-O omnibus test runs on the table without missing-value errors

## Limitations

- OTTERS cannot provide the direction of the identified gene-phenotype associations; users must refer to the sign of the TWAS Z-score per training method (direction is from the raw Z, not the adjusted p value)
- OTTERS currently considers only cis-eQTL effects, so the gene-level statistics being calibrated reflect cis associations only
- Forcing the median test p value to 0.5 assumes the majority of genes are null; a very dense true-association architecture could be over-corrected

## Evidence

- [intro] OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value: "OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value"
- [discussion] Apply ACAT-O to aggregate TWAS p values from individual training methods: "Apply ACAT-O to aggregate the TWAS p values from individual training methods"
- [discussion] The omnibus TWAS test had near-optimal performance across all simulation scenarios considered: "this test had near-optimal performance across all simulation scenarios considered"
- [readme] perform TWAS Stage II: gene-based association test using GWAS summary statistics: "perform TWAS Stage II: gene-based association test using GWAS summary statistics."
- [discussion] OTTERS cannot provide the direction of the identified gene-phenotype associations: "our OTTERS cannot provide the direction of the identified gene-phenotype associations"
