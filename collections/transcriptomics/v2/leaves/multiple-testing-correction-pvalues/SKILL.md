---
name: multiple-testing-correction-pvalues
description: Use when many hypothesis tests are run in parallel (e.g., scoring 64
  latent features for association with ICB responder vs non-responder groups) and
  per-feature significance must be assessed with adjusted p-values to control false
  positives.
when_to_use_negative:
- Only one hypothesis is tested — no correction is needed
- The tests are independent confirmatory analyses with a pre-registered primary endpoint,
  where correction would be inappropriate
- Input is already a table of adjusted p-values; re-correcting would be double-counting
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: Scanpy
  role: Source of the Wilcoxon rank-sum test framework whose per-feature p-values
    are corrected; Scanpy (with Seurat) is cited as the differential-analysis toolkit
  repo: https://github.com/lixiangchun/Miscell
- name: Seurat
  role: Reference implementation of the Wilcoxon rank-sum differential analysis whose
    p-values feed into the multiple-testing correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:6bc2f9f99d0bf1982fd2c232bfeac22d8b226816545c00c58fb600cba9eeba93
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - Scanpy
  - Seurat
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multiple-testing-correction-pvalues

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects raw p-values from a family of statistical tests (e.g., 64 Wilcoxon rank-sum tests of Miscell latent features against ICB therapy response) for multiple comparisons, retaining only features significant at adjusted p < 0.05.

## When to use

Use when many hypothesis tests are run in parallel (e.g., scoring 64 latent features for association with ICB responder vs non-responder groups) and per-feature significance must be assessed with adjusted p-values to control false positives.

## When NOT to use

- Only one hypothesis is tested — no correction is needed
- The tests are independent confirmatory analyses with a pre-registered primary endpoint, where correction would be inappropriate
- Input is already a table of adjusted p-values; re-correcting would be double-counting

## Inputs

- Vector of raw p-values from per-feature Wilcoxon rank-sum tests (one test per latent feature, 64 tests)
- Patient-level feature score matrix (298 urothelial carcinoma ICB patients x 64 Miscell latent features) with responder/non-responder labels

## Outputs

- Full association table (e.g., icb_response_latent_features.tsv) with test statistic, raw p-value, adjusted p-value, and significance flag per feature
- Set of significant latent features (adjusted p < 0.05)

## How to apply

First compute a raw p-value per feature using the Wilcoxon rank-sum test (the differential analysis adopted by Seurat and Scanpy) comparing responder and non-responder patient groups. Then correct the full vector of 64 p-values for multiple testing and apply an adjusted-p threshold of 0.05 to flag significant features. In the Miscell urothelial carcinoma ICB application (298 patients, Mariathasan et al. 2018 bulk RNA-seq), this procedure retained exactly 3 of 64 latent features, including the 31st feature whose low scores were characterized by upregulation of IFNG, IL12B, CXCR3, and CD19. Record the test statistic, raw and adjusted p-values, and significance flags for every feature so the correction is auditable, then write the full table (e.g., icb_response_latent_features.tsv). Validation hinges on the corrected threshold reproducing the expected number of significant hits rather than on raw p-values alone.

## Related tools

- **Scanpy** (Source of the Wilcoxon rank-sum test framework whose per-feature p-values are corrected; Scanpy (with Seurat) is cited as the differential-analysis toolkit) — https://github.com/lixiangchun/Miscell
- **Seurat** (Reference implementation of the Wilcoxon rank-sum differential analysis whose p-values feed into the multiple-testing correction)

## Examples

```
from scipy.stats import ranksums; from statsmodels.stats.multitest import multipletests; pvals = [ranksums(f[responder], f[nonresponder]).pvalue for f in features.T]; reject, padj, _, _ = multipletests(pvals, alpha=0.05, method='fdr_bh')
```

## Evaluation signals

- Exactly 3 of the 64 latent features reach adjusted p < 0.05 in the urothelial carcinoma ICB cohort, matching the reported result
- Adjusted p-values are monotonically >= raw p-values for every feature (a sanity invariant of any valid correction)
- The significant low-score feature (the 31st) shows upregulation of IFNG, IL12B, CXCR3 and CD19 in the low-score patient group
- Output table contains 64 rows, one per latent feature, each with test statistic, raw p-value, adjusted p-value, and significance flag

## Limitations

- The clinically significant latent feature extracted by Miscell lacks explainability; the paper notes future work is required to address this issue
- The biological mechanisms underlying the latent features remain unknown and require future exploration
- Significant features identified depend on cohort size; Miscell demands a large sample size to obtain good performance, limiting small-cohort applications

## Evidence

- [other] Wilcoxon test, adjusted p < 0.05: [evidence span withheld — anchor status: not_found]
- [other] immune environment signature of the significant low-scoring feature: [evidence span withheld — anchor status: not_found]
