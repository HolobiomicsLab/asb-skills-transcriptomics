---
name: pmhc-multimer-avidity-prediction
description: Use when a paired single-cell TCR + transcriptome dataset carries pMHC
  multimer read counts as a per-cell avidity annotation and the goal is to compare
  avidity regression across embedding modalities.
when_to_use_negative:
- The dataset lacks per-cell pMHC multimer read counts or specificity annotations
  — use tutorials/02_training_analysis.ipynb (analysis) instead of the prediction
  setup
- The data is not paired TCR + transcriptome (e.g. only TCR sequences or only scRNA-seq),
  since mvTCR requires both modalities to build the joint embedding
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Multiview VAE producing joint, RNA-only, and TCR-only cell-level embeddings
    used as inputs to avidity regression heads
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Deep learning backend for training mvTCR and the fully connected prediction
    heads with MSLE loss
- name: mvTCR_reproducibility
  role: Repository providing experiment settings, trained models (Zenodo), and evaluation
    notebooks to reproduce manuscript avidity results
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:dbf80be4a200166d430f3a791a9ecadaa589a498ef2b3838bd124d30162bae16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - mvTCR
  - mvTCR_reproducibility
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pmhc-multimer-avidity-prediction

## Summary

Train fully connected prediction heads on mvTCR embeddings (joint, RNA-only, TCR-only) to regress pMHC multimer read counts, quantifying whether the multimodal embedding improves antigen binding avidity prediction. Regression quality is measured by MSLE and Pearson correlation on held-out cells.

## When to use

Use when a paired single-cell TCR + transcriptome dataset carries pMHC multimer read counts as a per-cell avidity annotation and the goal is to compare avidity regression across embedding modalities.

## When NOT to use

- The dataset lacks per-cell pMHC multimer read counts or specificity annotations — use tutorials/02_training_analysis.ipynb (analysis) instead of the prediction setup
- The data is not paired TCR + transcriptome (e.g. only TCR sequences or only scRNA-seq), since mvTCR requires both modalities to build the joint embedding

## Inputs

- Preprocessed 10x dataset (>60k cells) with paired TCR and gene expression data
- Per-cell specificity/avidity annotations (pMHC multimer read counts)
- Trained mvTCR model or precomputed joint, RNA-only, and TCR-only embeddings

## Outputs

- avidity_metrics.csv with one row per embedding type (mvTCR, RNA, TCR) and columns for MSLE and Pearson correlation
- Trained fully connected avidity prediction heads for each embedding

## How to apply

Load the preprocessed 10x dataset (>60k cells) with specificity/avidity annotations for pMHC multimer read counts under mvTCR v0.1.3 and PyTorch on a GPU-enabled machine. Obtain three cell-level representations — the joint mvTCR embedding, the RNA-only embedding, and the TCR-only embedding — training the model on the 10x data if embeddings are not already available. For each embedding, train a fully connected prediction head optimized with mean squared logarithmic error (MSLE) loss to regress multimer read counts, following tutorials/03_training_prediction.ipynb. Evaluate each head on held-out cells, computing MSLE and Pearson correlation between predicted and true counts, then compare against reference values (TCR MSLE 0.045, RNA MSLE 0.063; Pearson correlation gains of 0.074 and 0.028 over baseline).

## Related tools

- **mvTCR** (Multiview VAE producing joint, RNA-only, and TCR-only cell-level embeddings used as inputs to avidity regression heads) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Deep learning backend for training mvTCR and the fully connected prediction heads with MSLE loss)
- **mvTCR_reproducibility** (Repository providing experiment settings, trained models (Zenodo), and evaluation notebooks to reproduce manuscript avidity results) — https://github.com/SchubertLab/mvTCR_reproducibility

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- avidity_metrics.csv exists with exactly three rows (mvTCR, RNA, TCR) and contains MSLE and Pearson correlation fields
- Reported TCR-only MSLE falls in the range 0.03–0.06, consistent with the manuscript value 0.045
- Reported RNA-only MSLE falls in the range 0.05–0.08, consistent with the manuscript value 0.063
- Joint mvTCR embedding yields lower MSLE than both unimodal embeddings (TCR 0.045, RNA 0.063) and higher Pearson correlation
- Regression loss used during training is mean squared logarithmic error (MSLE), matching the tutorial setup

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing modalities remains insufficiently addressed
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not incorporated
- Different hardware, CUDA versions, and runtimes can produce models that diverge from publication models; pre-trained models from Zenodo (doi 10.5281/zenodo.7215447) are provided for exact reproduction
- Adjusting the contribution of each modality requires repeated training, ideally with an additional hyperparameter search

## Evidence

- [results] The multimodal mvTCR embedding proved beneficial for avidity prediction, decreasing MSLE (TCR: 0.045; RNA: 0.063): [evidence span withheld — anchor status: not_found]
- [methods] tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation.: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [methods] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [discussion] Yet, we applied it mainly to paired data, where gene expression is available in combination with α- and β-CDR3 of the TCR.: "Yet, we applied it mainly to paired data, where gene expression is available in combination with α- and β-CDR3 of the TCR."
