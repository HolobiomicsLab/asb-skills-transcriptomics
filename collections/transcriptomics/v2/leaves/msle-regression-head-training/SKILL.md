---
name: msle-regression-head-training
description: Use when cells have pMHC multimer read counts (continuous avidity annotations)
  alongside paired TCR and transcriptome data, and you must compare joint vs. unimodal
  embedding quality for avidity regression on the 10x dataset.
when_to_use_negative:
- The task is antigen specificity classification rather than continuous avidity regression
  — use the training-prediction setup for discrete specificity labels instead of MSLE
  count regression.
- Only unpaired or clonotype-level TCR data is available (cells lack paired α/β CDR3
  with gene expression), since mvTCR was mainly applied to fully paired data.
- The input is already a metrics table or you only need qualitative clustering/UMAP
  analysis of embeddings, not supervised regression.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: mvTCR
  role: Provides the multimodal cell-level embedding and the prediction-head training
    framework used to regress pMHC multimer read counts (tutorials/03_training_prediction.ipynb)
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Deep learning backend for training mvTCR and the fully connected regression
    head (installed with mvTCR v0.1.3, PyTorch v1.8.0, CUDA-enabled GPU)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:e3034b1be16d18730b6ae4220629f1591855b1f8066a00da66d4975bf16c5087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - mvTCR
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# msle-regression-head-training

## Summary

Train fully connected regression heads on mvTCR (multimodal), RNA-only, and TCR-only embeddings to predict pMHC multimer read counts (binding avidity), optimized with mean squared logarithmic error (MSLE) loss and evaluated by MSLE and Pearson correlation on held-out cells. This quantifies whether the joint embedding outperforms unimodal representations for continuous avidity prediction.

## When to use

Use when cells have pMHC multimer read counts (continuous avidity annotations) alongside paired TCR and transcriptome data, and you must compare joint vs. unimodal embedding quality for avidity regression on the 10x dataset.

## When NOT to use

- The task is antigen specificity classification rather than continuous avidity regression — use the training-prediction setup for discrete specificity labels instead of MSLE count regression.
- Only unpaired or clonotype-level TCR data is available (cells lack paired α/β CDR3 with gene expression), since mvTCR was mainly applied to fully paired data.
- The input is already a metrics table or you only need qualitative clustering/UMAP analysis of embeddings, not supervised regression.

## Inputs

- Preprocessed 10x dataset (>60k cells) with pMHC multimer read count (specificity/avidity) annotations
- Cell-level mvTCR multimodal embedding
- RNA-only unimodal embedding
- TCR-only unimodal embedding

## Outputs

- avidity_metrics.csv with one row per embedding type (mvTCR, RNA, TCR) and columns for MSLE and Pearson correlation
- Trained fully connected regression head per embedding type

## How to apply

Load the preprocessed 10x dataset (>60k cells) with pMHC multimer read count annotations and obtain three cell-level representations: the multimodal mvTCR embedding plus RNA-only and TCR-only embeddings, training mvTCR (v0.1.3, PyTorch 1.8.0, GPU) if needed. For each embedding, train a fully connected prediction head with mean squared logarithmic error (MSLE) loss to regress multimer read counts, following the training-with-prediction setup of tutorials/03_training_prediction.ipynb. Evaluate each head on held-out cells, computing MSLE and Pearson correlation between predicted and true counts. The rationale is that MSLE loss suits the skewed count distribution of multimer reads, and comparing TCR vs RNA vs joint heads isolates the multimodal contribution. Reference values: TCR MSLE 0.045, RNA MSLE 0.063, with Pearson correlation gains of 0.074 (TCR) and 0.028 (RNA) over baseline; the joint mvTCR embedding decreases MSLE and increases Pearson correlation relative to both unimodal embeddings. For full reproducibility, use the SchubertLab/mvTCR_reproducibility repo and pre-trained models from Zenodo, since re-trained models may diverge from the publication.

## Related tools

- **mvTCR** (Provides the multimodal cell-level embedding and the prediction-head training framework used to regress pMHC multimer read counts (tutorials/03_training_prediction.ipynb)) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Deep learning backend for training mvTCR and the fully connected regression head (installed with mvTCR v0.1.3, PyTorch v1.8.0, CUDA-enabled GPU))

## Examples

```
pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y  # then run tutorials/03_training_prediction.ipynb with MSLE-loss prediction head on mvTCR, RNA-only, and TCR-only embeddings
```

## Evaluation signals

- avidity_metrics.csv exists and contains exactly three rows (mvTCR, RNA, TCR) with MSLE and Pearson correlation fields.
- TCR-only MSLE falls in the range 0.03–0.06 (reference: 0.045).
- RNA-only MSLE falls in the range 0.05–0.08 (reference: 0.063).
- The joint mvTCR embedding yields lower MSLE than both unimodal embeddings (TCR: 0.045; RNA: 0.063) and higher Pearson correlation (TCR gain 0.074; RNA gain 0.028).
- The trained heads use MSLE loss as the optimization objective, matching the manuscript's avidity prediction setup.

## Limitations

- Models re-trained on different hardware/CUDA versions may diverge from publication models; use pre-trained models from Zenodo (doi.org/10.5281/zenodo.7215447) for exact reproduction.
- Pearson correlation values are low in absolute terms (TCR: 0.074; RNA: 0.028), so gains are relative rather than strongly predictive.
- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing data remains insufficiently addressed.
- Hyperparameter optimization was run for 48 GPU-hours per dataset; skipping HPO may degrade avidity prediction performance.

## Evidence

- [results] The multimodal mvTCR embedding proved beneficial for avidity prediction, decreasing MSLE (TCR: 0.045; RNA: 0.063): [evidence span withheld — anchor status: not_found]
- [methods] Training with prediction tutorial: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [readme] Dataset scale reference: "As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)."
- [readme] Version pinning for reproducibility: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [discussion] Multimodal benefit claim from article: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction"
