---
name: tcr-cdr3-sequence-preprocessing
description: Use when paired single-cell transcriptome + TCR data must be prepared
  for antigen specificity prediction or clustering, and TCR input must be restricted
  to CDR3β for fair method comparison (e.g., mvTCR vs tessa).
when_to_use_negative:
- The analysis goal requires full paired α- and β-CDR3 plus VDJ-gene encoding — the
  article notes pre-trained TCR embedding models and VDJ-gene encoding are not yet
  incorporated besides the CDR3 sequence
- Data are unpaired or partially missing TCR sequences — the article states mvTCR
  was applied mainly to paired data and handling of partially missing data remains
  insufficiently addressed
- You only need generic TCR repertoire analysis without transcriptome integration
  — tools like Scirpy or scRepertoire may suffice
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: mvTCR
  role: Multi-view VAE trained on transcriptome plus CDR3β-only TCR input to produce
    cell-level embeddings for specificity prediction and clustering
  repo: https://github.com/SchubertLab/mvTCR
- name: mvTCR_reproducibility
  role: Repository with preprocessing notebooks, experiment settings, evaluation,
    and figure code used to reproduce the benchmark; provides trained models on Zenodo
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
- name: tessa
  role: Bayesian TCR functional landscape estimation model supervised with scRNA-Seq,
    run on the identically preprocessed datasets as the comparison baseline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:71d2102e4429861b0881e3d21a9c1639f4037aae033f161f0cf4e24607e7b6de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - mvTCR
  - mvTCR_reproducibility
  - tessa
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# tcr-cdr3-sequence-preprocessing

## Summary

Preprocess single-cell TCR datasets (e.g., 10x >60k cells, Minervina) by adding cell type, clonotype, and specificity annotations and restricting the TCR input to the CDR3β sequence, so that embeddings are comparable across methods such as mvTCR and tessa.

## When to use

Use when paired single-cell transcriptome + TCR data must be prepared for antigen specificity prediction or clustering, and TCR input must be restricted to CDR3β for fair method comparison (e.g., mvTCR vs tessa).

## When NOT to use

- The analysis goal requires full paired α- and β-CDR3 plus VDJ-gene encoding — the article notes pre-trained TCR embedding models and VDJ-gene encoding are not yet incorporated besides the CDR3 sequence
- Data are unpaired or partially missing TCR sequences — the article states mvTCR was applied mainly to paired data and handling of partially missing data remains insufficiently addressed
- You only need generic TCR repertoire analysis without transcriptome integration — tools like Scirpy or scRepertoire may suffice

## Inputs

- 10x Genomics single-cell dataset (>60k cells) with paired TCR sequencing
- Minervina T cell dataset with specificity annotations
- Cell type, clonotype, and antigen specificity annotations per cell

## Outputs

- Preprocessed AnnData objects with CDR3β-only TCR input and required annotations
- Shared mvTCR embeddings used for antigen specificity prediction and clustering
- benchmark_comparison.csv with per-dataset F1-Score, NMI, average gains, and p-values

## How to apply

Load and preprocess the 10x (>60k cells) and Minervina T cell datasets following the mvTCR preprocessing notebook (tutorials/01_preprocessing.ipynb), adding the required cell type, clonotype, and antigen specificity annotations and applying the notebook's filtering. Restrict the TCR input to the CDR3β sequence only so the learned representation is comparable to clonotype/sequence-based baselines such as tessa (Zhang et al.'s Bayesian TCR functional landscape estimation model supervised with scRNA-Seq). Train mvTCR v0.1.3 with PyTorch v1.8.0 in an isolated conda environment (python 3.8.8, pip install mvtcr==0.1.3) on a GPU-enabled machine. The identical preprocessing output is fed to both methods so that atlas-query F1-Score and NMI comparisons are valid.

## Related tools

- **mvTCR** (Multi-view VAE trained on transcriptome plus CDR3β-only TCR input to produce cell-level embeddings for specificity prediction and clustering) — https://github.com/SchubertLab/mvTCR
- **mvTCR_reproducibility** (Repository with preprocessing notebooks, experiment settings, evaluation, and figure code used to reproduce the benchmark; provides trained models on Zenodo) — https://github.com/SchubertLab/mvTCR_reproducibility
- **tessa** (Bayesian TCR functional landscape estimation model supervised with scRNA-Seq, run on the identically preprocessed datasets as the comparison baseline)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- mvTCR with CDR3β-only input should outperform tessa on atlas-query mapping across all datasets, with average gains of ~23.6% F1-Score (p=4.7e-6) and ~27.7% NMI (p=4.6e-6)
- Preprocessed data must contain the cell type, clonotype, and specificity annotations required by tutorials/01_preprocessing.ipynb before model training
- Environment must match mvTCR v0.1.3 with PyTorch v1.8.0 in an isolated conda environment, matching the reproducibility repo setup
- Tutorials run on a subsampled dataset should complete within ~15 minutes and match the expected outputs provided in each notebook

## Limitations

- Only the CDR3 sequence is used for TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not yet incorporated
- Handling of partially missing TCR data is insufficiently addressed; mvTCR was mainly applied to fully paired data
- Pretrained models from different hardware/CUDA versions may diverge from published models, so trained models are provided via Zenodo (doi:10.5281/zenodo.7215447)
- MAC with M1 chip lacks several dependencies; GPU and CUDA support is heavily encouraged

## Evidence

- [other] Load and preprocess the 10x (>60k cells) and Minervina T cell datasets following the mvTCR preprocessing notebook (tutorials/01_preprocessing.ipynb), adding required cell type, clonotype, and specificity annotations.: "tutorials/01_preprocessing.ipynb : adding requiered annotation and filtering on the data."
- [methods] mvTCR trained with CDR3β-only input outperformed tessa with average improvement of 23.6% in F1-Score and 27.7% in NMI.: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [intro] Zhang et al. developed a Bayesian model called TCR functional landscape estimation supervised with scRNA-Seq analysis (tessa).: "Zhang et al. developed a Bayesian model called TCR functional landscape estimation supervised with scRNA-Seq analysis (tessa)"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: [evidence span withheld — anchor status: external_doc]
- [discussion] Only CDR3 sequence currently used; VDJ-gene encoding and pre-trained TCR embedding models could be incorporated.: "pre-trained TCR embedding models58 in combination with VDJ-gene encoding could be incorporated besides the CDR3 sequence currently used"
