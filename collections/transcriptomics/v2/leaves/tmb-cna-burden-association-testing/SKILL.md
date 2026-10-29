---
name: tmb-cna-burden-association-testing
description: Use when testing whether per-sample viral status (e.g., IIV31 positivity
  in UCEC) differs between groups in TMB, chromosome-level CNA burden, CIBERSORT CD8+/Treg
  fractions, Kaplan-Meier survival, or TV co-occurrence.
when_to_use_negative:
- Input lacks per-sample viral-status labels (e.g., only assembled contigs or read-level
  viRNAtrap output) — group split cannot be formed without prior sample-level calling
- Groups are paired/dependent samples or there are more than two groups — Wilcoxon
  rank-sum as used here is a two-sample unpaired test
- Viral calls for divergent viruses have not yet been verified by targeted blastn/blastx
  sequence analyses
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0091
tools:
- name: viRNAtrap
  role: Produces the per-sample viral calls (IIV31 status) from unmapped RNAseq reads
    that define the two groups being compared for TMB/CNA burden associations
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: keras
  role: Deep learning backend behind viRNAtrap's TensorFlow model used to predict
    which unmapped reads are viral before contig assembly
  repo: keras-team/keras
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:6a5bae2a300565a2c75e9fb7e12dc5f37fb507f3077a607d6bad38fa6589fcac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - keras
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tmb-cna-burden-association-testing

## Summary

Statistical workflow that tests whether a viral-status grouping (e.g., IIV31-positive vs IIV31-negative UCEC samples from viRNAtrap) is associated with tumor mutation burden, chromosome-level copy-number alteration burden, CIBERSORT-inferred immune fractions, overall survival, and Trichomonas vaginalis co-occurrence.

## When to use

Use when testing whether per-sample viral status (e.g., IIV31 positivity in UCEC) differs between groups in TMB, chromosome-level CNA burden, CIBERSORT CD8+/Treg fractions, Kaplan-Meier survival, or TV co-occurrence.

## When NOT to use

- Input lacks per-sample viral-status labels (e.g., only assembled contigs or read-level viRNAtrap output) — group split cannot be formed without prior sample-level calling
- Groups are paired/dependent samples or there are more than two groups — Wilcoxon rank-sum as used here is a two-sample unpaired test
- Viral calls for divergent viruses have not yet been verified by targeted blastn/blastx sequence analyses

## Inputs

- Per-sample TSV/CSV (Source Data file 4) with columns for sample ID, IIV31 status, overall survival time/event, tumor mutation burden, chromosome-level copy-number alteration burden, CIBERSORT CD8+ T-cell and Treg fractions, and Trichomonas vaginalis status (UCEC patients from TCGA)
- IIV31-positive sample calls previously produced by viRNAtrap from unmapped RNAseq reads

## Outputs

- fig4b_e_association_statistics.csv with test statistics and p-values for the log-rank, Wilcoxon (TMB, CNA, CD8+, Treg), and Fisher exact (odds ratio, p = 1.4e-8) tests
- Kaplan-Meier curves comparing overall survival of IIV31-positive vs IIV31-negative UCEC samples

## How to apply

Load the per-sample table (deposited Source Data file 4) containing IIV31 status, overall survival, tumor mutation burden, chromosome-level copy-number alteration burden, CIBERSORT immune fractions, and Trichomonas vaginalis status for UCEC patients. Split patients into IIV31-positive and IIV31-negative groups, then test overall survival with a two-sided log-rank test on Kaplan-Meier curves. Compare TMB between groups with a Wilcoxon rank-sum test, and repeat the same two-sample rank-sum test for chromosome-level CNA burden and for CIBERSORT-inferred CD8+ T-cell and regulatory T-cell (Treg) proportions. Build a 2x2 contingency table of IIV31 status versus Trichomonas vaginalis positivity and apply Fisher's exact test, reporting both the odds ratio and p-value (expected p = 1.4e-8). Write all test statistics and p-values to fig4b_e_association_statistics.csv so each result maps to a Fig. 4b-e panel.

## Related tools

- **viRNAtrap** (Produces the per-sample viral calls (IIV31 status) from unmapped RNAseq reads that define the two groups being compared for TMB/CNA burden associations) — https://github.com/AuslanderLab/virnatrap.git
- **keras** (Deep learning backend behind viRNAtrap's TensorFlow model used to predict which unmapped reads are viral before contig assembly) — keras-team/keras

## Examples

```
from scipy.stats import mannwhitneyu, fisher_exact; U, p_tmb = mannwhitneyu(tmb_iiv31_pos, tmb_iiv31_neg, alternative='two-sided'); OR, p_tv = fisher_exact([[a, b], [c, d]])
```

## Evaluation signals

- Fisher exact test on the 2x2 IIV31-vs-TV contingency table reproduces the reported p-value of 1.4e-8
- All log-rank and Wilcoxon rank-sum statistics (TMB, CNA burden, CIBERSORT CD8+/Treg) return finite values consistent with the reported Fig. 4b-e results
- Fisher output includes both an odds ratio and a p-value, not just the p-value
- fig4b_e_association_statistics.csv contains one row per test with named statistics and p-values covering survival, TMB, CNA, CD8+, Treg, and TV enrichment
- Group split yields non-empty IIV31-positive and IIV31-negative samples with survival times available for the Kaplan-Meier log-rank test

## Limitations

- The source of IIV31 in UCEC samples is not confirmed; transmission (e.g., via crab louse) is only hypothesized as a consequence of Trichomonas vaginalis infection
- IIV31 presence may be a secondary infection in patients already infected with TV or another pathogen that suppresses the human anti-viral response — association does not establish causation
- All contaminants of viruses that may have infected samples during laboratory procedures cannot be modeled, so some viral-status labels may reflect contamination

## Evidence

- [discussion] characterize viruses that are expressed across 14 cancer tissues from TCGA and analyze their genomic and survival correlates: "We employ viRNAtrap for exploratory data analysis and characterize viruses that are expressed across 14 cancer tissues from TCGA and analyze their genomic and survival correlates"
- [methods] Split patients into IIV31-positive and IIV31-negative groups and test the overall survival difference with a two-sided log-rank test on Kaplan-Meier curves.: [evidence span withheld — anchor status: not_found]
- [methods] Build the 2x2 contingency table of IIV31 status versus Trichomonas vaginalis positivity and apply Fisher's exact test, reporting the odds ratio and p-value.: [evidence span withheld — anchor status: not_found]
- [discussion] While we have not yet confirmed the source of this virus, our results imply that its presence may be a direct or indirect consequence of Trichomonas vaginalis infection: "While we have not yet confirmed the source of this virus, our results imply that its presence may be a direct or indirect consequence of Trichomonas vaginalis infection"
- [readme] Use the Tensorflow model to predict which of the unmapped reads are viral; this is performed by the included software.: "Use the Tensorflow model to predict which of the unmapped reads are viral; this is performed by the included software."
