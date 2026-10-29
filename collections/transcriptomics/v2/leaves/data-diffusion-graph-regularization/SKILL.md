---
name: data-diffusion-graph-regularization
description: Use when inferring TF activities from scRNA-seq with a binding-site linear
  model where per-cell noise would overfit the model, and cells can be smoothed through
  a kNN graph in gene-expression space.
when_to_use_negative:
- When only bulk RNA-seq data are available — the original ISMARA-style linear model
  without the diffusion term suffices; this regularization addresses per-cell noise
  in single-cell data
- When the goal is trajectory/transition-probability inference from RNA velocity or
  differentiation potential rather than the linear model of gene regulation step itself
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Implements the linear model of gene regulation with the data-diffusion regularization;
    module provided in fatecompass.py
  repo: https://github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Suggested for basic pre-processing of the single-cell adata object before
    diffusion and model fitting
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Python
  role: Language in which the full diffusion-regularized model fitting and cross-validation
    sweep are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:00290bfe5f41538c0a844004e4cc10b6751014c8c12a08c0872e0425eb0914dd
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
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# data-diffusion-graph-regularization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

FateCompass's regularization for its ISMARA-style linear model of gene regulation applied to single-cell transcriptomics: TF activities are smoothed by diffusing expression through the cell-cell kNN adjacency graph, with the diffusion power t selected by cross-validation to control model complexity and avoid overfitting.

## When to use

Use when inferring TF activities from scRNA-seq with a binding-site linear model where per-cell noise would overfit the model, and cells can be smoothed through a kNN graph in gene-expression space.

## When NOT to use

- When only bulk RNA-seq data are available — the original ISMARA-style linear model without the diffusion term suffices; this regularization addresses per-cell noise in single-cell data
- When the goal is trajectory/transition-probability inference from RNA velocity or differentiation potential rather than the linear model of gene regulation step itself

## Inputs

- Pre-processed single-cell adata object (.h5ad) with cell-type ('clusters') annotations, e.g. mouse E15.5 pancreatic endocrine differentiation data
- TF binding-site matrix (CSV/table) with columns as TFs or motifs and rows as expressed genes
- Candidate diffusion powers t (swept values for cross-validation)

## Outputs

- Regularized TF activity matrix (regularized_activity_matrix.csv) at the cross-validation-selected diffusion power t
- MSE-vs-t cross-validation selection curve (mse_vs_t.png)
- Selected optimal diffusion power t (minimum test MSE)

## How to apply

Load the pre-processed adata object (annotated 'clusters', optionally RNA velocity) with Scanpy and the TF binding-site matrix (columns = TFs/motifs, rows = expressed genes) with pandas. Build the cell-cell adjacency graph in gene-expression space, add self-loops, and row-normalize it to form the diffusion operator. For each candidate diffusion power t, raise the operator to the tth power to smooth the expression data before fitting the linear model that expresses observed gene expression as a linear combination of regulatory sites and TF activities. Split the promoter/gene set 80/20 into training and test sets, fit on the 80% split, and record test MSE per candidate t; select the t minimizing test MSE and plot the MSE-vs-t curve with matplotlib. Refit at the selected t on the full data and export the regularized TF activity matrix and the selection curve.

## Related tools

- **FateCompass** (Implements the linear model of gene regulation with the data-diffusion regularization; module provided in fatecompass.py) — https://github.com/sarajimenez/fatecompass
- **Scanpy** (Suggested for basic pre-processing of the single-cell adata object before diffusion and model fitting) — https://scanpy.readthedocs.io/en/stable/
- **Python** (Language in which the full diffusion-regularized model fitting and cross-validation sweep are implemented)

## Examples

```
P = row_normalize(A + I); best_t = argmin([test_mse(np.linalg.matrix_power(P, t) @ expr, bs_matrix, split=0.8) for t in candidates])
```

## Evaluation signals

- Test MSE on the held-out 20% of the promoter/gene set shows a clear minimum at the selected diffusion power t (check the mse_vs_t.png curve)
- The diffusion operator equals the row-normalized self-loop adjacency raised to the selected t; verify rows sum to 1 after each normalization
- The exported regularized_activity_matrix.csv has TF activities for every TF column of the binding-site matrix and every cell of the adata object
- Refit at the selected t reproduces the recorded cross-validation MSE when re-run on the same 80/20 split
- TF activity vector is smoothed relative to an undiffused (t=0 or t=1) fit — neighboring cells on the kNN graph show more similar activities

## Limitations

- The pipeline depends on input parameters (including the diffusion power t) that must be hand tuned; the authors provide tutorials to guide the parameter setup
- FateCompass relies on computationally predicted regulatory sites in a binding-site matrix, which biases the structure of the gene-regulatory network
- Only TF-promoter interactions are considered (no long-range/distal regulatory sites), which limits the accuracy of the regularized activities
- Incorporating scATAC-seq-derived binding sites and appropriate regularization terms into the linear model may not be straightforward

## Evidence
