---
name: target-perturbed-network-in-silico-perturbation
description: Use when you have untreated diseased single-cell expression data plus
  a known drug target gene (e.g. SLC6A4 for fluoxetine) and need to rank which cell
  cluster is most drug-responsive without paired treated samples.
when_to_use_negative:
- 'The drug acts on multiple targets with target-to-target interactions: multi-target
  perturbation requires specifically tailored methodologies not yet developed in scRank'
- No prior knowledge of a drug target gene exists and the drug mechanism of action
  is unknown without candidate targets to prioritize
- The goal requires disease-treatment paired datasets analysis of differential expression
  rather than inference from untreated samples — scRank is designed specifically for
  untreated diseased single-cell data
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: scRank
  role: Core R package providing CreateScRank, Constr_net, rank_celltype, init_mod,
    plot_net, plot_dim, scRank_GSEA and plot_drug_function for target-perturbed GRN
    construction and cell-type ranking
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: Dependent R package required for tensor operations during GRN reconstruction
    in Constr_net
  repo: https://github.com/rikenbit/rTensor
- name: devtools
  role: R package used to install scRank and rTensor from GitHub
- name: tensorly
  role: Python module integrated inside the Constr_net() function for GRN reconstruction
- name: Seurat
  role: Provides the input Seurat object with count matrix and cell-type metadata
    consumed by CreateScRank
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:27f10c7256029c1e2f64bcf3d0a7299307ad3381348d49681dc89f45a3ff567c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - Seurat
  - devtools
  - rTensor
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# target-perturbed-network-in-silico-perturbation

## Summary

Rank drug-responsive cell types from untreated scRNA-seq by modeling an in silico drug perturbation as deletion (inhibitor) or activation (agonist) of the drug-target gene node within a target-perturbed gene regulatory network (tpGRN), implemented in the R package scRank.

## When to use

Use when you have untreated diseased single-cell expression data plus a known drug target gene (e.g. SLC6A4 for fluoxetine) and need to rank which cell cluster is most drug-responsive without paired treated samples.

## When NOT to use

- The drug acts on multiple targets with target-to-target interactions: multi-target perturbation requires specifically tailored methodologies not yet developed in scRank
- No prior knowledge of a drug target gene exists and the drug mechanism of action is unknown without candidate targets to prioritize
- The goal requires disease-treatment paired datasets analysis of differential expression rather than inference from untreated samples — scRank is designed specifically for untreated diseased single-cell data

## Inputs

- Seurat object of untreated diseased scRNA-seq/snRNA-seq counts with a cell-type label column (e.g. untreated MDD dorsolateral prefrontal cortex from GEO GSE144136)
- Drug target gene symbol(s) (e.g. SLC6A4, the serotonin transporter target of fluoxetine)
- Drug perturbation type flag: 'antagonist' (inhibitor) or 'agonist'
- Species identifier (e.g. 'human')

## Outputs

- Ranked table of cell types by in silico perturbation extent, stored in obj@cell_type_rank (exported e.g. as scRank_MDD_fluoxetine_ranking.csv)
- Cell-type-specific target-perturbed gene regulatory network (tpGRN) from Constr_net
- Modularized drug-target-gene subnetworks (init_mod) with module heatmaps (plot_net PNG exports) per cell type
- UMAP/dimension-reduction ranking visualization from plot_dim (PNG export)

## How to apply

Build an scRank S4 object with CreateScRank(input=<Seurat object>, species, cell_type=<label column>, target=<drug target gene>, type='antagonist'|'agonist'); for fluoxetine on MDD dorsolateral prefrontal cortex snRNA-seq (GSE144136) the target is SLC6A4 with type='antagonist'. Reconstruct the cell-type-specific target-perturbed GRN from expression profiles with Constr_net (which integrates the Python module 'tensorly'), then score the extent of in silico drug perturbation per cell type with rank_celltype and read the ranking from obj@cell_type_rank. For interpretation, modularize the drug-target-gene subnetwork with init_mod and compare module activity between top- and bottom-ranked cell types via plot_net(mode='heatmap', celltype=...); visualize rankings in dimension-reduction space with plot_dim, and optionally assess functional impact with scRank_GSEA() and plot_drug_function(). The rationale is that drug (inhibitor) perturbation is modeled as deletion of the drug target gene node in the GRN producing both global and local effects propagated via network diffusion, so cell types whose intrinsic GRN state is most distorted by that deletion are the drug-responsive ones.

## Related tools

- **scRank** (Core R package providing CreateScRank, Constr_net, rank_celltype, init_mod, plot_net, plot_dim, scRank_GSEA and plot_drug_function for target-perturbed GRN construction and cell-type ranking) — https://github.com/ZJUFanLab/scRank
- **rTensor** (Dependent R package required for tensor operations during GRN reconstruction in Constr_net) — https://github.com/rikenbit/rTensor
- **devtools** (R package used to install scRank and rTensor from GitHub)
- **tensorly** (Python module integrated inside the Constr_net() function for GRN reconstruction)
- **Seurat** (Provides the input Seurat object with count matrix and cell-type metadata consumed by CreateScRank)

## Examples

```
obj <- CreateScRank(input=seurat_obj, species='human', cell_type='celltype', target='SLC6A4', type='antagonist'); obj <- Constr_net(obj); obj <- rank_celltype(obj); obj@cell_type_rank
```

## Evaluation signals

- obj@cell_type_rank contains a complete ranking of all cell types with the expected top hit; for the MDD fluoxetine case Ex_9 is top-ranked and Inhib_5 ranks at the bottom
- Constr_net completes GRN reconstruction per cell type without error, confirming rTensor and tensorly dependencies are installed and the target gene is present in the expression matrix
- Module heatmaps from plot_net(mode='heatmap') show abnormally high activity of the drug-target-gene-containing module (module 2 with SLC6A4) in the top-ranked cell type versus the bottom-ranked one
- plot_dim renders a dimension-reduction overlay of cell-type rankings consistent with the obj@cell_type_rank table
- Ranked cell type is consistent with prior literature/known drug mechanism (as validated for medulloblastoma and major depressive disorder datasets in the paper)

## Limitations

- scRank primarily focuses on drugs acting as inhibitors; agonist modeling is a simplifying assumption that sets out-edge weights of the target gene node to their maximum possible value, and refinement awaits more single-cell agonist drug data
- Drugs with multiple targets need specifically tailored methodologies not yet developed; multi-target extension has only preliminary results
- Performance comparisons showed scDEAL and beyondcell results can be influenced by the quality and imbalance of bulk datasets, so cross-method comparisons require care
- Drug response depends on transcriptional programs associated with the drug target, so an incorrect or absent target gene undermines the tpGRN model

## Evidence
