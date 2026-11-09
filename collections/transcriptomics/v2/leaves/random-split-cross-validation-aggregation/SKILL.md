---
name: random-split-cross-validation-aggregation
description: Use when comparing model variants (multimodal vs unimodal) on a prediction
  task with specificity annotation and you need statistically robust mean performance
  across random splits rather than a single train/test result.
when_to_use_negative:
- The dataset lacks specificity annotation — use tutorials/02_training_analysis.ipynb
  (training/analysis with cell type and clonotype annotation) instead of the prediction
  setup
- You need a single deterministic train/test estimate rather than a variance-aware
  multi-split comparison, e.g. for quick debugging on the 15-minute subsampled tutorial
  dataset
- Input data is not paired TCR + transcriptome multimodal single-cell data (e.g. BCR/SHM
  datasets, which the paper notes lack large-scale specificity-annotated benchmarks)
tools:
- name: mvTCR
  role: Multimodal (TCR + transcriptome) VAE whose embeddings are benchmarked against
    unimodal baselines across five random splits
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
- name: PyTorch
  role: Deep-learning backend used to train mvTCR and baseline models; version must
    match CUDA build (v2.1.2 for current mvTCR, v1.8.0 for paper reproduction)
  repo: https://pytorch.org/get-started/previous-versions/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:42a0e9e85d1db9789d14d429de0e70274b64938c5fda41052aea5dd4579c3fde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR_reproducibility
  tools:
  - PyTorch
  - mvTCR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# random-split cross-validation aggregation

## Summary

Benchmarking protocol that trains a model (e.g. mvTCR and unimodal TCR/RNA baselines) across five random data splits, evaluates kNN antigen-specificity prediction on each held-out split, and aggregates mean F1-Scores for comparison against published values.

## When to use

Use when comparing model variants (multimodal vs unimodal) on a prediction task with specificity annotation and you need statistically robust mean performance across random splits rather than a single train/test result.

## When NOT to use

- The dataset lacks specificity annotation — use tutorials/02_training_analysis.ipynb (training/analysis with cell type and clonotype annotation) instead of the prediction setup
- You need a single deterministic train/test estimate rather than a variance-aware multi-split comparison, e.g. for quick debugging on the 15-minute subsampled tutorial dataset
- Input data is not paired TCR + transcriptome multimodal single-cell data (e.g. BCR/SHM datasets, which the paper notes lack large-scale specificity-annotated benchmarks)

## Inputs

- Preprocessed 10x Genomics single-cell dataset (>60k cells) with pMHC specificity annotation
- Annotation/filtering steps from tutorials/01_preprocessing.ipynb
- Model definitions: mvTCR MoE model plus unimodal TCR-only and RNA-only baselines
- Pretrained models downloadable from https://doi.org/10.5281/zenodo.7215447 (optional)

## Outputs

- Per-split F1-Scores for specificity prediction (each model, split, pooled/per-donor setting)
- Aggregated mean F1 across five random splits per model
- Results table (e.g. specificity_f1_results.csv)
- Trained model checkpoints per split

## How to apply

Set up an isolated conda environment (Python 3.10, pip install mvtcr, PyTorch v2.1.2 with CUDA-matched build; the paper's experiments used mvTCR v0.1.3 with PyTorch v1.8.0). Load the preprocessed 10x dataset (>60k cells with specificity annotation), apply the annotation/filtering steps from tutorials/01_preprocessing.ipynb, then train the mvTCR MoE model and the TCR-only and RNA-only baselines via the training-with-prediction setup of tutorials/03_training_prediction.ipynb, repeating training for five random data splits in pooled and per-donor settings. For each trained model, compute cell embeddings and train a kNN classifier on held-out splits to predict pMHC (SARS-CoV-2 antigen) specificity; compute F1-Score per model/split/setting and aggregate the mean across the five splits. Validate that aggregated means fall within tolerance of the reported mvTCR 0.821, TCR 0.767, RNA 0.759, and preserve the ordering multimodal > TCR-only > RNA-only.

## Related tools

- **mvTCR** (Multimodal (TCR + transcriptome) VAE whose embeddings are benchmarked against unimodal baselines across five random splits) — https://github.com/SchubertLab/mvTCR_reproducibility
- **PyTorch** (Deep-learning backend used to train mvTCR and baseline models; version must match CUDA build (v2.1.2 for current mvTCR, v1.8.0 for paper reproduction)) — https://pytorch.org/get-started/previous-versions/

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr
```

## Evaluation signals

- Aggregated mean F1-Scores fall within a small tolerance of the reported values: mvTCR 0.821, TCR-only 0.767, RNA-only 0.759
- Ordering preserved: multimodal (mvTCR) > TCR-only > RNA-only in every aggregation
- Reported significance holds: mvTCR vs TCR p = 0.0051, mvTCR vs RNA p = 0.0016 (five random splits)
- Results include both pooled and per-donor settings, each with five random splits
- Final table written (specificity_f1_results.csv) with rows for each model/split/setting before aggregation

## Limitations

- Resulting models may diverge from the publication across different compute clusters, hardware, and CUDA versions; pretrained models are provided on Zenodo (10.5281/zenodo.7215447) as a fallback
- Hyperparameter optimization for the manuscript consumed ~48 GPU-hours, and GPU support is heavily encouraged; 8GB RAM suffices for the >60k-cell 10x dataset
- mvTCR was applied mainly to fully paired data (gene expression plus α- and β-CDR3); partially missing data handling remains insufficiently addressed, and only the CDR3 sequence is currently used for the TCR representation

## Evidence

- [methods] tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation.: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [discussion] mvTCR embedding surpasses unimodal representations: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and"
- [readme] Paper experiments used pinned versions: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [readme] Hardware/CUDA reproducibility caveat: [evidence span withheld — anchor status: external_doc]
