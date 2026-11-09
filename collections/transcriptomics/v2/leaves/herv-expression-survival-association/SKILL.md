---
name: herv-expression-survival-association
description: Use when per-sample viral/HERV contig calls from RNAseq (e.g. viRNAtrap
  FASTA contigs) must be dichotomized into HERV-positive vs HERV-negative groups and
  tested against TCGA overall survival or somatic driver mutations.
when_to_use_negative:
- The input is raw RNAseq FASTQ rather than assembled per-sample contig calls — run
  viRNAtrap's read-to-contig pipeline first (map to hg19/phiX, then virnatrap-predict)
  before survival testing.
- The analysis targets canonical exogenous cancer viruses (e.g. HPV, HBV) rather than
  endogenous retroviruses; the curated intact-gene HERV database and dichotomization
  are specific to HERV-K/HERV-H.
- Cancer types outside the four tested cohorts (COAD, LUSC, LUAD, LIHC) are used without
  re-validating the survival association, which was reported only for those cohorts.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3308
tools:
- name: viRNAtrap
  role: Generates the per-sample predicted viral contigs from unmapped RNAseq reads
    (Tensorflow model scores + contig assembly) whose HERV-K/HERV-H calls define the
    positive/negative survival groups
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: blastn
  role: Compares assembled viral contigs to curated viral databases (including the
    curated functional-HERV database) to classify contigs as HERV-K/HERV-H
- name: keras
  role: Backend for the Tensorflow deep learning model in viRNAtrap that scores 48
    bp unmapped reads as viral prior to contig assembly
  repo: https://github.com/AuslanderLab/virnatrap.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:f0b4020434e6de9c224c151f3fc06a95a5c03d71a6be179fe09bde5b9ac0d935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - blastn
  - keras
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# herv-expression-survival-association

## Summary

Tests whether the presence of HERV-K/HERV-H transcripts (from viRNAtrap-predicted viral contigs) in TCGA tumor samples is associated with patients' overall survival, and whether HERV-positive samples are enriched among driver-gene mutants. It links the paper's curated functional-HERV database to Kaplan-Meier/log-rank survival statistics and hypergeometric mutation-enrichment tests.

## When to use

Use when per-sample viral/HERV contig calls from RNAseq (e.g. viRNAtrap FASTA contigs) must be dichotomized into HERV-positive vs HERV-negative groups and tested against TCGA overall survival or somatic driver mutations.

## When NOT to use

- The input is raw RNAseq FASTQ rather than assembled per-sample contig calls — run viRNAtrap's read-to-contig pipeline first (map to hg19/phiX, then virnatrap-predict) before survival testing.
- The analysis targets canonical exogenous cancer viruses (e.g. HPV, HBV) rather than endogenous retroviruses; the curated intact-gene HERV database and dichotomization are specific to HERV-K/HERV-H.
- Cancer types outside the four tested cohorts (COAD, LUSC, LUAD, LIHC) are used without re-validating the survival association, which was reported only for those cohorts.

## Inputs

- Per-sample HERV contig calls (HERV-K/HERV-H-positive samples) from viRNAtrap-predicted viral contigs
- TCGA clinical overall survival data for COAD, LUSC, LUAD, and LIHC cohorts
- TCGA somatic mutation calls (TP53, KRAS, ARID1A, PTEN status per sample)
- Curated database of HERVs with intact retroviral genes in the human genome (used to classify assembled contigs)

## Outputs

- Kaplan-Meier overall survival curves for HERV-positive vs HERV-negative groups per cancer type
- Log-rank test p-values comparing HERV-positive vs HERV-negative survival per cancer type (COAD, LUSC, LUAD, LIHC)
- Hypergeometric enrichment p-values per driver gene (TP53, KRAS, ARID1A, PTEN)
- A single CSV results table combining log-rank and hypergeometric p-values

## How to apply

Load per-sample HERV-K/HERV-H contig calls, derived from viRNAtrap-predicted viral contigs assembled from unmapped TCGA RNAseq reads, together with TCGA clinical survival data for the COAD, LUSC, LUAD, and LIHC cohorts. Dichotomize patients within each cancer type into HERV-positive and HERV-negative groups based on the presence of HERV-K/HERV-H contigs in the corresponding sample. Fit Kaplan-Meier overall survival estimators per group with lifelines and compute log-rank test p-values comparing HERV-positive versus HERV-negative patients within each of the four cancer types. Separately, load TCGA somatic mutation calls, label each sample as mutated or wild-type for TP53, KRAS, ARID1A, and PTEN, and use scipy.stats.hypergeom to test enrichment of HERV-positive samples among mutant samples for each gene. Write the per-cancer-type log-rank p-values and per-gene hypergeometric enrichment p-values into a single CSV results table.

## Related tools

- **viRNAtrap** (Generates the per-sample predicted viral contigs from unmapped RNAseq reads (Tensorflow model scores + contig assembly) whose HERV-K/HERV-H calls define the positive/negative survival groups) — https://github.com/AuslanderLab/virnatrap.git
- **blastn** (Compares assembled viral contigs to curated viral databases (including the curated functional-HERV database) to classify contigs as HERV-K/HERV-H)
- **keras** (Backend for the Tensorflow deep learning model in viRNAtrap that scores 48 bp unmapped reads as viral prior to contig assembly) — https://github.com/AuslanderLab/virnatrap.git

## Examples

```
from lifelines import KaplanMeierFitter; from lifelines.statistics import logrank_test; from scipy.stats import hypergeom
```

## Evaluation signals

- HERV-K/HERV-H-positive patients must show significantly lower overall survival (log-rank p < 0.05) in COAD, LUSC, LUAD, and LIHC
- The HERV-positive group must show significant hypergeometric enrichment among TP53, KRAS, ARID1A, and PTEN mutant samples
- Each input FASTQ maps one-to-one to an output contig file; samples with zero predicted viral contigs yield empty (HERV-negative) call files
- Dichotomization must place every sample in exactly one group per cancer type, and group sizes must match the number of samples with/without HERV contigs
- The results CSV must contain one log-rank p-value per cancer type and one hypergeometric p-value per driver gene, covering all 4 + 4 = 8 tests

## Limitations

- Sensitivity is only guaranteed when more than five viral reads are present in a sample, so low-expression HERV samples may be misclassified as HERV-negative
- Reads mapped to hg19 or the frequent phiX contaminant are removed before contig detection, so HERV contigs depend on the quality of the user's independent read-mapping step
- Not all laboratory contaminants of viruses that may have infected samples can be modeled, which can confound viral-presence calls
- viRNAtrap predictions for sequences from organisms other than human/viral are not defined by the current models

## Evidence

- [abstract] HERV survival association finding: "disclose human endogenous viruses whose expression is associated with poor overall survival"
- [intro] HERV database curation and survival analysis scope: "we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers"
- [discussion] Strict survival association: "we found that the expression of human endogenous viruses is strictly associated with poor survival rates"
- [readme] Unmapped-read viral scoring step underlying the contig calls: "Use the Tensorflow model to predict which of the unmapped reads are viral; this is performed by the included software."
