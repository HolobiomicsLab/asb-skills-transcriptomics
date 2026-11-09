---
name: roc-auc-curve-comparison
description: Use when two or more RNA editing detection tools (e.g. REDInet, EditPredict,
  RED-ML, RDDpred) must be compared head-to-head on the same labeled site set, and
  per-tool ROC curves, AUC, or FNR are the requested performance metrics.
when_to_use_negative:
- The task is single-tool detection of editing events on new data rather than a controlled
  comparison of multiple classifiers on one labeled benchmark set
- The site set lacks reliable positive/negative labels (e.g. only predicted sites
  from REDIportal without DNAseq confirmation), making ROC-AUC and FNR undefined or
  biased
- Tools were run on different site subsets or with non-comparable inputs — AUCs computed
  on different denominators are not interpretable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: REDInet
  role: Temporal convolutional network classifier whose predictions set the reference
    AUC (0.98) in the comparison; requires tabix-indexed REDItools outTables
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: EditPredict
  role: Convolutional neural network trained on sequence contexts, run on the same
    bona fide site set per its guidelines to produce AUC 0.78
- name: RED-ML
  role: Logistic regression classifier trained on three broad classes of features,
    run on the identical site set to produce the lower AUC (0.72)
  repo: https://github.com/BGIRED/RED-ML
- name: RDDpred
  role: Tool specialized in distinguishing sequencing artifacts from genuine A-to-G
    substitutions; outputs binary decisions used to estimate FNR (~7%) rather than
    an AUC
  repo: https://github.com/vibbits/RDDpred
- name: REDItools
  role: Generates the RNAseq pileup outTables (variant/invariant sites) from sample
    BAMs that all classifiers consume
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: Python
  role: Implementation language for joining predictions to labels and computing ROC
    curves/AUC (e.g. via scikit-learn)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:0256480e01d985b587412e0dd10c632def1b9bd6c29a9fe034d299de950b55f9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet
  tools:
  - EditPredict
  - Python
  - RDDpred
  - RED-ML
  - REDInet
  - REDItools
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# roc-auc-curve-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple RNA editing classifiers by joining each tool's per-site predicted editing probabilities to a common bona fide ground truth set and computing per-tool ROC curves, ROC-AUC, and (for label-only tools like RDDpred) false negative rate. Used in the REDInet paper to show REDInet (AUC 0.98) outperforms EditPredict (0.78) and RED-ML (0.72) on the HEK293T KO-WT dataset.

## When to use

Use when two or more RNA editing detection tools (e.g. REDInet, EditPredict, RED-ML, RDDpred) must be compared head-to-head on the same labeled site set, and per-tool ROC curves, AUC, or FNR are the requested performance metrics.

## When NOT to use

- The task is single-tool detection of editing events on new data rather than a controlled comparison of multiple classifiers on one labeled benchmark set
- The site set lacks reliable positive/negative labels (e.g. only predicted sites from REDIportal without DNAseq confirmation), making ROC-AUC and FNR undefined or biased
- Tools were run on different site subsets or with non-comparable inputs — AUCs computed on different denominators are not interpretable

## Inputs

- Ground truth site set with genomic positions and positive/negative labels (HEK293T KO-WT bona fide sites)
- Per-tool predicted editing probabilities or binary decisions on the same sites (EditPredict, RED-ML, RDDpred, REDInet outputs)
- Sample BAM files (RNAseq with paired DNAseq/WGS or WES) processed through REDItools pileup outTables, bgzip-compressed and tabix-indexed

## Outputs

- Per-tool ROC curves and ROC-AUC values on the shared site set
- False negative rate for tools emitting only binary per-site decisions (e.g. RDDpred ~7% vs REDInet 2%)
- Joined per-site table of tool predictions vs ground truth labels by genomic position

## How to apply

Build a single ground truth site set — here the HEK293T KO-WT bona fide set with positives (A-to-G evidence in RNAseq, no base change in DNAseq, present in REDIportal, ≥50 RNAseq and ≥10 DNAseq reads) and negatives (A-to-G in both RNAseq and DNAseq, RNAseq A-to-G rate ≥0.4, DNAseq rate within 5% of the RNAseq rate) — and run every classifier on these identical sites according to each program's own published guidelines. Collect per-site predicted editing probabilities from each tool and join them to the positive/negative labels by genomic position. Compute ROC curves and ROC-AUC per tool with a standard library (e.g. scikit-learn); for tools that output only binary decisions rather than probabilities, such as RDDpred, estimate the false negative rate on the true editing positives instead. Validate by checking the reproduced ordering matches the reported one: REDInet AUC highest (0.98), EditPredict 0.78, RED-ML 0.72, RDDpred FNR ~7% versus 2% for REDInet.

## Related tools

- **REDInet** (Temporal convolutional network classifier whose predictions set the reference AUC (0.98) in the comparison; requires tabix-indexed REDItools outTables) — https://github.com/BioinfoUNIBA/REDInet
- **EditPredict** (Convolutional neural network trained on sequence contexts, run on the same bona fide site set per its guidelines to produce AUC 0.78)
- **RED-ML** (Logistic regression classifier trained on three broad classes of features, run on the identical site set to produce the lower AUC (0.72)) — https://github.com/BGIRED/RED-ML
- **RDDpred** (Tool specialized in distinguishing sequencing artifacts from genuine A-to-G substitutions; outputs binary decisions used to estimate FNR (~7%) rather than an AUC) — https://github.com/vibbits/RDDpred
- **REDItools** (Generates the RNAseq pileup outTables (variant/invariant sites) from sample BAMs that all classifiers consume) — https://github.com/BioinfoUNIBA/REDItools
- **Python** (Implementation language for joining predictions to labels and computing ROC curves/AUC (e.g. via scikit-learn))

## Examples

```
git clone https://github.com/vibbits/RDDpred.git && docker build -t rddpred-github:1.1.4 . && docker run rddpred-github:1.1.4 python RDDpred.py
```

## Evaluation signals

- Reproduced AUC ordering matches the article: REDInet (0.98) > EditPredict (0.78) > RED-ML (0.72)
- RDDpred false negative rate on true editing positives is approximately 7%, versus ~2% for REDInet
- Every tool's predictions were computed on the same set of bona fide HEK293T KO-WT sites, per each program's own guidelines
- Joined per-site table covers all ground truth positions with no missing labels after position-keyed join
- ROC curves are monotone and AUC falls in [0,1] for every tool with probability-like outputs

## Limitations

- RDDpred emits binary decisions rather than calibrated probabilities, so only FNR (not ROC-AUC) could be estimated for it in the paper's comparison
- Complete REDInet architecture description and performance metric formulas are deferred to Supplementary Methods not included in the main text
- Ground truth datasets are said to be in 'our GitHub repository' without a specific URL or accession in the article
- Benchmark quality depends on the ground truth construction thresholds (50 RNAseq/10 DNAseq read support, A-to-G rate ≥0.4, DNAseq rate within 5% of RNAseq rate), which constrain which sites enter the comparison

## Evidence
