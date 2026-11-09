---
name: fisher-exact-co-infection-enrichment-testing
description: Use when two binary per-sample viral/infection status calls (e.g., IIV31
  positivity and Trichomonas vaginalis positivity in TCGA-UCEC) must be tested for
  enrichment using an exact 2x2 test.
when_to_use_negative:
- Per-sample binary viral status is unavailable (no viRNAtrap contig calls or verified
  TV status), so a 2x2 contingency table cannot be built
- The comparison involves continuous variables such as TMB, CNA burden, or CIBERSORT
  CD8+/Treg proportions, where log-rank or Wilcoxon rank-sum tests (not Fisher's exact
  test) are the appropriate procedure
- Fewer than five viral reads are present in a sample, where viRNAtrap sensitivity
  is not guaranteed and the resulting positivity call is unreliable
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: viRNAtrap
  role: Alignment-free deep learning pipeline that classifies unmapped RNA-seq reads
    as viral and assembles predicted viral contigs, producing the per-sample IIV31
    and TV positivity calls used to populate the 2x2 table
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: blastn
  role: Post-processing verification that assembled viral contigs correspond to the
    divergent virus (IIV31) classification before positivity is called per sample
- name: blastx
  role: Targeted protein-level verification of the presence and classification of
    multiple viral reads for reported divergent viruses
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:1d08ed44040fcf38383a993dd2bf9e27a8b7328179b1bdce8638602c38ce337f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - blastn
  - blastx
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fisher-exact-co-infection-enrichment-testing

## Summary

Build a 2x2 contingency table of binary per-sample viral status calls (e.g., IIV31-positive vs Trichomonas vaginalis-positive in UCEC) and apply Fisher's exact test to quantify co-infection enrichment, reporting the odds ratio and p-value alongside the log-rank/Wilcoxon statistics of Fig. 4b-e.

## When to use

Use when two binary per-sample viral/infection status calls (e.g., IIV31 positivity and Trichomonas vaginalis positivity in TCGA-UCEC) must be tested for enrichment using an exact 2x2 test.

## When NOT to use

- Per-sample binary viral status is unavailable (no viRNAtrap contig calls or verified TV status), so a 2x2 contingency table cannot be built
- The comparison involves continuous variables such as TMB, CNA burden, or CIBERSORT CD8+/Treg proportions, where log-rank or Wilcoxon rank-sum tests (not Fisher's exact test) are the appropriate procedure
- Fewer than five viral reads are present in a sample, where viRNAtrap sensitivity is not guaranteed and the resulting positivity call is unreliable

## Inputs

- Deposited Source Data file 4: per-sample IIV31 status, Trichomonas vaginalis status, overall survival, TMB, chromosome-level CNA burden, and CIBERSORT immune fractions for TCGA-UCEC patients
- Per-sample viral contigs in FASTA format predicted by viRNAtrap from unmapped RNA-seq reads (files named *_unmapped.fastq)
- 2x2 contingency table counts of IIV31 status by TV positivity

## Outputs

- Fisher exact test odds ratio and p-value for IIV31 x TV enrichment (expected p = 1.4e-8)
- fig4b_e_association_statistics.csv containing all test statistics and p-values (log-rank, Wilcoxon, Fisher)
- 2x2 contingency table of IIV31 status versus TV positivity

## How to apply

Load the deposited Source Data file 4 containing per-sample IIV31 status, Trichomonas vaginalis status, overall survival, TMB, chromosome-level CNA burden, and CIBERSORT CD8+/Treg fractions for UCEC patients; per-sample viral positivity derives from viRNAtrap-predicted contigs assembled from unmapped (hg19- and phiX-filtered) RNA-seq reads and verified by targeted blastn/blastx. Build the 2x2 contingency table of IIV31 status versus TV positivity, apply Fisher's exact test, and report the odds ratio and p-value; in this cohort the test reproduces p = 1.4e-8, indicating TV-positive UCEC samples are highly enriched with IIV31-positive samples. Complement the table test with the two-sided log-rank test on Kaplan-Meier overall survival curves and Wilcoxon rank-sum tests on TMB, CNA burden, and CIBERSORT fractions. Write all test statistics and p-values to fig4b_e_association_statistics.csv. Validation requires the Fisher exact test to reproduce 1.4e-8 and all remaining tests to return finite statistics consistent with the reported Fig. 4b-e results.

## Related tools

- **viRNAtrap** (Alignment-free deep learning pipeline that classifies unmapped RNA-seq reads as viral and assembles predicted viral contigs, producing the per-sample IIV31 and TV positivity calls used to populate the 2x2 table) — https://github.com/AuslanderLab/virnatrap.git
- **blastn** (Post-processing verification that assembled viral contigs correspond to the divergent virus (IIV31) classification before positivity is called per sample)
- **blastx** (Targeted protein-level verification of the presence and classification of multiple viral reads for reported divergent viruses)

## Examples

```
from scipy.stats import fisher_exact; odds_ratio, p = fisher_exact([[a, b], [c, d]], alternative='two-sided')
```

## Evaluation signals

- Fisher exact test on the IIV31 x TV contingency table reproduces the reported p-value of 1.4e-8
- The reported odds ratio is finite and consistent with enrichment of TV-positive samples among IIV31-positive samples
- Contingency table row/column marginals sum to the total number of UCEC samples with both status calls
- Two-sided log-rank test on Kaplan-Meier survival and Wilcoxon rank-sum tests on TMB, CNA burden, and CIBERSORT fractions all return finite statistics matching Fig. 4b-e
- fig4b_e_association_statistics.csv contains entries for every reported test with no missing or non-finite p-values

## Limitations

- The source of IIV31 in UCEC samples is not confirmed; transmission via crab louse is only hypothesized, and IIV31 presence may be a direct or indirect consequence of TV infection
- IIV31 may be a secondary infection in patients already infected with TV or another pathogen that suppresses the human anti-viral response; this hypothesis is untested
- It is not possible to model all contaminants of viruses that may have infected samples during laboratory procedures, which can affect per-sample positivity calls feeding the contingency table

## Evidence

- [results] Fisher exact test reproduces the reported enrichment signal: [evidence span withheld — anchor status: not_found]
- [discussion] Contig verification precedes per-sample positivity calls: "For all divergent viruses reported in this study, the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses"
- [discussion] IIV31-TV relationship is a hypothesis, not a confirmed mechanism: "While we have not yet confirmed the source of this virus, our results imply that its presence may be a direct or indirect consequence of Trichomonas vaginalis infection"
- [discussion] Secondary infection alternative explanation: "it is possible that the presence of IIV31 is a secondary infection in patients already infected with TV or some other pathogen that suppresses the human anti-viral response"
- [readme] Per-sample viral reads derive from viRNAtrap on unmapped reads: "Use the Tensorflow model to predict which of the unmapped reads are viral; this is performed by the included software."
