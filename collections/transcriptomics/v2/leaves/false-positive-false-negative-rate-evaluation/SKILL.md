---
name: false-positive-false-negative-rate-evaluation
description: Use when you must estimate how many RNA editing positives are missed
  (FN) and negatives accepted (FP) by REDInet — e.g. validating robustness under degraded
  inputs like >30% missing flanking positions.
when_to_use_negative:
- When running de novo A-to-I editing detection on a new unlabeled RNAseq sample via
  REDInet_Inference.py — no ground-truth labels exist, so FP/FN rates cannot be computed
- When the input is a raw REDItools outTable without any ADAR KO-vs-WT (or KO-vs-OE)
  contrast that defines bona fide positives and negatives
- When reporting headline performance metrics (accuracy, specificity, precision) on
  the published benchmark datasets, which the paper derives from formulas in the Supplementary
  Methods rather than the perturbation protocol
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: REDInet
  role: Pretrained TCN classifier (REDInet.h5) whose per-site SNP/editing probabilities
    and y_hat classes are evaluated against ground truth; run via REDInet_Inference.py
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: TensorFlow
  role: Executes the temporal convolutional network inference and softmax over SNP/editing
    probabilities for the degraded feature matrices
- name: NumPy
  role: Random sampling of flanking positions at which RNAseq nucleotide frequencies
    are replaced by genomic reference frequencies to simulate missing data
- name: REDItools
  role: Produces the REDItools outTables from which the 101-base RNAseq windows and
    their feature vectors derive; output must be bgzip/tabix indexed
  repo: https://github.com/BioinfoUNIBA/REDItools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:345f038635c69761309a7a5464bfd62a8acddbdc30debd33c5164fe992588a1c
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
  - NumPy
  - Python
  - REDInet
  - REDItools
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# false-positive-false-negative-rate-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Protocol for quantifying REDInet misclassification: compare per-site y_hat classes (softmax on SNP/editing probabilities) against ground-truth positives/negatives to compute false-positive and false-negative rates, e.g. after simulating missing flanking-region RNAseq data.

## When to use

Use when you must estimate how many RNA editing positives are missed (FN) and negatives accepted (FP) by REDInet — e.g. validating robustness under degraded inputs like >30% missing flanking positions.

## When NOT to use

- When running de novo A-to-I editing detection on a new unlabeled RNAseq sample via REDInet_Inference.py — no ground-truth labels exist, so FP/FN rates cannot be computed
- When the input is a raw REDItools outTable without any ADAR KO-vs-WT (or KO-vs-OE) contrast that defines bona fide positives and negatives
- When reporting headline performance metrics (accuracy, specificity, precision) on the published benchmark datasets, which the paper derives from formulas in the Supplementary Methods rather than the perturbation protocol

## Inputs

- Ground-truth test-set feature vectors: 8 × 101 nucleotide-frequency matrices centered on each putative editing site (REDInet Package/Data directory)
- Ground-truth class labels distinguishing RNA editing positives from SNP negatives
- Pretrained REDInet temporal convolutional network model (REDInet.h5)
- Genomic reference nucleotide frequencies (GRCh37 or GRCh38) for simulating missing RNAseq coverage

## Outputs

- Per-site SNP/editing probabilities and predicted classes (y_hat via softmax)
- Counts of misclassified positives (false negatives) and misclassified negatives (false positives)
- Aggregate false-positive and false-negative rates written with per-site predictions to missing_data_fp_fn_rates.csv

## How to apply

Load the ground-truth test-set feature vectors (8 × 101 nucleotide-frequency matrices centered on each putative editing site) from the REDInet Package/Data directory, plus their labels: positives are sites with A-to-G evidence in RNAseq, no base change in DNAseq, and presence in REDIportal, while negatives show A-to-G evidence in both RNAseq and DNAseq. To test robustness, randomly replace RNAseq-derived nucleotide frequencies with the corresponding genomic reference frequencies at >30% of the 101 flanking positions using NumPy random sampling, leaving the center site intact. Run inference with the pretrained REDInet temporal convolutional network (REDInet.h5) via TensorFlow, applying a softmax to the SNP/editing probabilities to obtain per-site y_hat classes. Count misclassified positives (false negatives) and misclassified negatives (false positives), then write the rates together with per-site predictions to missing_data_fp_fn_rates.csv. Accept the evaluation only if both the false-positive and false-negative rates stay below 10%, reproducing the robustness reported in Fig. S4.

## Related tools

- **REDInet** (Pretrained TCN classifier (REDInet.h5) whose per-site SNP/editing probabilities and y_hat classes are evaluated against ground truth; run via REDInet_Inference.py) — https://github.com/BioinfoUNIBA/REDInet
- **TensorFlow** (Executes the temporal convolutional network inference and softmax over SNP/editing probabilities for the degraded feature matrices)
- **NumPy** (Random sampling of flanking positions at which RNAseq nucleotide frequencies are replaced by genomic reference frequencies to simulate missing data)
- **REDItools** (Produces the REDItools outTables from which the 101-base RNAseq windows and their feature vectors derive; output must be bgzip/tabix indexed) — https://github.com/BioinfoUNIBA/REDItools

## Examples

```
python3 ../REDInet/Package/Utilities/REDInet_Inference_light_ver.py -r outTable.gz -o missing_data_fp_fn_rates -c 50 -s 3 -f 0.01
```

## Evaluation signals

- Both false-positive and false-negative rates remain below 10% on the perturbed test set, reproducing the robustness reported in Fig. S4
- Predicted y_hat vector has exactly one class per test site and the same length as the ground-truth label vector
- In degraded matrices, RNAseq frequencies are replaced by genomic reference frequencies at >30% of the 101 flanking positions while the centered putative editing site is untouched
- missing_data_fp_fn_rates.csv contains both the per-site predictions and the aggregate FP/FN rates
- Baseline (unperturbed) FP/FN counts should be near zero, consistent with REDInet's reported >99% accuracy on training, validation, and test sets

## Limitations

- The formulas used for the performance metrics are deferred to the Supplementary Methods, which are not included in the main text
- Ground truth datasets are said to be available in 'our GitHub repository' without a specific URL or accession, complicating exact reproduction
- Evaluation depends on strict ground-truth construction: sites require 50 RNAseq reads and 10 DNAseq reads; negatives need RNAseq A-to-G rate >= 0.4 with DNAseq rate within 5% of the RNAseq rate
- Imputation of missing values in the released REDInet_Inference.py is described as a BETA feature in the README, so manual NumPy-based replacement is the more controlled route

## Evidence
