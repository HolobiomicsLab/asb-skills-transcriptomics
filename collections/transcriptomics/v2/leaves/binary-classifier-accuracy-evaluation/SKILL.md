---
name: binary-classifier-accuracy-evaluation
description: Use when a binary classifier has produced per-site predictions on a labeled,
  held-out dataset and you must quantify performance as mean accuracy, AUC, specificity,
  and precision against gold-standard positives/negatives.
when_to_use_negative:
- The data are part of the GTEx training/validation/test split (accuracy there, >99%,
  reflects training performance and does not measure generalization to unseen samples)
- Predictions come from a multi-class or regression model rather than binary SNP-vs-editing-site
  classification (e.g. per-site probabilities without y_hat thresholds)
- Sites fail the coverage thresholds (minimum 50 reads, 3 G substitutions, A-to-G
  frequency 0.01) and were discarded before classification, so accuracy on them cannot
  be computed
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: REDInet
  role: Pretrained TCN classifier (REDInet.h5) whose per-site y_hat/probability outputs
    are scored against ground truth labels
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: REDItools
  role: Generates the low-stringency outTables that REDInet consumes; its candidate
    sites are the units being classified and scored
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools v3
  role: Alternative faster implementation used to produce pileup profiles for building
    ground truth sets
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: EditPredict
  role: Comparison classifier (CNN on sequence contexts, AUC 0.51 on U87) evaluated
    on the same bona fide site set
- name: RED-ML
  role: Logistic-regression comparator tool run on the same bona fide site set following
    its own guidelines
  repo: https://github.com/BGIRED/RED-ML
- name: RDDpred
  role: Comparator tool specialized in distinguishing sequencing artifacts from genuine
    AG substitutions
  repo: https://github.com/vibbits/RDDpred
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:dc9fb87db42f21225b7a9606c715cd85be58baec7ba01468e5295eb02d7b4856
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet.git
  tools:
  - EditPredict
  - Python
  - RDDpred
  - RED-ML
  - REDInet
  - REDItools
  - REDItools v3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# binary-classifier-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate a binary RNA-editing classifier (e.g. REDInet's per-site y_hat calls of SNP=0 vs editing site=1) against curated ground-truth labels by computing accuracy, AUC, specificity, and precision on held-out site sets such as A549 (>13,000 sites) or U87 (>2,800 sites).

## When to use

Use when a binary classifier has produced per-site predictions on a labeled, held-out dataset and you must quantify performance as mean accuracy, AUC, specificity, and precision against gold-standard positives/negatives.

## When NOT to use

- The data are part of the GTEx training/validation/test split (accuracy there, >99%, reflects training performance and does not measure generalization to unseen samples)
- Predictions come from a multi-class or regression model rather than binary SNP-vs-editing-site classification (e.g. per-site probabilities without y_hat thresholds)
- Sites fail the coverage thresholds (minimum 50 reads, 3 G substitutions, A-to-G frequency 0.01) and were discarded before classification, so accuracy on them cannot be computed

## Inputs

- REDInet per-site predictions file (*.predictions.tsv with y_hat, snp_proba, ed_proba) from tabix-indexed REDItools outTables
- Ground-truth label sets (positives: A-to-I in WT/none in ADAR-silenced; negatives: A-to-G in both RNAseq and DNAseq, REDIportal-supported)
- Ground truth datasets available in the REDInet GitHub repository

## Outputs

- Mean classification accuracy on the held-out ground truth set
- AUC, accuracy, specificity, and precision metrics (e.g. AUC 0.99, accuracy >97% on U87)
- Per-site label-versus-prediction comparison table supporting figures/tables such as Fig. 2b and Table S2

## How to apply

