---
name: linear-model-tf-activity-inference
description: Use when TF activities must be inferred from a pre-processed adata object
  plus a TF binding-site matrix (rows = expressed genes, columns = TFs/motifs), e.g.
  within a fate-trajectory pipeline where downstream differential TF activity analysis
  is planned.
when_to_use_negative:
- Input is bulk RNA-seq rather than single-cell data — use ISMARA or DOROTHEA2 directly
  instead, since this skill's data-diffusion regularization was designed for single-cell
  transcriptomics
- No TF binding-site matrix (promoter regulatory sites) is available for the organism/genome
  build, since the linear model is fit against computationally predicted TF-promoter
  binding sites
- The goal is clustering or dimensionality reduction of expression (e.g. UMAP embeddings)
  rather than inference of TF regulatory activities
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Implements the linear model of gene regulation with data-diffusion regularization;
    the module is provided in fatecompass.py
  repo: https://github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Recommended for basic pre-processing of the single-cell adata object used
    as model input
  repo: https://scanpy.readthedocs.io/en/stable/
- name: ISMARA
  role: Original bulk RNA-seq linear model of TF binding-site predictions that FateCompass
    extends to single-cell data
- name: MATLAB
  role: Reproducibility scripts (tf_activities.m, bootstrapping.m) run the activity
    inference for the pancreatic endocrine datasets
  repo: https://github.com/sarajimenez/fatecompass
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:2a09c47be968e4c96b1ce46357a3506d0851c2c655e6c0ddd330ef3430abb734
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - ISMARA
  - MATLAB
  - Python
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# linear-model-tf-activity-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers transcription factor (TF) activities from single-cell transcriptomics by modeling observed gene expression as a linear combination of regulatory (promoter binding) sites and TF activities — an extension of the ISMARA linear model (Balwierz et al. 2014) to single-cell data — stabilized by a data-diffusion regularization that smooths TF activities over a k-nearest-neighbor cell graph.

## When to use

Use when TF activities must be inferred from a pre-processed adata object plus a TF binding-site matrix (rows = expressed genes, columns = TFs/motifs), e.g. within a fate-trajectory pipeline where downstream differential TF activity analysis is planned.

## When NOT to use

- Input is bulk RNA-seq rather than single-cell data — use ISMARA or DOROTHEA2 directly instead, since this skill's data-diffusion regularization was designed for single-cell transcriptomics
- No TF binding-site matrix (promoter regulatory sites) is available for the organism/genome build, since the linear model is fit against computationally predicted TF-promoter binding sites
- The goal is clustering or dimensionality reduction of expression (e.g. UMAP embeddings) rather than inference of TF regulatory activities

## Inputs

- Pre-processed single-cell adata object with cell-type ('clusters') annotations and optionally an RNA velocity field (e.g. Scanpy .h5ad)
- TF binding-site matrix (columns = TFs or motifs, rows = expressed genes)
- Candidate diffusion power values t for cross-validated regularization

## Outputs

- Regularized TF activity matrix (cells x TFs), e.g. regularized_activity_matrix.csv
- Cross-validation curve of test MSE versus diffusion power t, e.g. mse_vs_t.png
- Selected optimal diffusion power t
- Bootstrap resamples of TF activity estimates for robustness assessment

## How to apply

Load a cell-type-annotated, pre-processed adata object (prepared with Scanpy/scVelo) and the TF binding-site matrix where columns are TFs/motifs and rows are expressed genes. Build the cell-cell adjacency graph in gene-expression space, add self-loops, and row-normalize it; implement the data-diffusion regularization as the adjacency matrix raised to power t, which lets cells share information through their local neighbors and smooths the learned TF activities to control model complexity. Before fitting, sweep candidate diffusion powers t: split the promoter/gene set 80/20 into training and test sets, fit the regularized linear model on the 80% split, record test MSE per candidate t, and select the t minimizing test MSE (the optimal t is fitted by cross-validation). Then fit the full linear model of gene regulation at the selected t to obtain the TF activity matrix, and optionally bootstrap-resample to assess robustness of the activity estimates before differential motif activity analysis.

## Related tools

- **FateCompass** (Implements the linear model of gene regulation with data-diffusion regularization; the module is provided in fatecompass.py) — https://github.com/sarajimenez/fatecompass
- **Scanpy** (Recommended for basic pre-processing of the single-cell adata object used as model input) — https://scanpy.readthedocs.io/en/stable/
- **ISMARA** (Original bulk RNA-seq linear model of TF binding-site predictions that FateCompass extends to single-cell data)
- **MATLAB** (Reproducibility scripts (tf_activities.m, bootstrapping.m) run the activity inference for the pancreatic endocrine datasets) — https://github.com/sarajimenez/fatecompass

## Examples

```
import scanpy as sc, pandas as pd; adata = sc.read_h5ad('e155_preprocessed.h5ad'); bsm = pd.read_csv('binding_sites_matrix.csv', index_col=0)
```

## Evaluation signals

- Cross-validation: the selected diffusion power t is the one minimizing held-out test MSE on an 80/20 split of the promoter/gene set; the MSE-vs-t curve should show a clear minimum
- The inferred TF activity matrix has dimensions cells x TFs matching the adata cells and binding-site matrix columns
- Bootstrapping resamples should yield consistent TF activity estimates (stable activities across bootstrap resamples)
- Recovered lineage-specific/differentially active TFs should match the reference results in the FateCompass Shiny App (e.g. the 126 differentially active TFs for Veres stage 5)
- The fitted model reproduces observed expression as a linear combination of regulatory sites and TF activities with residuals consistent with regularized (smoothed) solutions

## Limitations

- TF activities are estimated using binding sites only in promoter regions; essential regulatory interactions at distal regulatory sites and long-range interactions are not considered
- Reliance on computationally predicted regulatory sites in the binding-site matrix introduces a bias on the structure of the gene-regulatory network
- Pipeline input parameters (including the diffusion power t) must be hand tuned via cross-validation; incorporating scATAC-seq-derived binding sites and appropriate regularization terms is not straightforward

## Evidence
