---
name: single-cell-drug-response-inference
description: Use when you have untreated disease-state scRNA-seq data with cell-type
  annotations, a known drug target gene (e.g. Smo for vismodegib, Ctsk for tanshinone
  IIA), and need to rank which cell types are most drug-responsive.
when_to_use_negative:
- The drug acts as an agonist or has multiple targets — scRank primarily focuses on
  inhibitor drugs, and agonist/multi-target extensions are preliminary only
- You require disease-treatment paired datasets and differential-expression-based
  inference rather than target-informed ranking of untreated samples
- The drug target is unknown and cannot be resolved via utile_database or candidate
  gene ranking in the cell-type-specific GRN
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: 'Core method: builds target-perturbed GRNs and ranks drug-responsive cell
    types via CreateScRank, Constr_net, rank_celltype, init_mod, plot_dim, plot_net'
  repo: https://github.com/ZJUFanLab/scRank
- name: devtools
  role: Dependency installer used to install scRank from the ZJUFanLab GitHub repository
- name: rTensor
  role: Dependent R package for tensor analysis required before installing scRank
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for GRN reconstruction
- name: beyondcell
  role: Baseline drug-response prediction method benchmarked against scRank (72.2%
    vs 77.8% mean accuracy)
- name: scDEAL
  role: Baseline drug-response prediction method benchmarked against scRank (64.8%
    vs 77.8% mean accuracy)
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:e3c2d63a27367e5cfba3dfd42cd86cf252d5a961b10892f68c1d1d10f38777fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - Seurat
  - beyondcell
  - devtools
  - rTensor
  - scDEAL
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# single-cell-drug-response-inference

## Summary

Apply scRank's target-perturbed gene regulatory network (tpGRN) approach to rank drug-responsive cell populations via in silico drug perturbation using untreated single-cell transcriptomic data, as described in Li et al., Cell Reports Medicine 2024.

## When to use

Use when you have untreated disease-state scRNA-seq data with cell-type annotations, a known drug target gene (e.g. Smo for vismodegib, Ctsk for tanshinone IIA), and need to rank which cell types are most drug-responsive.

## When NOT to use

- The drug acts as an agonist or has multiple targets — scRank primarily focuses on inhibitor drugs, and agonist/multi-target extensions are preliminary only
- You require disease-treatment paired datasets and differential-expression-based inference rather than target-informed ranking of untreated samples
- The drug target is unknown and cannot be resolved via utile_database or candidate gene ranking in the cell-type-specific GRN

## Inputs

- Untreated disease-state scRNA-seq gene expression matrix (genes x cells, e.g. from GEO accession GSE110894 or GSE163465)
- Cell metadata with a cell-type/line label column
- Drug target gene name (e.g. 'Smo', 'Ctsk') or drug name resolvable via scRank's utile_database
- Species designation ('mouse' or 'human')
- Drug mechanism-of-action choice (default antagonist for inhibitors)

## Outputs

- Ranked list of cell types by in silico perturbation score (obj@cell_type_rank)
- Cell-type-specific gene regulatory networks (tpGRN) from Constr_net
- Perturbed gene module heatmaps from plot_net / init_mod (e.g. Smo-Cnbp connection in Node_B)
- Dimension-reduction visualization of ranked cell types via plot_dim
- Ranked candidate target gene tables (e.g. scrank_tanshinone_IIA_targets.csv with Ctsb, Ctsd, Fcgr1, Lgmn, Cd68)

## How to apply

Build an scRank S4 object with CreateScRank, supplying the gene expression profile, cell-type metadata column, species ('mouse' or 'human'), and either the drug target gene or a drug name resolved via the package's utile_database; choose the antagonist mode for inhibitor drugs (default). Reconstruct the cell-type-specific gene regulatory network with Constr_net (which integrates the Python module 'tensorly' for tensor decomposition), then score in silico drug perturbation per cell type with rank_celltype and read the ranking from obj@cell_type_rank. The rationale is that inhibitor perturbation is modeled as deletion of the drug-target gene node in the GRN, producing both global and local effects, and intrinsic cellular states are reflected by cell-type-specific GRNs. Optionally modularize with init_mod and visualize perturbed modules with plot_net (e.g. reproducing the Figure 4C Smo-Cnbp connection in Node_B) or ranks in embedding space with plot_dim; use scRank_GSEA to assess functional impact on highest-ranking cell types. Validate against ground-truth sensitivity labels (e.g. resistant/sensitive leukaemic cells in GSE110894, CTRP sensitivity for Kinker cell lines) where available.

## Related tools

- **scRank** (Core method: builds target-perturbed GRNs and ranks drug-responsive cell types via CreateScRank, Constr_net, rank_celltype, init_mod, plot_dim, plot_net) — https://github.com/ZJUFanLab/scRank
- **devtools** (Dependency installer used to install scRank from the ZJUFanLab GitHub repository)
- **rTensor** (Dependent R package for tensor analysis required before installing scRank) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for GRN reconstruction)
- **beyondcell** (Baseline drug-response prediction method benchmarked against scRank (72.2% vs 77.8% mean accuracy))
- **scDEAL** (Baseline drug-response prediction method benchmarked against scRank (64.8% vs 77.8% mean accuracy))

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(input=expr, meta=metadata, cell_type="celltype", species="mouse", target="Ctsk"); Constr_net(obj); rank_celltype(obj); obj@cell_type_rank
```

## Evaluation signals

- On the GSE110894 leukaemia and Kinker cancer cell line benchmarks, scRank's top-ranked cell type should match the drug-sensitive ground truth with mean accuracy ~77.8% (vs 72.2% beyondcell, 64.8% scDEAL)
- For vismodegib medulloblastoma analysis, the ranking must place Node_B as highest-perturbation (drug-sensitive) and Node_A as lowest (drug-resistant), matching Figure 4B
- For tanshinone IIA in myocardial infarction, obj@cell_type_rank must rank the M0-5 macrophage subpopulation top, and the gene-level top five targets must be Ctsb, Ctsd, Fcgr1, Lgmn, Cd68
- Ranked target genes (e.g. Ctsb, Ctsd) should show statistically significant downregulation in treated versus untreated macrophages
- The package installation must succeed with devtools and rTensor installed first, and rank_celltype output must be accessible via obj@cell_type_rank

## Limitations

- scRank primarily focuses on drugs acting as inhibitors, partly due to the nature of the available single-cell perturbation data
- Agonist modeling uses a simplifying assumption (setting out-edge weights of the target gene node to their maximum possible value)
- Multi-target perturbation requires specifically tailored methodologies not yet developed; multi-target extension results are preliminary (Figure S17)
- Robustness across clustering methodologies and drug-resistance scoring are only shown in supplementary figures (S15, S16)

## Evidence
