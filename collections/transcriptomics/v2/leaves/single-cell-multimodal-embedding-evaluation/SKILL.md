---
name: single-cell-multimodal-embedding-evaluation
description: Use when you have paired scRNA-seq + TCR-seq data and need to quantify
  whether a multimodal latent embedding improves downstream prediction (antigen specificity,
  binding avidity from pMHC multimer counts) or cluster identification relative to
  unimodal baselines.
when_to_use_negative:
- Data are unpaired or missing one modality (e.g. gene expression without α-/β-CDR3)
  — the authors note mvTCR was applied mainly to fully paired data and handling of
  partially missing data remains insufficiently addressed
- The input is already a reduced embedding or feature table and you only need classification/clustering
  — training a new multi-view VAE is unnecessary
- No specificity/avidity or cell-type/clonotype annotations exist to train or evaluate
  prediction heads (tutorials 02/03 require such annotation)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
tools:
- name: mvTCR
  role: Multi-view variational autoencoder that fuses transcriptome and TCR (CDR3)
    into the shared cell-level embedding under evaluation; installed via pip (mvtcr==0.1.3
    for reproduction)
  repo: https://github.com/SchubertLab/mvTCR
- name: PyTorch
  role: Deep learning backend for training mvTCR and the MSLE-optimized prediction
    heads; v1.8.0 with matching CUDA for paper reproduction
- name: Scanpy
  role: Standard single-cell workflow for clustering the mvTCR embedding and running
    rank_genes_groups differential expression between antigen-specific and bystander
    clusters
- name: scirpy
  role: TCR preprocessing, required annotations, and filtering of TCR sequences before
    embedding training
- name: mvTCR_reproducibility
  role: Repository of preprocessing notebooks, experiment settings, trained models
    (Zenodo), and figure-generation notebooks to reproduce the paper's embedding evaluations
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:3c109a8c2cbef6bacb934557b47361357c8d115152a394a66625d562c0f5ac6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  - mvTCR_reproducibility
  - scirpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-multimodal-embedding-evaluation

## Summary

Evaluate a multimodal single-cell embedding (mvTCR joint TCR+transcriptome latent space) against unimodal RNA-only and TCR-only representations on downstream tasks such as pMHC binding avidity regression, antigen-specific vs bystander clustering, and reference mapping. This replaces shallow 'use the embedding' claims with quantitative comparisons (MSLE, Pearson correlation) of joint vs. unimodal representations.

## When to use

Use when you have paired scRNA-seq + TCR-seq data and need to quantify whether a multimodal latent embedding improves downstream prediction (antigen specificity, binding avidity from pMHC multimer counts) or cluster identification relative to unimodal baselines.

## When NOT to use

- Data are unpaired or missing one modality (e.g. gene expression without α-/β-CDR3) — the authors note mvTCR was applied mainly to fully paired data and handling of partially missing data remains insufficiently addressed
- The input is already a reduced embedding or feature table and you only need classification/clustering — training a new multi-view VAE is unnecessary
- No specificity/avidity or cell-type/clonotype annotations exist to train or evaluate prediction heads (tutorials 02/03 require such annotation)

## Inputs

- Paired scRNA-seq + paired TCR-seq single-cell dataset (e.g. 10x dataset >60k cells, or 103,761-cell SARS-CoV-2 PBMC cohort) in Scanpy/AnnData format
- Specificity/avidity annotations: pMHC multimer read counts (binding avidity) per cell
- Trained mvTCR model or trained joint/RNA-only/TCR-only embeddings
- IEDB TCR-epitope pairings and donor HLA types (for specificity labeling via MHCFlurry)

## Outputs

- Cell-level embeddings: joint mvTCR, RNA-only, and TCR-only representations
- Metrics table (e.g. avidity_metrics.csv) with one row per embedding type (mvTCR, RNA, TCR) and columns for MSLE and Pearson correlation
- Cluster annotations marking antigen-specific vs bystander T cell clusters with differential gene expression signatures

## How to apply

Preprocess paired scRNA-seq and TCR data in Scanpy/Scirpy following tutorials/01_preprocessing.ipynb (required annotations and filtering), then train the mvTCR multi-view VAE with PyTorch on a CUDA-capable GPU (pin mvTCR v0.1.3 + PyTorch v1.8.0 for paper reproduction, per SchubertLab/mvTCR_reproducibility; ~8GB RAM sufficed for the >60k-cell 10x dataset). Obtain three cell-level representations: joint mvTCR embedding, RNA-only embedding, and TCR-only embedding. For each representation, train a fully connected prediction head optimized with mean squared logarithmic error (MSLE) loss to regress pMHC multimer read counts per tutorials/03_training_prediction.ipynb, and evaluate MSLE and Pearson correlation on held-out cells. Decision point: the joint embedding is considered beneficial when it decreases MSLE and increases Pearson correlation relative to both unimodal baselines; for the 10x dataset the reference values were TCR MSLE 0.045, RNA MSLE 0.063. For cluster-level evaluation (e.g. SARS-CoV-2 PBMCs), cluster the joint embedding in Scanpy, label clusters via IEDB TCR-epitope queries filtered by MHCFlurry HLA-binding predictions, and verify cluster markers with Scanpy rank_genes_groups.

## Related tools

- **mvTCR** (Multi-view variational autoencoder that fuses transcriptome and TCR (CDR3) into the shared cell-level embedding under evaluation; installed via pip (mvtcr==0.1.3 for reproduction)) — https://github.com/SchubertLab/mvTCR
- **PyTorch** (Deep learning backend for training mvTCR and the MSLE-optimized prediction heads; v1.8.0 with matching CUDA for paper reproduction)
- **Scanpy** (Standard single-cell workflow for clustering the mvTCR embedding and running rank_genes_groups differential expression between antigen-specific and bystander clusters)
- **scirpy** (TCR preprocessing, required annotations, and filtering of TCR sequences before embedding training)
- **mvTCR_reproducibility** (Repository of preprocessing notebooks, experiment settings, trained models (Zenodo), and figure-generation notebooks to reproduce the paper's embedding evaluations) — https://github.com/SchubertLab/mvTCR_reproducibility

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- avidity_metrics.csv exists with three rows (mvTCR, RNA, TCR) and contains MSLE and Pearson fields
- Reported TCR MSLE falls in range 0.03–0.06 (reference 0.045) and RNA MSLE in range 0.05–0.08 (reference 0.063), consistent with manuscript values
- Joint embedding decreases MSLE (TCR: 0.045; RNA: 0.063) and increases Pearson correlation (TCR: 0.074; RNA: 0.028) relative to both unimodal baselines
- On the SARS-CoV-2 PBMC cohort, clustering reproduces five SARS-CoV-2-specific and three bystander clusters among IFN-response-enriched effector CD8+ T cells
- Marker gene directions match published signatures: KLRD1 and GZMB up in bystander clusters; CD8A down and IL7R and MT-CO genes up in specific clusters

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing data remains insufficiently addressed
- Only the CDR3 sequence is currently used for TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not yet incorporated
- Adjusting the contribution of each modality requires repeated retraining, ideally with an additional hyperparameter search, and the desired contribution is study-dependent
- Reproducibility README notes resulting models may diverge from publication models due to differing hardware/CUDA versions, and UMAP visualisation may not be fully recreated

## Evidence

- [abstract] we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [methods] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [discussion] mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction"
- [discussion] In a large-scale SARS-CoV-2 dataset, mvTCR effectively distinguished between bystander and antigen-specific T cells.: "In a large-scale SARS-CoV-2 dataset, mvTCR effectively distinguished between bystander and antigen-specific T cells."
