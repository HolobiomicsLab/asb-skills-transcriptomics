---
name: pathomics-trajectory-inference-with-eiplgraph
description: Use when you have per-sample cell-type/structure distributions from histopathology
  (e.g., .h5ad pathomics objects) and want an unsupervised disease progression ordering
  of samples plus associated morphometric changes.
when_to_use_negative:
- Data are single-cell gene expression rather than pathomics structure distributions
  — use the PILOT single-cell (e.g., myocardial infarction) workflow instead
- Samples were collected with distinct technologies or isolation methods (single nucleus
  vs. single cell, droplet vs. well based), which PILOT does not explicitly correct
  for
- You need patient labels supervised in advance — PILOT is designed for unsupervised
  sample-level analysis, unlike PhEMD/SCANCell/Multi-scale PHATE approaches that require
  labels
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2269
tools:
- name: PILOT
  role: Computes sample-level Wasserstein distances via optimal transport (emd2 from
    POT), builds diffusion-map embedding, fits the EIPLGraph principal graph, and
    tests cell-population and morphometric changes along pseudotime
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the pathomics AnnData objects (sc.read_h5ad) as input to the PILOT workflow
- name: POT library
  role: Underlying optimal-transport solver (emd2 function) used by PILOT for Wasserstein
    distance estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:44961001133d97db1936f35fbf24d21bc4296cab6dfb7f727e2bad6984f863f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pathomics-trajectory-inference-with-eiplgraph

## Summary

Reconstruct a patient-level disease progression trajectory from pathomics data by computing sample-level Wasserstein distances with PILOT, embedding them via diffusion maps, and fitting an EIPLGraph principal graph to rank samples along pseudotime.

## When to use

Use when you have per-sample cell-type/structure distributions from histopathology (e.g., .h5ad pathomics objects) and want an unsupervised disease progression ordering of samples plus associated morphometric changes.

## When NOT to use

- Data are single-cell gene expression rather than pathomics structure distributions — use the PILOT single-cell (e.g., myocardial infarction) workflow instead
- Samples were collected with distinct technologies or isolation methods (single nucleus vs. single cell, droplet vs. well based), which PILOT does not explicitly correct for
- You need patient labels supervised in advance — PILOT is designed for unsupervised sample-level analysis, unlike PhEMD/SCANCell/Multi-scale PHATE approaches that require labels

## Inputs

- Pathomics AnnData objects (.h5ad) with per-sample cell/structure cluster distributions, e.g. Kidney_IgAN_T.h5ad (tubuli) and Kidney_IgAN_G.h5ad (glomeruli)
- Sample metadata columns: sampleID, status, and clinical groups such as eGFR classes (<30 Low, 30-60 Reduced, >60 normal)
- Morphometric features per structure (glomerular and tubular sizes, diameters, distances)

## Outputs

- Sample-level EMD/Wasserstein cost matrix (adata.uns['EMD']) per compartment and combined
- Diffusion-map embedding and EIPLGraph-derived ranked samples (disease progression score)
- Cell-type importance ranking along the trajectory
- Significant morphological features fitted along pseudotime (e.g., 14 glomerular and 3 tubular features at p<0.05)

## How to apply

Load the compartment AnnData objects (e.g., Kidney_IgAN_T.h5ad tubuli and Kidney_IgAN_G.h5ad glomeruli) with scanpy, and compute sample-level Wasserstein distances using pilotpy's pl.tl.wasserstein_distance with clusters_col='Cell_type', sample_col='sampleID', status='status', data_type='Pathomics'; inspect cost matrices and EMD heatmaps with pl.pl.heatmaps. Combine compartments by summing their EMD distance matrices (adata_Com.uns['EMD'] = adata_G.uns['EMD'] + adata_T.uns['EMD']) so both glomerular and tubular structure contributions inform the trajectory. Embed the combined distance matrix with pl.pl.trajectory (diffusion map, colors ['red','blue','orange']) and fit the EIPLGraph principal graph with pl.pl.fit_pricipla_graph using source_node set to a normal-eGFR control sample (e.g., source_node=2), since EIPLGraph requires a root sample chosen to reflect the control end of the trajectory. Rank the ordered samples as the disease progression score, then run pl.tl.cell_importance to rank cell types along the trajectory and pl.tl.extract_cells_from_pathomics to map morphological features (glom_sizes, glom_diameters, glom_tuft_sizes, glom_bowman_sizes, tubule_diameters, tubule_sizes, etc.) to pseudotime, log-transforming them with pl.tl.norm_morphological_features before fitting linear/quadratic robust (Huber) models with pl.tl.morphological_features_importance.

## Related tools

- **PILOT** (Computes sample-level Wasserstein distances via optimal transport (emd2 from POT), builds diffusion-map embedding, fits the EIPLGraph principal graph, and tests cell-population and morphometric changes along pseudotime) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the pathomics AnnData objects (sc.read_h5ad) as input to the PILOT workflow)
- **POT library** (Underlying optimal-transport solver (emd2 function) used by PILOT for Wasserstein distance estimation)

## Examples

```
import pilotpy as pl, scanpy as sc; adata_T=sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad'); pl.tl.wasserstein_distance(adata_T, clusters_col='Cell_type', sample_col='sampleID', status='status', data_type='Pathomics')
```

## Evaluation signals

- The inferred trajectory should place normal-eGFR (control) samples at the start and case samples toward the end of the pseudotime ordering
- The combined EMD matrix equals the element-wise sum of the glomerular and tubular EMD matrices (adata_Com.uns['EMD'] = adata_G.uns['EMD'] + adata_T.uns['EMD'])
- Significant morphological features are recovered: ~14 significant glomerular features and 3 tubular features with p<0.05 in the linear/quadratic robust fits
- The progression score associates with eGFR groups, e.g., likelihood-ratio p=2.4e-11 against kidney failure versus p=7e-05 for the multivariate morphometric model
- pl.pl.heatmaps should render interpretable cost-matrix and EMD heatmaps; pl.pl.select_best_sil (start=0.2) can verify cluster-count choice

## Limitations

- EIPLGraph requires a root (source) sample, which was visually selected to reflect control samples rather than chosen by an objective rule — a reproducibility gap
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence, and the effect on disease-group comparison may be unclear
- Analysis of data from distinct technologies or single-cell isolation methods remains an open challenge; uniform tissue collection (similar areas, same technology) is assumed

## Evidence

- [abstract] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples.: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph: "PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph"
- [discussion] EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples.: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
- [methods] adata_T=sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad'): "adata_T=sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad')"
- [discussion] it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit.: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
