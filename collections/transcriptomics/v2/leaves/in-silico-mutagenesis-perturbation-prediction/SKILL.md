---
name: in-silico-mutagenesis-perturbation-prediction
description: Use when enhancer–gene assignments must be validated against CRISPR perturbation
  measurements and a sequence-model counterfactual (predicted expression change after
  enhancer removal) is needed as an alternative to activity-by-contact scoring.
when_to_use_negative:
- Candidate enhancer–gene pairs exceed the 96 kb distance cap at which the Enformer
  ISM interaction scores are defined
- The goal is TF-affinity-based gene scoring or single-cell regulatory analysis with
  STARE/GAZE rather than perturbation-based benchmarking of enhancer–gene assignment
- No CRISPR perturbation ground truth is available to label positives, so ROC/PR comparison
  of scoring methods is not possible
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: Enformer
  role: Sequence model whose in-silico mutagenesis predictions (expression change
    at target gene after 2-kb enhancer replacement, averaged over 25 shuffles) quantify
    enhancer–gene interaction strength
- name: STARE / stare-abc
  role: Produces the adapted ABC (gABC) scores on the same candidate interactions
    for head-to-head comparison against the ISM scores
  repo: https://github.com/schulzlab/STARE
- name: pROC
  role: Computes precision–recall curves and ROC AUC for both scoring methods on each
    validation dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:dcf1c7f9aae2cb9ede009d6ce4f0644c358089ee73e1b85cf10587a69fe04203
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Enformer
  - STARE
  - STARE / stare-abc
  - pROC
  - stare-abc
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# in-silico-mutagenesis-perturbation-prediction

## Summary

Quantify enhancer–gene interaction strength with a sequence-based Enformer in-silico mutagenesis (ISM) score, replacing/shuffling each candidate enhancer and reading out the predicted expression change at the target gene, to benchmark against the adapted ABC (gABC) score on CRISPRi perturbation data.

## When to use

Use when enhancer–gene assignments must be validated against CRISPR perturbation measurements and a sequence-model counterfactual (predicted expression change after enhancer removal) is needed as an alternative to activity-by-contact scoring.

## When NOT to use

- Candidate enhancer–gene pairs exceed the 96 kb distance cap at which the Enformer ISM interaction scores are defined
- The goal is TF-affinity-based gene scoring or single-cell regulatory analysis with STARE/GAZE rather than perturbation-based benchmarking of enhancer–gene assignment
- No CRISPR perturbation ground truth is available to label positives, so ROC/PR comparison of scoring methods is not possible

## Inputs

- Candidate-region BED file with enhancer activity values (activity column passed via -n)
- Gencode GTF gene annotation
- Chromatin contact folder (e.g. normalized Hi-C matrices, Fulco et al. 2019 average hg19 or Gschwind et al. 2023 hg38 ENCODE ENCFF134PUN)
- CRISPR perturbation validation measurements (for positive-label assignment)
- Genome sequence for Enformer input (2-kb enhancer replacements / shuffles)

## Outputs

- Enformer ISM interaction-strength scores per enhancer–gene pair (interactions <= 96 kb)
- gABC-scored enhancer–gene interactions from STARE_ABCpp
- Precision–recall curves and ROC AUC values per method per dataset (pROC)
- Comparison table gabc_enformer_pr_roc_comparison.csv

## How to apply

First run STARE_ABCpp on each CRISPRi validation dataset (candidate-region bed file, activity column via -n, Gencode gtf via -a, chromatin contact folder via -f, default 5 MB window and 0.02 ABC-score cutoff) to obtain gABC-scored interactions. For the same candidate set, compute Enformer ISM scores only for interactions within 96 kb: replace each enhancer with a neutral 2-kb sequence, average the predicted expression change at the target gene over 25 shuffle iterations, and take that change as the interaction strength. Rank the shared candidate interactions by gABC score and by Enformer ISM score, label positives from the CRISPR perturbation measurements, then compute precision–recall curves and ROC AUC for both methods per dataset with the pROC package. The rationale is that ISM provides a purely sequence-based counterfactual of enhancer function, whereas the adapted ABC score adapts enhancer activity according to the number of contacts to candidate target genes.

## Related tools

- **Enformer** (Sequence model whose in-silico mutagenesis predictions (expression change at target gene after 2-kb enhancer replacement, averaged over 25 shuffles) quantify enhancer–gene interaction strength)
- **STARE / stare-abc** (Produces the adapted ABC (gABC) scores on the same candidate interactions for head-to-head comparison against the ISM scores) — https://github.com/schulzlab/STARE
- **pROC** (Computes precision–recall curves and ROC AUC for both scoring methods on each validation dataset)

## Examples

```
STARE_ABCpp -b candidates.bed -n activity -a gencode.gtf -f contacts/  # default 5MB window, 0.02 ABC-score cutoff; then compute Enformer ISM scores for interactions <= 96 kb
```

## Evaluation signals

- ISM scores are averaged over 25 shuffle iterations per enhancer to stabilize the predicted expression change
- Only candidate interactions <= 96 kb carry an Enformer ISM score; longer-range pairs are excluded from the comparison
- ROC AUC values are computed per dataset with pROC and assembled into gabc_enformer_pr_roc_comparison.csv
- Expected outcome from the source card: gABC attains higher accuracy and significantly higher ROC AUCs than Enformer ISM on all three CRISPRi validation datasets
- gABC runs use the default 5 MB window and 0.02 ABC-score cutoff; deviations must be documented

## Limitations

- Enformer ISM is restricted to interactions within 96 kb, whereas the ABC model uses a default 5 MB window around the TSS, so the compared candidate sets differ in range
- ISM scores are sequence-model predictions of expression change, not measured perturbation effects, and require 25 shuffle iterations to reduce variance
- The comparison depends on CRISPR perturbation validation measurements as ground truth; datasets lacking such measurements cannot be benchmarked this way

## Evidence

- [other] Compute Enformer in-silico mutagenesis scores for the same candidate interactions (interactions <= 96 kb) by replacing each enhancer with a neutral 2-kb sequence: [evidence span withheld — anchor status: not_found]
- [abstract] ABC score accuracy via adapted activity: "We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes"
- [intro] STARE framework definition: "STARE is a framework that combines two tasks, which can also work independently"
- [readme] README STARE tasks: " - score enhancer-gene interactions with the Activity-By-Contact (ABC)"
