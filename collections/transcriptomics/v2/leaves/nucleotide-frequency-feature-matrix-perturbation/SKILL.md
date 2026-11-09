---
name: nucleotide-frequency-feature-matrix-perturbation
description: Use when stress-testing a sequence-window classifier (e.g. REDInet) for
  robustness to incomplete RNAseq coverage, by injecting reference-genome substitutions
  into feature matrices and re-scoring predictions against ground truth.
when_to_use_negative:
- Feature matrices already contain imputed/genomic values or are known to be fully
  covered — perturbation would not represent genuine missing-data conditions
- Evaluating REDInet on unperturbed benchmark performance (A549, HEK293T KO-WT/KO-OE
  ground truth sets) rather than robustness to missing flanking data
- Input is raw BAM/REDItools outTable rather than precomputed 8x101 feature matrices
  — run the REDInet pipeline first to generate feature vectors
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: REDInet
  role: Pretrained TCN classifier (REDInet.h5) run on the degraded 8x101 matrices
    via REDInet_Inference.py to obtain per-site SNP/editing probabilities
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: TensorFlow
  role: Deep learning framework used to load and run the REDInet.h5 model for inference
    on perturbed feature matrices
- name: NumPy
  role: Random sampling to select and replace RNAseq-derived nucleotide frequencies
    with genomic reference frequencies at >30% of flanking positions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:a0b342e2eb8d9374533cf13133c6d8f0f9b02c936c3eb8e333da0c1f743c326b
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
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nucleotide-frequency-feature-matrix-perturbation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates missing RNAseq coverage in 8x101 nucleotide-frequency feature matrices by replacing RNAseq-derived frequencies with corresponding genomic reference frequencies at >30% of the 101 flanking positions, then re-runs REDInet inference to quantify false-positive/false-negative rates under degraded input.

## When to use

Use when stress-testing a sequence-window classifier (e.g. REDInet) for robustness to incomplete RNAseq coverage, by injecting reference-genome substitutions into feature matrices and re-scoring predictions against ground truth.

## When NOT to use

- Feature matrices already contain imputed/genomic values or are known to be fully covered — perturbation would not represent genuine missing-data conditions
- Evaluating REDInet on unperturbed benchmark performance (A549, HEK293T KO-WT/KO-OE ground truth sets) rather than robustness to missing flanking data
- Input is raw BAM/REDItools outTable rather than precomputed 8x101 feature matrices — run the REDInet pipeline first to generate feature vectors

## Inputs

- Ground truth test-set feature vectors: 8x101 nucleotide-frequency matrices centered on putative A-to-I editing sites (REDInet Package/Data directory)
- Ground truth binary labels (editing vs SNP) for each test site
- Genomic reference nucleotide frequencies for the same 101-base windows (e.g. from GRCh37/GRCh38 primary assembly FASTA)
- Pretrained REDInet model file (REDInet.h5)

## Outputs

- missing_data_fp_fn_rates.csv with per-site predictions and overall false-positive and false-negative rates on perturbed inputs
- Degraded 8x101 feature matrices with >30% flanking positions replaced by genomic reference frequencies

## How to apply

Load ground truth test-set feature vectors — 8x101 nucleotide-frequency matrices centered on each putative editing site — from the REDInet package's Data directory. Using NumPy random sampling, select more than 30% of the 101 positions in each flanking region and replace the RNAseq-derived nucleotide frequencies with the corresponding genomic reference frequencies, mimicking missing RNAseq coverage. Run inference on the degraded matrices with the pretrained REDInet temporal convolutional network model (REDInet.h5) via the REDInet_Inference.py script in TensorFlow to obtain per-site SNP/editing probabilities. Apply softmax to obtain y_hat class predictions and compare against ground truth labels to count misclassified positives and negatives. Compute false-positive and false-negative rates and write them, with per-site predictions, to missing_data_fp_fn_rates.csv; the experiment replicates the paper's Fig. S4 robustness analysis.

## Related tools

- **REDInet** (Pretrained TCN classifier (REDInet.h5) run on the degraded 8x101 matrices via REDInet_Inference.py to obtain per-site SNP/editing probabilities) — https://github.com/BioinfoUNIBA/REDInet.git
- **TensorFlow** (Deep learning framework used to load and run the REDInet.h5 model for inference on perturbed feature matrices)
- **NumPy** (Random sampling to select and replace RNAseq-derived nucleotide frequencies with genomic reference frequencies at >30% of flanking positions)

## Examples

```
python3 REDInet_Inference.py --I <REDItools_outTable_folderpath> --O <output_folder> --A GRCh37 --N <outTable.gz>
```

## Evaluation signals

- False-positive rate on perturbed inputs remains below 10%
- False-negative rate on perturbed inputs remains below 10%
- Perturbation coverage: >30% of the 101 positions per flanking region actually replaced by genomic reference frequencies (verifiable by counting position mismatches between original and degraded matrices)
- Per-site y_hat classes (via softmax on SNP/editing probabilities) match expected prediction distribution against ground truth labels
- missing_data_fp_fn_rates.csv is written with both per-site predictions and aggregate FP/FN rates, reproducing Fig. S4 behavior

## Limitations

- REDInet's inference relies on Tabix-indexed REDItools output tables; feature matrices must come from fully characterized sites with complete 101-base windows for unperturbed baselines
- Complete REDInet architecture and feature rescaling details are deferred to Supplementary Material, so the perturbation pipeline may not expose all internal feature normalizations
- Ground truth datasets are referenced only as being in 'our GitHub repository' without a specific URL or accession, complicating exact reproduction

## Evidence
