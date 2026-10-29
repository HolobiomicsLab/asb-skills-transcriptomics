---
name: self-distillation-deep-learning-model-training
description: Use when running Anchored-fusion with default filtering on paired-end
  RNA-seq FASTQ data and a retrained HVLD filter is desired, or when benchmarking
  HVLD against bi-LSTM, Transformer, MLP, SVM, and HVL baselines on chimeric-read
  classification.
when_to_use_negative:
- The input dataset is too small to retrain — use the default model parameters provided
  on GitHub instead
- Running Anchored-fusion with '--not_filter_false_positive' (time-saving mode) —
  no filter model is applied
- Data is not paired-end RNA-seq — Anchored-fusion cannot be used for single-end RNA-seq,
  whole-genome, or whole-exome sequencing data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Anchored-fusion
  role: Hosts the HVLD filter training; run via Anchored_Fusion.py with '--model_file'
    specifying the trained filter
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: PyTorch
  role: Framework for training HVLD, HVL, bi-LSTM, Transformer, and MLP classifiers
- name: scikit-learn
  role: Trains the SVM baseline classifier on the same chimeric-read splits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:87882fdbf757fbc3a2e77212791888e8f2a05a08306a47e7b0401bb0f1a22a31
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# self-distillation-deep-learning-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains the HVLD (hierarchical view learning and distillation) deep learning filter of Anchored-fusion per input RNA-seq dataset, using PCAWG fusions as positive samples and artificial chimeric reads as negatives, to distinguish true fusion reads from sequencing/PCR chimeras.

## When to use

Use when running Anchored-fusion with default filtering on paired-end RNA-seq FASTQ data and a retrained HVLD filter is desired, or when benchmarking HVLD against bi-LSTM, Transformer, MLP, SVM, and HVL baselines on chimeric-read classification.

## When NOT to use

- The input dataset is too small to retrain — use the default model parameters provided on GitHub instead
- Running Anchored-fusion with '--not_filter_false_positive' (time-saving mode) — no filter model is applied
- Data is not paired-end RNA-seq — Anchored-fusion cannot be used for single-end RNA-seq, whole-genome, or whole-exome sequencing data

## Inputs

- Positive training samples: PCAWG fusion genes (3540 fusions), provided in the 'data' folder
- Homologous gene file, provided in the 'data' folder
- Input paired-end RNA-seq dataset (FASTQ) from which artificial chimeric negatives are generated
- Anchored gene target transcript FASTA (from NCBI GENE) for defining the anchored gene X

## Outputs

- Retrained HVLD filter model file (location given by '--model_file', default in the 'data' folder)
- Per-candidate fusion HVLD probability scores; candidates with probability < 0.1 filtered out
- Benchmark accuracy table when training baselines (per-dataset and average accuracy across K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12)

## How to apply

Build per-dataset training sets: use the 3540 PCAWG fusion genes as positive samples and generate artificial chimeric reads as negatives for the current dataset, removing negative samples containing the anchored gene X and reads aligned to homologous genes. Train the HVLD model in PyTorch (>=1.10.2), combining hierarchical view learning with the self-distillation loss whose two loss types are balanced by the alpha hyperparameter; positive training samples and the homologous gene file are shipped in the program's 'data' folder and must not be deleted. For benchmarking, train baselines on the same splits: bi-LSTM, Transformer, and MLP in PyTorch, SVM with scikit-learn (>=1.3), and the HVL variant without distillation. Save the retrained filter via '--model_file' (default under the 'data' folder); candidates with HVLD probability below 0.1 are subsequently filtered out. If the user dataset is too small, fall back to the default pre-trained model parameters on GitHub instead of retraining.

## Related tools

- **Anchored-fusion** (Hosts the HVLD filter training; run via Anchored_Fusion.py with '--model_file' specifying the trained filter) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **PyTorch** (Framework for training HVLD, HVL, bi-LSTM, Transformer, and MLP classifiers)
- **scikit-learn** (Trains the SVM baseline classifier on the same chimeric-read splits)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- HVLD reaches approximately 82.1% average accuracy across the six CCLE datasets (max 83.4%, median 82.1%, min 80.8%)
- HVLD exceeds the Transformer baseline by about 8.85% and the HVL no-distillation variant by about 0.75%
- Negative training samples containing the anchored gene X and reads aligned to homologous genes are absent from the training set
- The retrained model file is written to the '--model_file' path (default under the 'data' folder)
- At inference, fusions with HVLD probability below 0.1 are excluded from final calls

## Limitations

- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified in the article
- If the user dataset is too small, retraining is unreliable; use the default model parameters from GitHub
- Using the pre-trained model via '--not_train_filter_model' may give deviating results since it was not trained on the user's input data
- Only fusion genes including the user-provided target gene can be detected

## Evidence
