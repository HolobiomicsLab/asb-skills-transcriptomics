---
name: spatial-gene-expression-prediction
description: Use when you have paired non-spatial scRNA-seq data and spatial in situ/spatial
  transcriptomics measurements of a subset of genes (e.g., Drosophila embryo bdtnp.txt
  with geometry.txt) and need to predict spatial expression of unmeasured genes or
  benchmark reconstruction accuracy by LOOCV.
when_to_use_negative:
- When the scRNA-seq dataset is excessively large — the article notes computation
  can become intractable; use landmark approximation or another method
- When the goal is to infer time-delayed cell-cell communication (ligand diffusion
  / intracellular cascade timing) — the approach does not consider time delay
- When the input already has full spatial transcriptomics per gene and no mapping
  from scRNA-seq is needed — direct spatial analysis pipelines are more appropriate
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: 'Core method: SpaOTsc.spatial_sc initialization and spsc.transport_plan(cost_matrix)
    solve the structured/unbalanced optimal transport problem yielding the cell-to-position
    mapping gamma* used for weighted spatial expression prediction'
  repo: https://github.com/zcang/SpaOTsc
- name: pandas
  role: Loads dge_normalized.txt and bdtnp.txt matrices; SpaOTsc depends on pandas
    and takes df_sc as a DataFrame with rows as cells and columns as genes
- name: numpy
  role: Provides array operations for the cost matrix, sc_dmat, is_dmat, and the transport-plan-weighted
    prediction sum_j gamma*_j,i * g_j / sum_j gamma*_j,i
- name: scipy
  role: Dependency used for dissimilarity/distance computations and statistical scoring;
    SpaOTsc depends on scipy
- name: scikit-learn
  role: Dependency supporting AUC and regression/scoring metrics for LOOCV evaluation;
    SpaOTsc depends on scikit-learn
- name: Scanpy
  role: Selects highly variable genes from the binarized, log(1+X)-transformed scRNA-seq
    data prior to building the cost matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:8bc2fcfa58d4b5c6ee683b01c4ca7eedb7a038ca1c406f546d611c02f86b561f
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spatial-gene-expression-prediction

## Summary

Predicts spatial expression of genes at tissue positions by mapping scRNA-seq cells onto spatial measurement positions via SpaOTsc's structured/unbalanced optimal transport, then computing a transport-plan-weighted average of gene expression. This enables reconstruction and leave-one-out cross-validation of spatial gene expression when only a small set of genes has spatial measurements.

## When to use

Use when you have paired non-spatial scRNA-seq data and spatial in situ/spatial transcriptomics measurements of a subset of genes (e.g., Drosophila embryo bdtnp.txt with geometry.txt) and need to predict spatial expression of unmeasured genes or benchmark reconstruction accuracy by LOOCV.

## When NOT to use

- When the scRNA-seq dataset is excessively large — the article notes computation can become intractable; use landmark approximation or another method
- When the goal is to infer time-delayed cell-cell communication (ligand diffusion / intracellular cascade timing) — the approach does not consider time delay
- When the input already has full spatial transcriptomics per gene and no mapping from scRNA-seq is needed — direct spatial analysis pipelines are more appropriate

## Inputs

- scRNA-seq gene expression matrix (dge_normalized.txt, cells x genes) as a pandas DataFrame
- Spatial gene expression matrix from in situ hybridization (bdtnp.txt, positions x genes)
- Spatial coordinates / physical distance matrix among spatial positions (geometry.txt; 3039 positions)
- Gene-expression dissimilarity cost matrix between single cells and spatial positions
- Cell-cell dissimilarity matrix (sc_dmat) within the scRNA-seq data

## Outputs

- Optimal transport plan gamma* mapping cells to spatial positions
- Predicted spatial expression of held-out genes at every spatial position
- LOOCV metrics per gene/dataset: AUC, Spearman rank similarity (RS), RMSE (written to drosophila_loocv_metrics.csv)

## How to apply

