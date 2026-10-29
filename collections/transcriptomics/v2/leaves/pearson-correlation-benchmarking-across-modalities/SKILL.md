---
name: pearson-correlation-benchmarking-across-modalities
description: Use when you have cell-level embeddings from multiple modalities (joint
  mvTCR, RNA-only, TCR-only) plus a continuous per-cell target such as pMHC multimer
  read counts, and need to quantify whether multimodality improves prediction.
when_to_use_negative:
- The target is categorical antigen specificity rather than continuous pMHC multimer
  read counts — a classification benchmark is required instead of MSLE regression
- Only one modality or embedding is available, so no cross-modality comparison can
  be made
- No specificity/avidity annotation exists for the dataset (the 03_training_prediction
  workflow requires specificity annotation)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Generates the multimodal cell-level embedding (plus RNA-only and TCR-only
    ablations) used for training the avidity prediction heads
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Deep learning backend for training the mvTCR model and the fully connected
    prediction heads with MSLE loss
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:01c7398569b316e4a2ebaacee86c4270a47dcec22c8909d7c58fc9bf432a78b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - mvTCR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pearson-correlation-benchmarking-across-modalities

## Summary

Benchmark a multimodal embedding (mvTCR) against unimodal RNA-only and TCR-only embeddings by training per-embedding prediction heads with mean squared logarithmic error (MSLE) loss and comparing MSLE and Pearson correlation on held-out cells, using pMHC multimer read counts as the regression target.

## When to use

Use when you have cell-level embeddings from multiple modalities (joint mvTCR, RNA-only, TCR-only) plus a continuous per-cell target such as pMHC multimer read counts, and need to quantify whether multimodality improves prediction.

## When NOT to use

- The target is categorical antigen specificity rather than continuous pMHC multimer read counts — a classification benchmark is required instead of MSLE regression
- Only one modality or embedding is available, so no cross-modality comparison can be made
- No specificity/avidity annotation exists for the dataset (the 03_training_prediction workflow requires specificity annotation)

## Inputs

- Preprocessed 10x dataset (>60k cells) with paired scRNA-seq and TCR data
- pMHC multimer read counts (avidity) annotations per cell
- Cell-level embeddings: multimodal mvTCR, RNA-only, and TCR-only (trained with mvTCR v0.1.3 on PyTorch, GPU-enabled)

## Outputs

- avidity_metrics.csv with one row per embedding type (mvTCR, RNA, TCR) and columns for MSLE and Pearson correlation
- Trained per-embedding fully connected regression heads

## How to apply

Load the preprocessed 10x dataset (>60k cells) with specificity/avidity annotations and obtain the three cell-level representations: the multimodal mvTCR embedding, the RNA-only embedding, and the TCR-only embedding. For each embedding, train a fully connected prediction head optimized with MSLE loss to regress pMHC multimer read counts, following the training-with-prediction setup of tutorials/03_training_prediction.ipynb. Evaluate each head on held-out cells, computing MSLE and Pearson correlation between predicted and true multimer read counts. Compare against the reference values (TCR MSLE 0.045, RNA MSLE 0.063; Pearson gains of TCR 0.074 and RNA 0.028) and write a metrics table with one row per embedding type to avidity_metrics.csv. The rationale is that the joint embedding fuses modality-specific information across transcriptome and TCR, so it should decrease MSLE and increase Pearson correlation relative to either unimodal representation.

## Related tools

- **mvTCR** (Generates the multimodal cell-level embedding (plus RNA-only and TCR-only ablations) used for training the avidity prediction heads) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Deep learning backend for training the mvTCR model and the fully connected prediction heads with MSLE loss)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3
```

## Evaluation signals

- avidity_metrics.csv exists and contains exactly three rows (mvTCR, RNA, TCR) with MSLE and Pearson fields
- Reported TCR MSLE falls in the range 0.03–0.06, consistent with the manuscript value of 0.045
- Reported RNA MSLE falls in the range 0.05–0.08, consistent with the manuscript value of 0.063
- The joint mvTCR embedding yields lower MSLE and higher Pearson correlation than both unimodal embeddings (TCR Pearson 0.074, RNA Pearson 0.028 as reference gains)
- Prediction heads were trained with MSLE loss following the tutorials/03_training_prediction.ipynb setup and evaluated on held-out cells

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing data remains insufficiently addressed
- Hyperparameter optimization was performed for 48 GPU-hours and results may diverge across hardware/CUDA versions; pre-trained models are provided at https://doi.org/10.5281/zenodo.7215447
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embeddings are not incorporated

## Evidence

- [results] The multimodal mvTCR embedding proved beneficial for avidity prediction, decreasing MSLE (TCR: 0.045; RNA: 0.063): [evidence span withheld — anchor status: not_found]
- [methods] tutorial for training with prediction: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [readme] computational reference for 10x dataset: "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)."
- [readme] reproducibility environment versions: [evidence span withheld — anchor status: external_doc]
- [abstract] multimodal added value claim: "Our analysis demonstrates the added value of multimodal over unimodal approaches to capture antigen specificity"
