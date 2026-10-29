---
name: cell-type-perturbation-ranking
description: Use when you have untreated diseased scRNA-seq data with cell-type annotations
  and a known drug target gene, and need to identify which cell types are drug-responsive
  or resistant without paired treated/untreated samples.
when_to_use_negative:
- The drug acts as an agonist — scRank primarily focuses on inhibitor drugs due to
  data availability, and agonist modeling is preliminary only
- The drug has multiple targets — multi-target perturbation requires specifically
  tailored methodologies not yet developed in scRank
- You need paired disease-treatment datasets analyzed by differential expression or
  separability-based methods (e.g., Augur) rather than target-informed in silico perturbation
edam_operation: http://edamontology.org/operation_3804
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: 'Core method: builds target-perturbed GRN and ranks drug-responsive cell types
    via CreateScRank, Constr_net, rank_celltype, init_mod, plot_net, plot_dim'
  repo: https://github.com/ZJUFanLab/scRank
- name: devtools
  role: R dependency installer used to install scRank from GitHub
- name: rTensor
  role: Dependent R package required for scRank installation (tensor operations in
    GRN reconstruction)
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for gene regulatory
    network reconstruction
- name: Seurat
  role: Single-cell data handling framework used with the vehicle-treated medulloblastoma
    scRNA-seq expression data in the workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:ebee2cd6b5bb36299b5376fd67d90c4b73bcb9f1656eb7f5a7c801cc665dac47
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
  edam_operation: http://edamontology.org/operation_3804
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cell-type-perturbation-ranking

## Summary

Rank and infer drug-responsive cell populations from untreated single-cell transcriptomic data by simulating in silico drug perturbation of a target-perturbed gene regulatory network (tpGRN) with scRank, prioritizing cell types most sensitive to perturbing a known drug target gene.

## When to use

Use when you have untreated diseased scRNA-seq data with cell-type annotations and a known drug target gene, and need to identify which cell types are drug-responsive or resistant without paired treated/untreated samples.

## When NOT to use

- The drug acts as an agonist — scRank primarily focuses on inhibitor drugs due to data availability, and agonist modeling is preliminary only
- The drug has multiple targets — multi-target perturbation requires specifically tailored methodologies not yet developed in scRank
- You need paired disease-treatment datasets analyzed by differential expression or separability-based methods (e.g., Augur) rather than target-informed in silico perturbation

## Inputs

- Untreated (pre-treatment) single-cell RNA-seq expression matrix (genes x cells) with cell-type metadata annotations
- Drug target gene symbol (e.g., 'Smo' for vismodegib)
- Species specification ('mouse' or 'human')

## Outputs

- Cell-type perturbation ranking (obj@cell_type_rank), exportable as CSV (e.g., scrank_smo_celltype_rank.csv) with highest-ranked = drug-sensitive and lowest-ranked = drug-resistant cell types
- Target-perturbed gene module heatmaps per cell type (via plot_net(mode = 'heatmap'))
- Dimension-reduction visualization of ranked cell types (via plot_dim())

## How to apply

Build a scRank S4 object from the expression matrix and cell-type metadata using CreateScRank(), specifying species ('mouse' or 'human') and the drug target gene (e.g., target = 'Smo' for the Smo inhibitor vismodegib). Reconstruct the gene regulatory network from expression profiles with Constr_net() (which integrates the Python module 'tensorly' and requires the rTensor dependency). Score the in silico target perturbation per cell type with rank_celltype() using the default antagonist mode for inhibitor drugs, and extract the ranking from obj@cell_type_rank. For inhibitors, the drug perturbation is modeled as deletion of the drug target gene node in the GRN, producing both global and local network effects. Optionally initialize modularization with init_mod() and render target-perturbed gene module heatmaps with plot_net(), and visualize the ranking in dimension-reduction space with plot_dim().

## Related tools

- **scRank** (Core method: builds target-perturbed GRN and ranks drug-responsive cell types via CreateScRank, Constr_net, rank_celltype, init_mod, plot_net, plot_dim) — https://github.com/ZJUFanLab/scRank
- **devtools** (R dependency installer used to install scRank from GitHub)
- **rTensor** (Dependent R package required for scRank installation (tensor operations in GRN reconstruction)) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for gene regulatory network reconstruction)
- **Seurat** (Single-cell data handling framework used with the vehicle-treated medulloblastoma scRNA-seq expression data in the workflow)

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(expr, meta = celltype_meta, species = 'mouse', target = 'Smo'); obj <- Constr_net(obj); obj <- rank_celltype(obj); write.csv(obj@cell_type_rank, 'scrank_smo_celltype_rank.csv')
```

## Evaluation signals

- The extracted obj@cell_type_rank places the known drug-sensitive cell type (e.g., medulloblastoma Node_B) at the highest perturbation score and the drug-resistant type (Node_A) at the lowest, matching the reported Figure 4B prioritization
- The tpGRN heatmap (plot_net mode='heatmap') reproduces documented target-gene connections, e.g., the Smo-Cnbp connection in Node_B (Figure 4C)
- Installation completed with dependencies 'devtools' and 'rTensor' before running CreateScRank
- Antagonist mode (default) was used for inhibitor drugs such as vismodegib, consistent with the drug's mechanism of action
- Identified drug-responsive cell types are consistent with the literature/original study, as demonstrated on medulloblastoma and major depressive disorder datasets

## Limitations

- scRank primarily focuses on drugs acting as inhibitors; agonist capability is preliminary and uses the simplifying assumption of setting out-edge weights of the target gene node to their maximum value
- Multi-target drugs are not yet supported with tailored methodologies; only preliminary results exist (Figure S17)
- Performance comparisons with beyondcell and scDEAL show disparities, with lowest performances partly influenced by the quality and imbalance of bulk datasets used
- The integrated drug-resistance score in untreated samples and adaptability across clustering methodologies are only shown in supplementary benchmark tests (Figures S15–S16)

## Evidence
