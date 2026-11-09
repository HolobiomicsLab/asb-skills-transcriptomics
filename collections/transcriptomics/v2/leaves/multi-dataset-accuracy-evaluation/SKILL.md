---
name: multi-dataset-accuracy-evaluation
description: Use when comparing a deep learning filter (e.g., HVLD) against classifier
  baselines across multiple paired-end RNA-seq datasets, with per-dataset and averaged
  accuracy as the target metrics.
when_to_use_negative:
- Data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads, since
  Anchored-fusion requires paired-end RNA-seq
- The user's input dataset is too small to retrain the HVLD filter; use the default
  pre-trained model parameters from GitHub instead
- Fusions not including the anchored target gene are of interest — the anchored search
  will not detect them
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Anchored-fusion
  role: Provides the HVLD deep learning filter and the 'data' folder with positive
    training samples and homologous gene file used to train and benchmark all classifiers
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: PyTorch
  role: Framework for training HVLD, HVL, bi-LSTM, Transformer, and MLP classifiers
- name: scikit-learn
  role: Implements the SVM baseline classifier on the same per-dataset splits
- name: Biopython
  role: Sequence handling during training-set construction
- name: NumPy
  role: Numerical processing of read features for classifier input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:8056b65e7e9c6056a32554145f3dfb05cc0b25959a50f3bf3b11a95678deb6d8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Biopython
  - NumPy
  - PyTorch
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# multi-dataset-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the HVLD chimeric-read classifier against machine learning and deep learning baselines (bi-LSTM, Transformer, MLP, SVM, and the no-distillation HVL variant) across six CCLE cancer cell line bRNA-seq datasets, reporting per-dataset and average accuracy.

## When to use

Use when comparing a deep learning filter (e.g., HVLD) against classifier baselines across multiple paired-end RNA-seq datasets, with per-dataset and averaged accuracy as the target metrics.

## When NOT to use

- Data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads, since Anchored-fusion requires paired-end RNA-seq
- The user's input dataset is too small to retrain the HVLD filter; use the default pre-trained model parameters from GitHub instead
- Fusions not including the anchored target gene are of interest — the anchored search will not detect them

## Inputs

- Six CCLE cancer cell line paired-end bRNA-seq FASTQ datasets (K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12; SRA:SRP186687)
- 3540 PCAWG fusion genes as positive training samples
- Artificial chimeric reads as negative training samples
- Homologous gene file from the 'data' folder of the Anchored-Fusion repository
- Target gene transcript FASTA and genome reference FASTA plus GTF annotation (hg38 or hg19)

## Outputs

- Per-dataset and average classification accuracy table for HVLD, HVL, bi-LSTM, Transformer, MLP, and SVM
- Retrained HVLD filter model file saved to the location given by '--model_file'

## How to apply

Build per-dataset training sets for the six CCLE cell lines (K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12) using PCAWG fusions as positive samples and artificial chimeric reads as negatives, removing negative samples containing the anchored gene and reads aligned to homologous genes. Train HVLD in PyTorch (>=1.10.2) with its hierarchical view learning and self-distillation loss, balancing the two loss types with the alpha hyperparameter, using the positive training samples and homologous gene file shipped in the 'data' folder. Train bi-LSTM, Transformer, and MLP in PyTorch and SVM with scikit-learn (>=1.3) on the same splits, plus the HVL variant without distillation. Evaluate all six models per dataset and compute per-dataset, average, max, median, and min accuracy; save the retrained HVLD filter via '--model_file' (default under the 'data' folder) and write the accuracy table to the output folder.

## Related tools

- **Anchored-fusion** (Provides the HVLD deep learning filter and the 'data' folder with positive training samples and homologous gene file used to train and benchmark all classifiers) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **PyTorch** (Framework for training HVLD, HVL, bi-LSTM, Transformer, and MLP classifiers)
- **scikit-learn** (Implements the SVM baseline classifier on the same per-dataset splits)
- **Biopython** (Sequence handling during training-set construction)
- **NumPy** (Numerical processing of read features for classifier input)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- HVLD must reach approximately 82.1% average accuracy across the six CCLE datasets (max 83.4%, median 82.1%, min 80.8%)
- HVLD should exceed the Transformer baseline by about 8.85% and the HVL no-distillation variant by about 0.75%
- HVLD accuracy must exceed bi-LSTM, MLP, and SVM baselines on the chimeric-read classification task in each dataset comparison
- The retrained HVLD filter file is written to the '--model_file' path (default under the 'data' folder) and the accuracy table is present in the output folder

## Limitations

- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified
- For small input datasets, users should rely on the default model parameters provided on GitHub rather than retrained HVLD parameters
- Training a per-dataset filter is time-consuming; skipping it via '--not_filter_false_positive' or using a pre-trained model via '--not_train_filter_model' saves time but may cause result deviations

## Evidence

- [results] HVLD outperformed bi-LSTM, Transformer, MLP, SVM, and the no-distillation HVL baseline on chimeric-read classification across the bRNA-seq datasets, achieving an average accuracy of 82.1% (max 83.4%, median 82.1%, min 80.8%).: [evidence span withheld — anchor status: not_found]
