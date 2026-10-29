---
name: knn-label-transfer-evaluation
description: Use when paired TCR+transcriptome query single-cell studies must be annotated
  by mapping them into a pre-trained mvTCR reference embedding, and accuracy of kNN
  label transfer for metadata labels (e.g. cancer source, tissue type) must be quantified
  against a baseline projection.
when_to_use_negative:
- Data is unimodal (gene expression or TCR only) — mvTCR's benefit is fusing both
  modalities; use a unimodal label-transfer tool instead
- No pre-annotated reference atlas exists — there is no labeled reference embedding
  to transfer labels from
- Goal is predicting antigen specificity from specificity annotations rather than
  transferring metadata labels via reference mapping (use tutorials/03_training_prediction.ipynb
  instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mvTCR
  role: Multi-view generative model trained conditionally on study and cell-type annotations
    to produce the reference embedding used for kNN label transfer
  repo: https://github.com/SchubertLab/mvTCR
- name: scArches
  role: Architectural surgery that freezes reference mvTCR weights and fine-tunes
    query weights to map new multimodal studies into the reference latent space
- name: Scanpy
  role: Single-cell workflow format in which the mvTCR embedding is stored and neighbor
    graphs/kNN label transfer are computed
- name: PyTorch
  role: Deep-learning runtime (v1.8.0 with matching CUDA version) required to train
    and fine-tune the mvTCR model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:2665eb785ea557c2c4693f98ca2bb59f6ad0e6c70829ebddf30ea78ae346f5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  - scArches
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# kNN label-transfer evaluation via mvTCR reference mapping

## Summary

Evaluates whether a conditional mvTCR model combined with scArches architectural surgery improves k-nearest-neighbor label transfer (cancer source, tissue type, cell type) of query cells mapped into a large TIL reference atlas, compared to a non-conditional projection baseline.

## When to use

Use when paired TCR+transcriptome query single-cell studies must be annotated by mapping them into a pre-trained mvTCR reference embedding, and accuracy of kNN label transfer for metadata labels (e.g. cancer source, tissue type) must be quantified against a baseline projection.

## When NOT to use

- Data is unimodal (gene expression or TCR only) — mvTCR's benefit is fusing both modalities; use a unimodal label-transfer tool instead
- No pre-annotated reference atlas exists — there is no labeled reference embedding to transfer labels from
- Goal is predicting antigen specificity from specificity annotations rather than transferring metadata labels via reference mapping (use tutorials/03_training_prediction.ipynb instead)

## Inputs

- Preprocessed TIL reference atlas with paired TCR and transcriptome data plus cell type, cancer source, and tissue type annotations (multi-study reference)
- Held-out paired TCR + gene expression query studies (e.g. lung cancer queries)
- Trained conditional mvTCR reference model checkpoint
- Non-conditional mvTCR projection baseline model

## Outputs

- Query-cell label predictions for cancer source, tissue type, and cell type via kNN voting in the reference embedding
- Per-condition kNN label-transfer accuracy metrics (e.g. knn_label_transfer_metrics.csv) with conditional-vs-baseline improvement
- Query cells embedded in the reference mvTCR latent space

## How to apply

Download the preprocessed TIL reference atlas from SchubertLab/mvTCR_reproducibility and run mvTCR preprocessing (tutorials/01_preprocessing.ipynb) to add cell type, cancer source, and tissue type annotations to the multi-study reference. Train a conditional mvTCR model (mvTCR v0.1.3 with PyTorch v1.8.0 on a GPU machine) following tutorials/02_training_analysis.ipynb, conditioning on study and cell-type annotations. Freeze the reference weights and map held-out query studies into the reference latent space via scArches architectural surgery (fine-tuning only the query weights). Annotate query cells by k-nearest-neighbor voting in the reference embedding for each label (cancer source, tissue type, cell type), once with the conditional model and once with a non-conditional projection baseline, and compute accuracy improvements per condition (reported as 4.1% for cancer source and 15.2% for tissue type).

## Related tools

- **mvTCR** (Multi-view generative model trained conditionally on study and cell-type annotations to produce the reference embedding used for kNN label transfer) — https://github.com/SchubertLab/mvTCR
- **scArches** (Architectural surgery that freezes reference mvTCR weights and fine-tunes query weights to map new multimodal studies into the reference latent space)
- **Scanpy** (Single-cell workflow format in which the mvTCR embedding is stored and neighbor graphs/kNN label transfer are computed)
- **PyTorch** (Deep-learning runtime (v1.8.0 with matching CUDA version) required to train and fine-tune the mvTCR model)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- Conditional model's kNN label-transfer accuracy on both held-out lung cancer queries exceeds the non-conditional projection baseline
- Improvement magnitudes approximate the reported values: +4.1% for cancer source and +15.2% for tissue type, within a small tolerance
- Per-condition metrics are written for all three labels (cancer source, tissue type, cell type), with cell type showing similar performance between conditional and baseline as reported
- Query cells occupy the reference latent space after scArches surgery (reference weights frozen, only query weights fine-tuned)
- Environment matches publication settings: mvTCR v0.1.3 with PyTorch v1.8.0; alternatively compare against the Zenodo-hosted trained models

## Limitations

- Different GPU hardware and CUDA versions cause resulting models to diverge from the publication's models; pre-trained models from Zenodo (https://doi.org/10.5281/zenodo.7215447) are provided as a reference
- UMAP visualization may not be fully recreated due to reproducibility issues in the underlying library
- Hyperparameter optimization is GPU-intensive (48 GPU-hours in the paper), and adjusting the contribution of each modality requires repeated retraining with an additional hyperparameter search
- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); partially missing data is insufficiently addressed

## Evidence

- [discussion] When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases: "When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases"
- [discussion] mvTCR embedding surpasses unimodal representations and other multimodal methods, including reference mapping: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction, reference mapping"
- [readme] Reproducibility environment pinned to mvTCR v0.1.3 and PyTorch v1.8.0: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [readme] Reproducibility repo instructions to clone and install the pinned environment: [evidence span withheld — anchor status: external_doc]