Load the Drosophila numerical scRNA-seq matrix (dge_normalized.txt) and spatial matrix (bdtnp.txt) from the Dream Single cell Transcriptomics Challenge (Synapse syn15665609) with pandas, and assign the 3039 spatial positions from geometry.txt. Binarize the scRNA-seq data, apply an elementwise log(1 + X) transformation, and use Scanpy to select highly variable genes. Build three matrices for SpaOTsc.spatial_sc: the gene-expression dissimilarity cost matrix between cells and positions, the cell-cell dissimilarity matrix (sc_dmat), and the physical distance matrix among spatial positions (is_dmat); then solve for the cell-to-position mapping with spsc.transport_plan(cost_matrix). Predict each held-out gene at position i as sum_j gamma*_j,i * g_j / sum_j gamma*_j,i (a transport-plan-weighted average over cells), looping over measured spatial genes for leave-one-out cross-validation. Score predictions against measured in situ values with AUC, Spearman rank similarity (RS), and RMSE.

## Related tools

- **SpaOTsc** (Core method: SpaOTsc.spatial_sc initialization and spsc.transport_plan(cost_matrix) solve the structured/unbalanced optimal transport problem yielding the cell-to-position mapping gamma* used for weighted spatial expression prediction) — https://github.com/zcang/SpaOTsc
- **pandas** (Loads dge_normalized.txt and bdtnp.txt matrices; SpaOTsc depends on pandas and takes df_sc as a DataFrame with rows as cells and columns as genes)
- **numpy** (Provides array operations for the cost matrix, sc_dmat, is_dmat, and the transport-plan-weighted prediction sum_j gamma*_j,i * g_j / sum_j gamma*_j,i)
- **scipy** (Dependency used for dissimilarity/distance computations and statistical scoring; SpaOTsc depends on scipy)
- **scikit-learn** (Dependency supporting AUC and regression/scoring metrics for LOOCV evaluation; SpaOTsc depends on scikit-learn)
- **Scanpy** (Selects highly variable genes from the binarized, log(1+X)-transformed scRNA-seq data prior to building the cost matrix)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- LOOCV metrics on the Drosophila embryo dataset should match the reported D. Em. results (AUC 0.876, Spearman RS 0.495, RMSE 0.225) within a reasonable tolerance
- Predicted expression should outperform DistMap and Achim et al. baselines on the same dataset
- The transport plan gamma* rows should yield valid weighted averages: sum_j gamma*_j,i is finite and non-zero at each predicted position i
- Spatial coordinates parsed from geometry.txt must yield 3039 spatial positions matching the rows of bdtnp.txt
- Output drosophila_loocv_metrics.csv should contain one row per held-out gene with AUC, RS, and RMSE columns populated

## Limitations

- Computationally intractable for excessively large datasets; cell-cell distance inference requires iterative optimal transport over all cell pairs (approximation via a small number of landmark positions is possible)
- The approach does not consider time delay in cell-cell communication (ligand diffusion time, intracellular cascade reacting time)
- No software version of SpaOTsc is stated in the article

## Evidence

- [results] SpaOTsc shows high accuracy on the Drosophila embryo dataset, achieving an AUC of 0.876 in leave-one-out cross-validation, with Spearman correlation 0.495 (binarized) and RMSE 0.225, outperforming DistMap and Achim et al.: [evidence span withheld — anchor status: not_found]
- [methods] Optimal transport solve: "spsc.transport_plan(cost_matrix)"
- [methods] Dataset provenance: "the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609)"
- [methods] Binarization and HVG selection: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
- [methods] Package dependencies: "SpaOTsc depends on the following packages: pandas, numpy, scipy, networkx, python-igraph, louvain, POT, dit, astropy, scikit-learn, matplotlib, umap-learn."
- [methods] Spatial expression prediction workflow: "prediction of spatial gene expression at the ith position by"
- [readme] Usage pattern from README: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
- [discussion] Intractability limitation: "the computation can become intractable when the dataset is excessively large"
- [discussion] No time delay modeled: "our approach does not consider the time delay that may take place in cell–cell communication"
- [methods] Prediction expression formula: [evidence span withheld — anchor status: not_found]
