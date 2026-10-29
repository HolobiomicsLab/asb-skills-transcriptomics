---
name: unimodal-baseline-model-training
description: Use when benchmarking a multimodal TCR+transcriptome model against unimodal
  baselines on specificity-annotated single-cell data, to test whether modality fusion
  improves pMHC antigen specificity prediction.
when_to_use_negative:
- The dataset lacks pMHC specificity annotations, so the training-with-prediction
  setup cannot be applied (use the training-analysis setup of tutorials/02_training_analysis.ipynb
  instead)
- Only paired α/β-CDR3 TCR data with gene expression is unavailable for one modality
  — mvTCR and its unimodal baselines were mainly applied to fully paired data and
  partially missing data handling is insufficiently addressed
- No GPU/CUDA-capable machine is available, since model training for >60k cells requires
  GPU support
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Provides the unimodal TCR-only and RNA-only baseline models trained with the
    same training-with-prediction setup as the multimodal model; installed via pip
    install mvtcr in a Python 3.10 conda environment
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Deep-learning backend required to train the unimodal baseline models; v2.1.2
    (or v1.8.0 for exact paper reproduction) installed with a CUDA-matched build
- name: Scanpy
  role: Data format for the preprocessed 10x single-cell input that the baselines
    consume
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:bc18889e65ad0aea0925f22224a865d3dada2d57534be4a5a9a4be795a6f68c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# unimodal-baseline-model-training

## Summary

Trains unimodal TCR-only and RNA-only baseline models alongside the multimodal mvTCR model so that the added value of fusing transcriptome and TCR modalities for pMHC antigen specificity prediction can be quantified via kNN F1-Score on held-out splits.

## When to use

Use when benchmarking a multimodal TCR+transcriptome model against unimodal baselines on specificity-annotated single-cell data, to test whether modality fusion improves pMHC antigen specificity prediction.

## When NOT to use

- The dataset lacks pMHC specificity annotations, so the training-with-prediction setup cannot be applied (use the training-analysis setup of tutorials/02_training_analysis.ipynb instead)
- Only paired α/β-CDR3 TCR data with gene expression is unavailable for one modality — mvTCR and its unimodal baselines were mainly applied to fully paired data and partially missing data handling is insufficiently addressed
- No GPU/CUDA-capable machine is available, since model training for >60k cells requires GPU support

## Inputs

- Preprocessed 10x Genomics SARS-CoV-2 dataset (>60k cells) with pMHC specificity annotations (.h5ad/Scanpy format)
- Five random train/test data splits (pooled and per-donor settings)
- Trained unimodal TCR-only and RNA-only model checkpoints (mvTCR v0.1.3 was used for the paper's reference runs with PyTorch v1.8.0)

## Outputs

- Cell embeddings from the TCR-only model
- Cell embeddings from the RNA-only model
- Per-split and mean F1-Scores of kNN-based pMHC specificity prediction for each unimodal baseline (reference: TCR 0.767, RNA 0.759)
- Results table (e.g. specificity_f1_results.csv) with F1 per model, split, and setting

## How to apply

Set up an isolated conda environment (Python 3.10, pip install mvtcr, PyTorch v2.1.2 with a CUDA-matched build per pytorch.org previous-versions) and preprocess the 10x Genomics SARS-CoV-2 dataset (>60k specificity-annotated cells) following tutorials/01_preprocessing.ipynb annotations and filtering. Train the unimodal TCR-only and RNA-only baselines using the training-with-prediction setup of tutorials/03_training_prediction.ipynb, repeating each training across five random data splits in both pooled and per-donor settings to match the multimodal model's splits. Compute cell embeddings for each unimodal model and train a kNN classifier on the embeddings to predict pMHC (SARS-CoV-2 antigen) specificity on the held-out split. Compute per-split F1-Scores, aggregate means over the five splits, and compare against the multimodal model with paired significance tests; the paper reports TCR-only 0.767 (p=0.0051 vs mvTCR) and RNA-only 0.759 (p=0.0016 vs mvTCR).

## Related tools

- **mvTCR** (Provides the unimodal TCR-only and RNA-only baseline models trained with the same training-with-prediction setup as the multimodal model; installed via pip install mvtcr in a Python 3.10 conda environment) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Deep-learning backend required to train the unimodal baseline models; v2.1.2 (or v1.8.0 for exact paper reproduction) installed with a CUDA-matched build)
- **Scanpy** (Data format for the preprocessed 10x single-cell input that the baselines consume)

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr
```

## Evaluation signals

- Mean F1-Score over five random splits falls within a small tolerance of the reported TCR-only 0.767 and RNA-only 0.759
- Preserved ordering multimodal (0.821) > TCR-only (0.767) > RNA-only (0.759) across pooled and per-donor settings
- Paired significance tests against the multimodal model are significant (p=0.0051 for TCR-only, p=0.0016 for RNA-only)
- All five random splits completed for both pooled and per-donor settings, matching the splits used for the mvTCR model
- Results table written to specificity_f1_results.csv with per-model, per-split, per-setting rows

## Limitations

- Results may diverge from the publication across different hardware, compute clusters, and CUDA versions; the reproducibility repo provides all trained models via Zenodo (https://doi.org/10.5281/zenodo.7215447)
- The paper's reference results were obtained with mvTCR v0.1.3 and PyTorch v1.8.0, not the newer default versions
- The 10x dataset used contains only CDR3-based TCR representation; VDJ-gene encoding and pre-trained TCR embeddings are not incorporated
- Training on the full >60k-cell dataset requires GPU support and scales with dataset size (8GB RAM was sufficient for the paper's 10x dataset)

## Evidence

- [methods] tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation.: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [results] Averaged over five random splits, mvTCR significantly outperforms both unimodal baselines in pMHC specificity prediction: [evidence span withheld — anchor status: not_found]
- [abstract] Our analysis demonstrates the added value of multimodal over unimodal approaches to capture antigen specificity: "Our analysis demonstrates the added value of multimodal over unimodal approaches to capture antigen specificity"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "To reproduce the results of the paper, please refer to: https://github.com/SchubertLab/mvTCR_reproducibility
The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [readme] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)."
