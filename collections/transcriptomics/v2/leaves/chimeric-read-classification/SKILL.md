---
name: chimeric-read-classification
description: Use when building or evaluating the false-positive filter of a targeted
  fusion-detection pipeline that must distinguish true fusion reads from artificial
  chimeric reads in paired-end RNA-seq data.
when_to_use_negative:
- The input dataset is too small to retrain the filter reliably — use the default
  pretrained model parameters provided on GitHub instead
- You only need raw fusion candidates and want to save time — skip the filter with
  '--not_filter_false_positive'
- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads,
  which Anchored-fusion cannot process
edam_operation: http://edamontology.org/operation_0465
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Anchored-fusion
  role: Pipeline whose HVLD module performs the chimeric-read classification and filtering
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: PyTorch
  role: Framework for training HVLD, HVL, bi-LSTM, Transformer, and MLP classifiers
- name: scikit-learn
  role: Provides the SVM baseline classifier for the benchmark
- name: NumPy
  role: Numerical handling of training feature arrays
- name: Biopython
  role: Sequence handling during training-sample construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:e5932795d4296cc924010826dbd39729994cb6c6fbc54140a1cb1158f4f93e6e
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
  edam_operation: http://edamontology.org/operation_0465
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# chimeric-read-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and benchmark the HVLD deep-learning filter (hierarchical view learning with self-distillation) that classifies candidate chimeric reads in Anchored-fusion as true fusion reads versus artificial cDNA/PCR chimeras, comparing against bi-LSTM, Transformer, MLP, SVM, and the no-distillation HVL variant.

## When to use

Use when building or evaluating the false-positive filter of a targeted fusion-detection pipeline that must distinguish true fusion reads from artificial chimeric reads in paired-end RNA-seq data.

## When NOT to use

- The input dataset is too small to retrain the filter reliably — use the default pretrained model parameters provided on GitHub instead
- You only need raw fusion candidates and want to save time — skip the filter with '--not_filter_false_positive'
- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads, which Anchored-fusion cannot process

## Inputs

- Positive training samples: PCAWG fusion genes (3540 fusions, from Synapse:syn10003873)
- Artificial chimeric reads generated from the input dataset as negative samples
- Homologous gene file shipped in the 'data' folder of the Anchored-Fusion repository
- Paired-end RNA-seq FASTQ of the target dataset (e.g., CCLE cell lines, SRA:SRP186687)
- Pretrained HVLD model file (for '--not_train_filter_model' mode, via '--model_file')

## Outputs

- Retrained HVLD classification model saved to the '--model_file' path (default under the 'data' folder)
- Per-dataset and average accuracy table for HVLD, HVL, bi-LSTM, Transformer, MLP, and SVM
- Per-candidate fusion probability used downstream to filter fusions with probability < 0.1

## How to apply

For each dataset (e.g., six CCLE bRNA-seq cell lines K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12), build training sets using PCAWG fusions (3540 fusion genes) as positive samples and artificial chimeric reads as negatives, removing negative samples containing the anchored gene and reads aligned to homologous genes. Train HVLD in PyTorch (>=1.10.2) with its hierarchical view learning and self-distillation loss, balancing the two loss types with the alpha hyperparameter, using the positive training samples and homologous gene file shipped in the 'data' folder. Train baselines on the same splits: bi-LSTM, Transformer, and MLP in PyTorch, and SVM with scikit-learn (>=1.3), plus the HVL variant without distillation. Evaluate all models per dataset by classification accuracy; alternatively skip retraining via '--not_train_filter_model' with a pretrained model passed through '--model_file'. Save the retrained filter to the '--model_file' location (default under the 'data' folder) and write the benchmark accuracy table to the output folder.

## Related tools

- **Anchored-fusion** (Pipeline whose HVLD module performs the chimeric-read classification and filtering) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **PyTorch** (Framework for training HVLD, HVL, bi-LSTM, Transformer, and MLP classifiers)
- **scikit-learn** (Provides the SVM baseline classifier for the benchmark)
- **NumPy** (Numerical handling of training feature arrays)
- **Biopython** (Sequence handling during training-sample construction)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- HVLD must reach approximately 82.1% average accuracy across the six CCLE datasets (max 83.4%, median 82.1%, min 80.8%)
- HVLD must exceed the Transformer baseline by about 8.85% and the HVL no-distillation variant by about 0.75%
- The retrained model file must exist at the '--model_file' path (default under the 'data' folder)
- Downstream fusion filtering must remove genes with HVLD probability less than 0.1 while retaining true fusion genes
- Benchmark accuracy table must cover all six datasets (K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12) and all six models

## Limitations

- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified
- Pretrained models ('--not_train_filter_model') were not trained on the user's input data, so results may deviate
- If the input dataset is too small, retraining is unreliable and the default GitHub model parameters are recommended
- Skipping the filter ('--not_filter_false_positive') saves time but leaves artificial chimeric false positives in the output

## Evidence
