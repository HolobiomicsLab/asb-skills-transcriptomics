---
name: single-cell-expression-matrix-analysis
description: Use when inferring TF activities from single-cell expression with a promoter
  binding-site linear model that is under-determined/overfits, and cell-to-cell smoothing
  over a kNN gene-expression graph is needed to control model complexity.
when_to_use_negative:
- Bulk RNA-seq or non-scRNA-seq input, where the ISMARA-style model can be applied
  without data-diffusion regularization over a cell-cell graph
- Only a single candidate t is available with no held-out promoter/gene split, so
  the diffusion step cannot be fitted by cross-validated test MSE
- Binding-site information comes from distal/long-range interactions rather than the
  promoter-only matrix FateCompass expects
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Provides the linear model of gene regulation whose data-diffusion regularization
    step is reconstructed; implemented in fatecompass.py
  repo: https://github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Basic pre-processing of the single-cell adata object and building/handling
    the cell-cell neighborhood graph in gene-expression space
  repo: https://scanpy.readthedocs.io/en/stable/
- name: ISMARA
  role: Origin of the linear model of gene regulation that FateCompass extends to
    single-cell transcriptomics with data-diffusion regularization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:6f2a2630adde5fbca19c4440b4f4f13be10567c825a865b511383d559d9d53e5
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
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-expression-matrix-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Regularizes the FateCompass/ISMARA-style linear model of gene regulation (observed expression as a linear combination of regulatory sites and TF activities) via data diffusion: TF activities are smoothed over cells on the k-nearest-neighbor graph, and the diffusion power t is selected by cross-validated test MSE.

## When to use

Use when inferring TF activities from single-cell expression with a promoter binding-site linear model that is under-determined/overfits, and cell-to-cell smoothing over a kNN gene-expression graph is needed to control model complexity.

## When NOT to use

- Bulk RNA-seq or non-scRNA-seq input, where the ISMARA-style model can be applied without data-diffusion regularization over a cell-cell graph
- Only a single candidate t is available with no held-out promoter/gene split, so the diffusion step cannot be fitted by cross-validated test MSE
- Binding-site information comes from distal/long-range interactions rather than the promoter-only matrix FateCompass expects

## Inputs

- Pre-processed single-cell AnnData (adata) object with cell-type ('clusters') annotations (e.g., mouse E15.5 pancreatic endocrine differentiation)
- TF binding-site matrix where columns are TFs/motifs and rows are expressed genes

## Outputs

- Regularized TF activity matrix (e.g., regularized_activity_matrix.csv)
- MSE-vs-t cross-validation selection curve (e.g., mse_vs_t.png)
- Optimal diffusion step t (minimizing test MSE)

## How to apply

Load the pre-processed adata object (Scanpy) and the TF binding-site matrix (columns = TFs/motifs, rows = expressed genes, pandas). Build the cell-cell adjacency graph in gene-expression space, add self-loops, and row-normalize it; implement the diffusion operator as this adjacency matrix raised to the t-th power and diffuse the expression data before fitting the linear model. Split the promoter/gene set 80/20 into train/test, fit the regularized model on the 80% split, and record test MSE for each candidate t; sweep candidate t values and select the t minimizing test MSE. Refit at the selected t on the full data and export the regularized TF activity matrix. The rationale is that FateCompass extends ISMARA — which modeled expression as a linear combination of TF binding-site predictions — to single-cell transcriptomics where the system is under-determined, and cells sharing information with local neighbors smooths the learned activities.

## Related tools

- **FateCompass** (Provides the linear model of gene regulation whose data-diffusion regularization step is reconstructed; implemented in fatecompass.py) — https://github.com/sarajimenez/fatecompass
- **Scanpy** (Basic pre-processing of the single-cell adata object and building/handling the cell-cell neighborhood graph in gene-expression space) — https://scanpy.readthedocs.io/en/stable/
- **ISMARA** (Origin of the linear model of gene regulation that FateCompass extends to single-cell transcriptomics with data-diffusion regularization)

## Examples

```
import scanpy as sc, pandas as pd; adata = sc.read_h5ad('mouse_E15.5.h5ad'); bs = pd.read_csv('binding_sites_matrix.csv', index_col=0)
```

## Evaluation signals

- The adjacency matrix used for diffusion has self-loops and is row-normalized before being raised to power t
- A held-out 20% promoter/gene split exists and test MSE is reported for every candidate t in the sweep
- The selected t is the argmin of test MSE, and the final activity matrix is refitted on the full data at that t
- The exported activity matrix has rows = cells (diffused) and columns = TFs/motifs matching the binding-site matrix columns
- The MSE-vs-t curve shows a defined minimum corresponding to the chosen t (no monotone-decreasing-only artifact indicating over/under-smoothing)

## Limitations

- TF activities are estimated using binding sites only in promoter regions; long-range (distal) regulatory interactions are not considered
- Reliance on computationally predicted regulatory sites in the binding-site matrix introduces a bias in the gene-regulatory network structure
- Pipeline input parameters (including the diffusion step) must be hand tuned; the authors provide tutorials to guide parameter setup

## Evidence
