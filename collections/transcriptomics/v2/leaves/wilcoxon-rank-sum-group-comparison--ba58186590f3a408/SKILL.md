---
name: wilcoxon-rank-sum-group-comparison
description: Use when you have a per-sample table with a binary viral-status grouping
  (e.g., IIV31-positive vs -negative from Source Data file 4) and continuous per-sample
  quantities (TMB, CNA burden, immune fractions) whose between-group differences must
  be tested without distributional assumptions.
when_to_use_negative:
- The outcome is binary (e.g., Trichomonas vaginalis positivity) rather than a continuous
  measurement — a Fisher exact test on the 2x2 contingency table is required instead
- The comparison involves survival curves rather than per-sample scalar values — a
  two-sided log-rank test on Kaplan-Meier curves is required instead
- Per-sample grouping labels (IIV31 status) have not yet been generated from viRNAtrap
  viral-read detection and expert verification
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: viRNAtrap
  role: Alignment-free deep-learning pipeline whose viral-read detection calls define
    the IIV31-positive/negative sample grouping that the Wilcoxon tests compare
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:6389f47c6d07f473bc1d96656174fa2b2c204a849c5ddc78905381655c95ac6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# wilcoxon-rank-sum-group-comparison

## Summary

Non-parametric (two-sided Wilcoxon rank-sum / Mann-Whitney) comparison of continuous measurements between IIV31-positive and IIV31-negative UCEC groups, used in the divergent-virus association analysis (Fig. 4b-e) to test TMB, chromosome-level CNA burden, and CIBERSORT CD8+ T-cell/Treg fractions.

## When to use

Use when you have a per-sample table with a binary viral-status grouping (e.g., IIV31-positive vs -negative from Source Data file 4) and continuous per-sample quantities (TMB, CNA burden, immune fractions) whose between-group differences must be tested without distributional assumptions.

## When NOT to use

- The outcome is binary (e.g., Trichomonas vaginalis positivity) rather than a continuous measurement — a Fisher exact test on the 2x2 contingency table is required instead
- The comparison involves survival curves rather than per-sample scalar values — a two-sided log-rank test on Kaplan-Meier curves is required instead
- Per-sample grouping labels (IIV31 status) have not yet been generated from viRNAtrap viral-read detection and expert verification

## Inputs

- Source Data file 4: per-sample UCEC table with IIV31 status, overall survival, tumor mutation burden, chromosome-level copy-number alteration burden, CIBERSORT immune cell fractions, and Trichomonas vaginalis status
- Binary grouping column: IIV31-positive vs IIV31-negative sample labels
- Continuous measurement columns: TMB, CNA burden, CD8+ T-cell fraction, Treg fraction

## Outputs

- Wilcoxon rank-sum test statistics and two-sided p-values for each variable (TMB, CNA burden, CD8+ T-cell proportion, Treg proportion) between IIV31 groups
- fig4b_e_association_statistics.csv containing all association test statistics (Wilcoxon, log-rank, Fisher exact) and p-values

## How to apply

Load the deposited Source Data file 4 for UCEC, split patients into IIV31-positive and IIV31-negative groups based on the per-sample IIV31 status column, then apply a Wilcoxon rank-sum test to each continuous variable: (1) tumor mutation burden (TMB), (2) chromosome-level copy-number alteration (CNA) burden, and (3) CIBERSORT-inferred CD8+ T-cell and regulatory T-cell (Treg) proportions. Report finite test statistics and two-sided p-values for each comparison, consistent with the Fig. 4b-e results, and write all statistics together with the complementary log-rank survival test and Fisher exact test (IIV31 x TV positivity, reported p = 1.4e-8) to fig4b_e_association_statistics.csv. The rationale is that TMB, CNA burden, and CIBERSORT fractions are non-normal, heteroscedastic tumor measurements, so a rank-based test is preferred over a t-test; the grouping is dictated by the viRNAtrap-derived viral detection calls rather than by user-defined thresholds.

## Related tools

- **viRNAtrap** (Alignment-free deep-learning pipeline whose viral-read detection calls define the IIV31-positive/negative sample grouping that the Wilcoxon tests compare) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(df.loc[df.IIV31_pos, 'TMB'], df.loc[~df.IIV31_pos, 'TMB'], alternative='two-sided')
```

## Evaluation signals

- The Fisher exact test on the 2x2 IIV31 x TV table reproduces the reported p-value of 1.4e-8
- All Wilcoxon statistics (TMB, CNA burden, CD8+ T-cell, Treg comparisons) return finite values consistent with the published Fig. 4b-e results
- Group sizes after splitting on IIV31 status sum to the total number of UCEC patients in Source Data file 4 with no unclassified samples
- Output CSV fig4b_e_association_statistics.csv contains one row per test with non-null statistic and p-value columns
- Each IIV31-positive sample is additionally verified to have multiple viral reads by targeted blastn-/blastx-based sequence analyses

## Limitations

- The source of IIV31 in UCEC samples is not confirmed; the association with Trichomonas vaginalis infection (transmission route, e.g., via crab louse) is only hypothesized, so the group comparison cannot establish causality
- IIV31 positivity may be a secondary infection in patients already infected with TV or another pathogen that suppresses the anti-viral response — an untested confounding hypothesis for any group difference
- Not possible to model all contaminants of viruses that may have infected samples during laboratory procedures, which can perturb the viral-status labels defining the groups

## Evidence

- [methods] Wilcoxon comparisons between IIV31 groups: [evidence span withheld — anchor status: not_found]
- [methods] Immune-fraction comparisons: [evidence span withheld — anchor status: not_found]
- [methods] Output file specification: [evidence span withheld — anchor status: not_found]
- [methods] Fisher exact validation target: [evidence span withheld — anchor status: not_found]
- [readme] Grouping basis (viral detection): "an alignment-free method to identify viral reads in RNAseq datasets based on a deep learning model and to assemble predicted viral contigs"
