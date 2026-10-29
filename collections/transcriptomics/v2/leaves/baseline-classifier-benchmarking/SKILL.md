---
name: baseline-classifier-benchmarking
description: Use when evaluating whether a deep learning filter (e.g. HVLD for chimeric-read/fusion-gene
  classification) genuinely improves over bi-LSTM, Transformer, MLP, SVM, or a no-distillation
  ablation on identical training splits.
when_to_use_negative:
- The input dataset is too small to reliably retrain per-dataset models — the paper
  recommends using the default pre-trained model parameter provided on GitHub instead
- The goal is de novo fusion discovery without an anchored gene of interest — this
  skill serves the targeted-anchoring/HVLD-filtering benchmark, not general fusion
  calling
- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads —
  chimeric-read classification for Anchored-fusion requires paired-end RNA-seq
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0622
tools:
- name: Anchored-fusion
  role: Provides the HVLD filter module, the 'data' folder with positive training
    samples and homologous gene file, and the '--model_file' mechanism for saving
    the retrained filter
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: PyTorch
  role: Framework for training HVLD (>=1.10.2) and the bi-LSTM, Transformer, and MLP
    baselines
- name: scikit-learn
  role: Provides the SVM baseline classifier (>=1.3)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:aa78f4556c27b2ac733940f6e7c5474bfda5eb57061709e711455320355d6f16
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# baseline-classifier-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the HVLD hierarchical-view-learning self-distillation filter against machine/deep learning baselines (bi-LSTM, Transformer, MLP, SVM, and the no-distillation HVL variant) on chimeric-read classification, quantifying how much distillation and hierarchical structure contribute to false-positive fusion filtering.

## When to use

Use when evaluating whether a deep learning filter (e.g. HVLD for chimeric-read/fusion-gene classification) genuinely improves over bi-LSTM, Transformer, MLP, SVM, or a no-distillation ablation on identical training splits.

## When NOT to use

- The input dataset is too small to reliably retrain per-dataset models — the paper recommends using the default pre-trained model parameter provided on GitHub instead
- The goal is de novo fusion discovery without an anchored gene of interest — this skill serves the targeted-anchoring/HVLD-filtering benchmark, not general fusion calling
- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing reads — chimeric-read classification for Anchored-fusion requires paired-end RNA-seq

## Inputs

- Per-dataset training sets for six CCLE cell lines (K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12; bRNA-seq from SRA:SRP186687)
- Positive training samples: PCAWG fusion genes (3540 fusion genes, Pan-Cancer Analysis of Whole Genomes)
- Negative training samples: artificial chimeric reads generated for each dataset
- Homologous gene file (included in the 'data' folder of the Anchored-Fusion repository)

## Outputs

- Per-dataset and average classification accuracy for HVLD, HVL (no distillation), bi-LSTM, Transformer, MLP, and SVM
- Benchmark accuracy table written to the output folder
- Retrained HVLD filter model file saved at the '--model_file' path (default under the 'data' folder)

## How to apply

Build per-dataset training sets for the six CCLE cell lines (K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12; SRA:SRP186687) using PCAWG fusion genes (3540 fusions) as positive samples and artificial chimeric reads as negatives, then remove negative samples containing the anchored gene X and reads aligned to homologous genes. Train HVLD in PyTorch (>=1.10.2) with its hierarchical view learning and self-distillation loss, balancing the two loss types with the alpha hyperparameter, using the positive training samples and homologous gene file shipped in the 'data' folder. Train the baselines on the same splits: bi-LSTM, Transformer, and MLP in PyTorch, SVM with scikit-learn (>=1.3), plus the HVL variant without distillation as the key ablation. Evaluate all six models on chimeric-read classification per dataset, computing per-dataset and average accuracy, and record how far HVLD exceeds the strongest baseline. Save the retrained HVLD filter to the location given by '--model_file' (default under the 'data' folder) and write the benchmark accuracy table to the output folder.

## Related tools

- **Anchored-fusion** (Provides the HVLD filter module, the 'data' folder with positive training samples and homologous gene file, and the '--model_file' mechanism for saving the retrained filter) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **PyTorch** (Framework for training HVLD (>=1.10.2) and the bi-LSTM, Transformer, and MLP baselines)
- **scikit-learn** (Provides the SVM baseline classifier (>=1.3))

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- HVLD must reach approximately 82.1% average chimeric-read classification accuracy across the six CCLE bRNA-seq datasets (max 83.4%, median 82.1%, min 80.8%)
- HVLD must exceed the Transformer baseline by about 8.85% average accuracy
- HVLD must exceed the HVL no-distillation variant by about 0.75% average accuracy, confirming the self-distillation component's contribution
- All six models (HVLD, HVL, bi-LSTM, Transformer, MLP, SVM) are evaluated on identical per-dataset splits for each of the six cell lines, with per-dataset accuracies reported
- The retrained HVLD filter file is written to the '--model_file' location (default under the 'data' folder) and the benchmark table is present in the output folder

## Limitations

- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified in the paper
- For small input datasets, users must rely on default model parameters from GitHub rather than retrained HVLD parameters, so the per-dataset benchmark training is not feasible
- Results of the pre-trained filter option ('--not_train_filter_model') may deviate because it has not been trained on the user's input data

## Evidence

- [discussion] We train a decision model to learn the distribution of the artificial fusion genes of the current dataset for each dataset inputted by users.: "We train a decision model to learn the distribution of the artificial fusion genes of the current dataset for each dataset inputted by users."
- [discussion] We utilized six CCLE cancer cell line bRNA-seq datasets: "We utilized six Cancer Cell Line Encyclopedia (CCLE) cancer cell line bRNA-seq datasets"
- [intro] We benchmarked Anchored-fusion against other methods in several conditions, including simulated fusion gene transcriptomic data, real bulk RNA-seq (bRNA-seq) data, and scRNA-seq data: "We benchmarked Anchored-fusion against other methods in several conditions, including simulated fusion gene transcriptomic data, real bulk RNA-seq (bRNA-seq) data, and scRNA-seq data"
- [readme] Anchored-fusion use a deep learning model to predict if the candidate fusion gene is derived from natural chromsome fusion or artifical cDNA fusion during PCR.: "Anchored-fusion use a deep learning model to predict if the candidate fusion gene is derived from natural chromsome fusion or artifical cDNA fusion during PCR."
- [discussion] genes with a low probability (less than 0.1) will be filtered out: "genes with a low probability (less than 0.1) will be filtered out."
