---
name: leave-one-out-cross-validation-evaluation
description: Use when you have a spatial dataset measuring a small panel of genes
  (e.g. bdtnp.txt in situ values at 3039 Drosophila embryo positions) and a matched
  scRNA-seq matrix, and you need to quantify how accurately an optimal-transport mapping
  reconstructs spatial expression of held-out genes.
when_to_use_negative:
- No spatial reference measurements of even a small gene panel exist — the optimal
  transport mapping and its cross-validation require spatial values at known positions.
- The scRNA-seq dataset is excessively large, where per the article the computation
  can become intractable and one must approximate with a small number of landmark
  positions.
- The task is cell-cell communication or signaling-range inference (spsc.spatial_signaling_ot
  / infer_signal_range_ml) rather than spatial gene-expression reconstruction scoring.
tools:
- name: SpaOTsc
  role: Initializes spatial_sc from sc_data, sc_dmat, is_dmat; solves the structured/unbalanced
    optimal transport via transport_plan(cost_matrix) to obtain gamma* used for held-out
    gene prediction
  repo: https://github.com/zcang/SpaOTsc
- name: Scanpy
  role: Selects highly variable genes from the binarized, log(1 + X)-transformed scRNA-seq
    data before building the cost matrix
- name: pandas
  role: Loads dge_normalized.txt, bdtnp.txt, geometry.txt and writes per-gene LOOCV
    metric rows to drosophila_loocv_metrics.csv
- name: scikit-learn
  role: Computes AUC and other scoring metrics against the measured in situ values
- name: numpy
  role: Computes the physical distance matrix among the 3039 spatial positions from
    geometry.txt and the weighted-average predictions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:2a1c91f3aeed8f71f05707fe39502a30a3ad0599b1baae863f8feb6adf64a36b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - Scanpy
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# leave-one-out-cross-validation-evaluation

## Summary

Evaluates SpaOTsc spatial gene-expression prediction by leave-one-out cross-validation over measured spatial genes: hold out one in situ gene, predict it at all spatial positions via the transport-plan-weighted cell average, and score against measured values with AUC, Spearman correlation (RS), and RMSE.

## When to use

Use when you have a spatial dataset measuring a small panel of genes (e.g. bdtnp.txt in situ values at 3039 Drosophila embryo positions) and a matched scRNA-seq matrix, and you need to quantify how accurately an optimal-transport mapping reconstructs spatial expression of held-out genes.

## When NOT to use

- No spatial reference measurements of even a small gene panel exist — the optimal transport mapping and its cross-validation require spatial values at known positions.
- The scRNA-seq dataset is excessively large, where per the article the computation can become intractable and one must approximate with a small number of landmark positions.
- The task is cell-cell communication or signaling-range inference (spsc.spatial_signaling_ot / infer_signal_range_ml) rather than spatial gene-expression reconstruction scoring.

## Inputs

- dge_normalized.txt — Drosophila embryo numerical scRNA-seq matrix (Dream Single cell Transcriptomics Challenge, Synapse ID syn15665609)
- bdtnp.txt — spatial in situ hybridization gene-expression matrix
- geometry.txt — spatial coordinates of the 3039 embryo positions
- Cost matrix of gene-expression dissimilarity between cells and positions
- Cell-cell dissimilarity matrix (sc_dmat) and physical distance matrix among spatial positions (is_dmat)

## Outputs

- drosophila_loocv_metrics.csv — one row per held-out spatial gene with AUC, Spearman rank similarity (RS), and RMSE
- Optimal transport plan gamma* (cell-to-position mapping) from spsc.transport_plan(cost_matrix)

## How to apply

Load the Drosophila numerical scRNA-seq matrix (dge_normalized.txt) and spatial matrix (bdtnp.txt) from the Dream Single cell Transcriptomics Challenge (Synapse syn15665609), with the 3039 spatial positions' coordinates from geometry.txt. Binarize the scRNA-seq data, apply elementwise log(1 + X), and use Scanpy to select highly variable genes. Build three input matrices — the cell-to-position gene-expression dissimilarity (cost matrix), the cell-cell dissimilarity sc_dmat, and the physical distance matrix is_dmat among spatial positions — then initialize SpaOTsc.spatial_sc and solve the structured/unbalanced optimal transport with spsc.transport_plan(cost_matrix) to obtain the mapping gamma*. For each held-out spatial gene, predict expression at every position as the transport-plan-weighted average sum_j gamma*_j,i * g_j / sum_j gamma*_j,i, and score predictions against measured in situ values using AUC, Spearman rank similarity (RS), and RMSE, writing one metric row per held-out gene to drosophila_loocv_metrics.csv.

## Related tools

- **SpaOTsc** (Initializes spatial_sc from sc_data, sc_dmat, is_dmat; solves the structured/unbalanced optimal transport via transport_plan(cost_matrix) to obtain gamma* used for held-out gene prediction) — https://github.com/zcang/SpaOTsc
- **Scanpy** (Selects highly variable genes from the binarized, log(1 + X)-transformed scRNA-seq data before building the cost matrix)
- **pandas** (Loads dge_normalized.txt, bdtnp.txt, geometry.txt and writes per-gene LOOCV metric rows to drosophila_loocv_metrics.csv)
- **scikit-learn** (Computes AUC and other scoring metrics against the measured in situ values)
- **numpy** (Computes the physical distance matrix among the 3039 spatial positions from geometry.txt and the weighted-average predictions)

## Examples

```
spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- Computed LOOCV metrics should match the reported D. Em. results — AUC 0.876, Spearman RS 0.495 (binarized), RMSE 0.225 — within a reasonable tolerance.
- Predictions for the held-out gene must be transport-plan-weighted averages sum_j gamma*_j,i * g_j / sum_j gamma*_j,i over cells, not raw scRNA-seq values.
- SpaOTsc should outperform the comparison baselines DistMap and Achim et al. on the same Drosophila embryo splits.
- The metrics CSV should contain one AUC/RS/RMSE row per held-out spatial gene, and the transport plan must be solved from the cost matrix built on Scanpy-selected highly variable genes.

## Limitations

- The computation can become intractable when the dataset is excessively large, so effective approximation may be needed by only using a small number of landmark positions.
- Validation depends on the exact preprocessing (binarization of scRNA-seq data and highly-variable-gene selection); deviating from the tutorial pipeline can shift metrics away from AUC 0.876 / RS 0.495 / RMSE 0.225.
- The mapping of a cell-position pair depends on not only the gene expression profile similarity between this pair but also the mapping of all other pairs, so the transport plan must be re-derived consistently for the full evaluation.

## Evidence

- [abstract] Four datasets are employed for cross-validation of spatial gene expression prediction and comparison to known cell-cell communications: "Four datasets are employed for cross-validation of spatial gene expression prediction and comparison to known cell–cell communications."
- [methods] OT mapping construction: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [methods] Preprocessing of scRNA-seq: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
- [methods] Prediction at each position: "(1) prediction of spatial gene expression at the ith position by"
- [readme] SpaOTsc initialization API: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
