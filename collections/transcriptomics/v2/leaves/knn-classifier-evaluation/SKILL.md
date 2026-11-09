---
name: knn-classifier-evaluation
description: Use when single-cell data has specificity annotation (e.g. pMHC binding
  labels on a 10x Genomics T cell dataset) and you need to score embedding quality
  by antigen-specificity prediction.
when_to_use_negative:
- Data lacks specificity annotation — use the training-analysis workflow (tutorials/02_training_analysis.ipynb)
  for cell-type/clonotype annotation instead
- Data is unpaired (gene expression without α- and β-CDR3 TCR sequences) — mvTCR was
  mainly applied to fully paired data and partially missing data handling is insufficiently
  addressed
- Goal is reference mapping of new query datasets — combine mvTCR with scArches reference
  mapping rather than kNN specificity prediction
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Multimodal generative model (multi-view VAE) producing the shared TCR+transcriptome
    cell embeddings on which the kNN specificity classifier is trained; also provides
    TCR-only and RNA-only unimodal baselines
  repo: https://github.com/SchubertLab/mvTCR
- name: mvTCR_reproducibility
  role: Repository with experiment settings, trained models (Zenodo), evaluation and
    figure notebooks used to reproduce the kNN F1-Score benchmark
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
- name: Scanpy
  role: Format underlying the mvTCR representation so embeddings and predictions integrate
    into standard single-cell analysis workflows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:697adda1aa9f6042b59b587993f7c745c4b4cf70ee5d6dbc1a041e51a4840328
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - Scanpy
  - mvTCR
  - mvTCR_reproducibility
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# kNN-classifier evaluation of mvTCR embeddings for antigen specificity prediction

## Summary

Trains a k-nearest-neighbor classifier on mvTCR (and unimodal TCR-only/RNA-only) cell embeddings to predict pMHC (SARS-CoV-2) antigen specificity, and benchmarks F1-Scores across five random splits to quantify the multimodal benefit.

## When to use

Use when single-cell data has specificity annotation (e.g. pMHC binding labels on a 10x Genomics T cell dataset) and you need to score embedding quality by antigen-specificity prediction.

## When NOT to use

- Data lacks specificity annotation — use the training-analysis workflow (tutorials/02_training_analysis.ipynb) for cell-type/clonotype annotation instead
- Data is unpaired (gene expression without α- and β-CDR3 TCR sequences) — mvTCR was mainly applied to fully paired data and partially missing data handling is insufficiently addressed
- Goal is reference mapping of new query datasets — combine mvTCR with scArches reference mapping rather than kNN specificity prediction

## Inputs

- Preprocessed 10x Genomics single-cell T cell dataset (>60k cells) with pMHC specificity annotation (SARS-CoV-2 antigens)
- Trained mvTCR, TCR-only, and RNA-only models (mvTCR v0.1.3 / PyTorch 1.8.0 for paper reproduction, PyTorch v2.1.2 for the tutorial setup)
- Cell embeddings computed from each trained model

## Outputs

- Per-model, per-split, per-setting F1-Scores of pMHC specificity prediction
- Aggregated mean F1-Score across five random splits (mvTCR 0.821, TCR-only 0.767, RNA-only 0.759 in the paper)
- Results table written to specificity_f1_results.csv

## How to apply

Set up a conda environment (Python 3.10, pip install mvtcr, PyTorch v2.1.2 with a CUDA-matched build) and preprocess the 10x dataset (>60k cells) following tutorials/01_preprocessing.ipynb. Train the mvTCR MoE model plus TCR-only and RNA-only baselines using the training-with-prediction setup of tutorials/03_training_prediction.ipynb, repeating each training over five random data splits in pooled and per-donor settings. Compute cell embeddings for every trained model and fit a kNN classifier on the embeddings to predict pMHC specificity on the held-out split. Compute per-model, per-split F1-Scores and aggregate the mean across the five splits; compare against the reported 0.821 / 0.767 / 0.759 (mvTCR / TCR / RNA) and require the ordering multimodal > TCR-only > RNA-only to hold. Write the full results table to specificity_f1_results.csv.

## Related tools

- **mvTCR** (Multimodal generative model (multi-view VAE) producing the shared TCR+transcriptome cell embeddings on which the kNN specificity classifier is trained; also provides TCR-only and RNA-only unimodal baselines) — https://github.com/SchubertLab/mvTCR
- **mvTCR_reproducibility** (Repository with experiment settings, trained models (Zenodo), evaluation and figure notebooks used to reproduce the kNN F1-Score benchmark) — https://github.com/SchubertLab/mvTCR_reproducibility
- **Scanpy** (Format underlying the mvTCR representation so embeddings and predictions integrate into standard single-cell analysis workflows)

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr
```

## Evaluation signals

- Aggregated mean F1-Scores fall within a small tolerance of the reported mvTCR 0.821, TCR-only 0.767, RNA-only 0.759
- The ordering multimodal > TCR-only > RNA-only is preserved across the five splits
- Statistical significance holds: mvTCR vs TCR-only p-value 0.0051 and mvTCR vs RNA-only p-value 0.0016 (Wilcoxon-type comparison reported in the paper)
- Each of the five random splits is trained and evaluated in both pooled and per-donor settings before aggregation
- specificity_f1_results.csv contains the full per-split, per-model, per-setting results table

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing data remains insufficiently addressed
- Trained models may diverge from the publication's due to differing hardware and CUDA versions, so pre-trained models from Zenodo (doi 10.5281/zenodo.7215447) are recommended for exact reproduction
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embeddings are not incorporated, potentially capping specificity-prediction accuracy
- Hyperparameter optimization is compute-intensive (48 GPU-hours per dataset in the reproducibility repo); 8GB RAM suffices for the 10x dataset but GPU support is heavily encouraged

## Evidence

- [methods] training with prediction task: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [methods] computational resources: "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [discussion] benchmark finding: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction"
- [readme] reproducibility environment: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [abstract] application to SARS-CoV-2 data: "we use mvTCR to distinguish T cell subpopulations binding to SARS-CoV-2 antigens from bystander cells"