Collect per-site predictions (REDInet emits y_hat, snp_proba, and ed_proba in *.predictions.tsv) from tabix-indexed REDItools outTables filtered with --C 50 minimum coverage, --M 3 minimum A-to-G substitutions, and --F 0.01 minimum A-to-G frequency over 101-nt windows. Build or load an independent ground truth set not seen in training — positives with A-to-I evidence in WT samples but not in ADAR-silenced/KO samples, and negatives with A-to-G evidence in both RNAseq and DNAseq — requiring 50 RNAseq reads and 10 DNAseq reads of support, with RNAseq A-to-G rate >= 0.4 and DNA rate within 5% for negatives. Map each prediction to its ground-truth label and compute mean classification accuracy; where probabilities are available, compute AUC, specificity, and precision using the formulas given in the Supplementary Methods. Compare against reported benchmarks (e.g. >96% mean accuracy on A549, AUC 0.99 on U87) and, optionally, against competing tools (EditPredict, RED-ML, RDDpred) run on the same bona fide site set per each program's guidelines.

## Related tools

- **REDInet** (Pretrained TCN classifier (REDInet.h5) whose per-site y_hat/probability outputs are scored against ground truth labels) — https://github.com/BioinfoUNIBA/REDInet.git
- **REDItools** (Generates the low-stringency outTables that REDInet consumes; its candidate sites are the units being classified and scored) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools v3** (Alternative faster implementation used to produce pileup profiles for building ground truth sets) — https://github.com/BioinfoUNIBA/REDItools3
- **EditPredict** (Comparison classifier (CNN on sequence contexts, AUC 0.51 on U87) evaluated on the same bona fide site set)
- **RED-ML** (Logistic-regression comparator tool run on the same bona fide site set following its own guidelines) — https://github.com/BGIRED/RED-ML
- **RDDpred** (Comparator tool specialized in distinguishing sequencing artifacts from genuine AG substitutions) — https://github.com/vibbits/RDDpred

## Examples

```
python3 REDInet_Inference_light_ver.py -r outTable_SRR12492045.gz -o U87_run -c 50 -s 3 -f 0.01
```

## Evaluation signals

- Mean accuracy on the >13,000-site A549 ground truth set reaches the reported >96% (Fig. 2b and Table S2)
- U87 ground truth evaluation reproduces AUC 0.99 with accuracy >97%, specificity >96%, and precision >97%
- HEK293T KO-WT (>20,000 events) and KO-OE (>35,000 sites) evaluations yield accuracy >95%, specificity >93%, precision >96%
- Training/validation/test accuracy is >99% with no under- or over-fitting, confirming a stable reference ceiling before held-out evaluation
- Every scored position passed REDInet's own filters (minimum coverage 50, minimum 3 A-to-G substitutions, minimum frequency 0.01, fully covered 101-base window) so discards are logged in the *_discarded.txt file rather than silently scored

## Limitations

- Complete REDInet architecture and the performance-metric formulas are only in the Supplementary Material, not in the main text
- Ground truth datasets are said to be in 'our GitHub repository' without a specific URL or accession in the article
- Ground truth sets were built from only A549 and HEK293T cell lines, not included in the GTEx atlas, so generalization beyond these contexts (e.g. TCGA disease data) is untested here
- REDInet is specific to A-to-G (A-to-I) substitutions and 101-nt window features; other editing types or shorter coverage windows are out of scope

## Evidence

- [results] REDInet achieved an accuracy of >95% on both KO-WT (Fig. 2c) and KO-OE datasets, with a specificity of >93% and precision of >96%: "REDInet achieved an accuracy of >95% on both KO-WT (Fig. 2c) and KO-OE datasets, with a specificity of >93% and precision of >96%"
- [results] accuracy >99% on training/validation/test: "was initially applied on training, validation, and test sets, achieving accuracy values >99% without under- or over-fitting"
- [discussion] metric formulas in supplementary methods: "according to formulas detailed in the Supplementary Methods"
- [readme] REDInet filtering thresholds before prediction: "Genomic positions are filtered on the basis of minimum base coverage, minimum A to G substitution rate and minimum number of guanosine in place of adenines."
- [results] comparator evaluation on shared bona fide sites: "the analyses were carried out on the same set of bona fide sites from HEK293T KO and WT samples, according the guidelines of each specific program"
