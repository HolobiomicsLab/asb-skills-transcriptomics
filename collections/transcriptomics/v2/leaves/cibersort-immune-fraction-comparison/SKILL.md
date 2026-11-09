---
name: cibersort-immune-fraction-comparison
description: Use when testing whether inferred immune composition (e.g., CIBERSORT
  CD8+ T-cell and Treg fractions) differs between two predefined sample groups, such
  as viral-positive vs viral-negative TCGA tumor samples.
when_to_use_negative:
- You need to run immune deconvolution itself from an expression matrix — this skill
  consumes precomputed CIBERSORT fractions from the deposited Source Data file, it
  does not perform CIBERSORT inference
- The hypothesis is the IIV31-versus-Trichomonas vaginalis overlap — that is a separate
  2x2 Fisher exact test step (reported p = 1.4e-8) in the same workflow, not a rank-sum
  immune-fraction comparison
- Groups are paired, matched, or more than two — the article's workflow specifies
  two-sided Wilcoxon rank-sum tests on two independent patient groups
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CIBERSORT
  role: Upstream method whose inferred CD8+ T-cell and Treg fractions per UCEC sample
    are the quantities compared by Wilcoxon rank-sum tests between IIV31-positive
    and IIV31-negative groups
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:434ba67c136e3f88fecbe176896078b7385dfe33dfdeb5d6bb9ec07b6e83aa7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIBERSORT
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cibersort-immune-fraction-comparison

## Summary

Compares CIBERSORT-inferred immune cell proportions (CD8+ T cells and regulatory T cells) between IIV31-positive and IIV31-negative UCEC patients using two-sided Wilcoxon rank-sum tests, as part of the divergent-virus association analysis in the viRNAtrap study.

## When to use

Use when testing whether inferred immune composition (e.g., CIBERSORT CD8+ T-cell and Treg fractions) differs between two predefined sample groups, such as viral-positive vs viral-negative TCGA tumor samples.

## When NOT to use

- You need to run immune deconvolution itself from an expression matrix — this skill consumes precomputed CIBERSORT fractions from the deposited Source Data file, it does not perform CIBERSORT inference
- The hypothesis is the IIV31-versus-Trichomonas vaginalis overlap — that is a separate 2x2 Fisher exact test step (reported p = 1.4e-8) in the same workflow, not a rank-sum immune-fraction comparison
- Groups are paired, matched, or more than two — the article's workflow specifies two-sided Wilcoxon rank-sum tests on two independent patient groups

## Inputs

- Deposited Source Data file 4 with per-sample IIV31 status, overall survival, tumor mutation burden, chromosome-level copy-number alteration, CIBERSORT immune fractions (CD8+ T-cell and Treg proportions), and Trichomonas vaginalis status for UCEC patients
- Group labels splitting patients into IIV31-positive and IIV31-negative cohorts

## Outputs

- Wilcoxon rank-sum test statistics and p-values for CD8+ T-cell fraction differences between IIV31-positive and IIV31-negative groups
- Wilcoxon rank-sum test statistics and p-values for Treg fraction differences between the two groups
- fig4b_e_association_statistics.csv containing all test statistics for the Fig. 4b-e association panel

## How to apply

Load the deposited Source Data file 4, which contains per-sample IIV31 status, overall survival, tumor mutation burden, chromosome-level copy-number alteration, CIBERSORT immune fractions, and Trichomonas vaginalis status for UCEC patients. Split patients into IIV31-positive and IIV31-negative groups. For each immune cell type of interest (CD8+ T cells and Tregs), run a two-sided Wilcoxon rank-sum test comparing the CIBERSORT-inferred proportions between the two groups. Write the resulting statistics and p-values to fig4b_e_association_statistics.csv alongside the companion tests of the same Fig. 4b-e panel (log-rank survival, Wilcoxon TMB and CNA burden, and the Fisher exact test of TV enrichment, which must reproduce p = 1.4e-8). The rationale is to characterize the immune microenvironment correlates of divergent-virus presence detected by viRNAtrap.

## Related tools

- **CIBERSORT** (Upstream method whose inferred CD8+ T-cell and Treg fractions per UCEC sample are the quantities compared by Wilcoxon rank-sum tests between IIV31-positive and IIV31-negative groups)

## Examples

```
from scipy.stats import ranksums; stat, p = ranksums(cd8_iiv31_positive, cd8_iiv31_negative)
```

## Evaluation signals

- Both Wilcoxon rank-sum tests (CD8+ T cells, Tregs) return finite statistics consistent with the reported Fig. 4b-e results
- Rows for the CIBERSORT comparisons are present in fig4b_e_association_statistics.csv together with the survival, TMB, CNA, and Fisher test results
- Running the Fisher exact test on the same loaded Source Data file 4 reproduces the reported TV-enrichment p-value of 1.4e-8, validating the patient grouping
- Every patient with IIV31 status and non-missing CIBERSORT fractions falls into exactly one of the two comparison groups

## Limitations

- Immune fractions are CIBERSORT-inferred estimates rather than direct measurements, so results depend on the upstream deconvolution
- The article cautions that IIV31 presence may be a secondary infection in patients already infected with TV or another pathogen, so observed immune differences may be indirect
- Group labels depend on viRNAtrap detection, whose sensitivity is only guaranteed when more than five viral reads are present in a sample

## Evidence

- [other] Compare CIBERSORT-inferred CD8+ T-cell and regulatory T-cell (Treg) proportions between the two groups with Wilcoxon rank-sum tests.: [evidence span withheld — anchor status: not_found]
- [results] TV-positive UCEC samples are highly enriched with IIV31-positive samples, as determined by Fisher exact test with p value = 1.4e-8.: [evidence span withheld — anchor status: not_found]
- [discussion] Thus, it is possible that the presence of IIV31 is a secondary infection in patients already infected with TV or some other pathogen that suppresses the human anti-viral response: "Thus, it is possible that the presence of IIV31 is a secondary infection in patients already infected with TV or some other pathogen that suppresses the human anti-viral response"
