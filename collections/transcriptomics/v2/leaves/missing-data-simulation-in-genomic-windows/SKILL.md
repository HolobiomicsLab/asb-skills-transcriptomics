---
name: missing-data-simulation-in-genomic-windows
description: Use when evaluating robustness of a 101-base-window RNA editing classifier
  to missing RNAseq coverage — e.g. quantifying FP/FN rates after randomly swapping
  RNAseq nucleotide frequencies for genomic reference frequencies in >30% of flanking
  positions.
when_to_use_negative:
- Input is real RNAseq data whose 101-base windows are already fully covered — perturbation
  would misrepresent actual missing-data behavior
- You need to impute missing values in production inference rather than measure robustness
  — REDInet_Inference.py already ships reference-genome imputation as a BETA FEATURE,
  so use that instead of a simulation harness
- Ground truth labels are unavailable — FP/FN rates cannot be computed against unlabeled
  predictions
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: REDInet
  role: Pretrained TCN classifier (REDInet.h5) applied to degraded 8×101 feature matrices
    via REDInet_Inference.py
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: NumPy
  role: Random sampling of the >30% of 101 flanking positions whose RNAseq frequencies
    are replaced with genomic reference frequencies
- name: TensorFlow
  role: Loads REDInet.h5 and runs inference to obtain per-site SNP/editing probabilities
    (softmax y_hat)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:5555a7ec2ea87b26c03a155b607fe4dfa15946932ceefc786fcaba9f9d9a6c08
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
  - NumPy
  - Python
  - REDInet
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# missing-data-simulation-in-genomic-windows

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stress-test the REDInet A-to-I RNA editing classifier by simulating missing RNAseq coverage: RNAseq-derived nucleotide frequencies are randomly replaced with genomic reference frequencies across flanking positions of 8×101 feature matrices, and false-positive/false-negative rates are measured on the degraded inputs.

## When to use

Use when evaluating robustness of a 101-base-window RNA editing classifier to missing RNAseq coverage — e.g. quantifying FP/FN rates after randomly swapping RNAseq nucleotide frequencies for genomic reference frequencies in >30% of flanking positions.

## When NOT to use

- Input is real RNAseq data whose 101-base windows are already fully covered — perturbation would misrepresent actual missing-data behavior
- You need to impute missing values in production inference rather than measure robustness — REDInet_Inference.py already ships reference-genome imputation as a BETA FEATURE, so use that instead of a simulation harness
- Ground truth labels are unavailable — FP/FN rates cannot be computed against unlabeled predictions

## Inputs

- Ground truth test-set feature vectors: 8×101 nucleotide-frequency matrices centered on putative A-to-I editing sites (from the REDInet Package/Data directory)
- Genomic reference nucleotide frequencies (GRCh37 or GRCh38 primary assembly, e.g. from Utilities/*.genome.fa)
- Pretrained TCN model REDInet.h5
- Ground truth labels (editing vs SNP) per site

## Outputs

- missing_data_fp_fn_rates.csv with per-site predictions and aggregate false-positive and false-negative rates
- FP/FN rate summary under the >30% missing-flanking-data condition (compare to Fig. S4)

## How to apply

Load the ground truth test-set feature vectors — 8×101 nucleotide-frequency matrices centered on each putative editing site — from the REDInet package's Data directory. Use NumPy random sampling to select and replace RNAseq-derived nucleotide frequencies with the corresponding genomic reference frequencies at >30% of the 101 positions in each flanking region, mimicking sites where the 101-base window is not fully covered by RNAseq reads. Run inference on the perturbed matrices with the pretrained REDInet temporal convolutional network (REDInet.h5) via TensorFlow using the REDInet_Inference.py script to obtain per-site SNP/editing probabilities. Compare predicted classes (y_hat via softmax on SNP/editing probabilities) against ground truth labels to count misclassified positives and negatives, then compute false-positive and false-negative rates. Write the rates together with per-site predictions to missing_data_fp_fn_rates.csv; the experiment reproduces the paper's Fig. S4 robustness claim if both rates stay below 10%.

## Related tools

- **REDInet** (Pretrained TCN classifier (REDInet.h5) applied to degraded 8×101 feature matrices via REDInet_Inference.py) — https://github.com/BioinfoUNIBA/REDInet.git
- **NumPy** (Random sampling of the >30% of 101 flanking positions whose RNAseq frequencies are replaced with genomic reference frequencies)
- **TensorFlow** (Loads REDInet.h5 and runs inference to obtain per-site SNP/editing probabilities (softmax y_hat))

## Examples

```
python3 REDInet_Inference.py --I <REDItools_outTable_folderpath> --O <output_folder> --A GRCh37 --C 50 --M 3 --N outTable.gz
```

## Evaluation signals

- False-positive rate < 10% on perturbed inputs
- False-negative rate < 10% on perturbed inputs
- More than 30% of the 101 positions per flanking region actually replaced (verify substitution counts against the simulated mask)
- Replaced positions match genomic reference frequencies for the corresponding assembly (GRCh37/GRCh38)
- Result directionally consistent with Fig. S4 robustness reported in the paper

## Limitations

- The imputation of missing values in REDInet_Inference.py is flagged as a BETA FEATURE and is assembly-dependent (hg38 or hg19), so simulated missingness does not equal real pipeline imputation behavior
- REDInet further filters positions for missing nucleotides in 101-nucleotide regions, so heavily degraded windows may be discarded rather than classified
- Minimum coverage, minimum A-to-G substitution, and minimum guanosine-count filters apply before prediction, so perturbation experiments should preserve these thresholds (e.g. --C 50, --M 3)

## Evidence
