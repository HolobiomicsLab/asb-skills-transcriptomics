---
name: drug-target-gene-module-visualization
description: Use when scRank has ranked cell types for a drug target (e.g., Smo for
  vismodegib, SLC6A4 for fluoxetine) and you need to see which perturbed gene modules
  — especially the target-containing module — differ between top- and bottom-ranked
  cell types.
when_to_use_negative:
- Only the cell-type ranking is needed — obj@cell_type_rank from rank_celltype() already
  answers which cell type responds; module inspection adds mechanistic detail, not
  the ranking itself
- The drug is an agonist or has multiple targets — scRank's agonist handling is a
  simplifying assumption (out-edge weights set to maximum) and multi-target perturbation
  has only preliminary results (Figure S17)
- No tpGRN exists yet — init_mod requires the gene regulatory network reconstructed
  by Constr_net() from the expression profiles for the target gene
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: Provides init_mod() to modularize the drug-target-gene subnetwork and plot_net(mode='heatmap',
    celltype=...) to render perturbed gene module heatmaps per cell type
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: Dependent R package that must be installed for scRank (and thus its init_mod/plot_net
    workflow) to run
  repo: https://github.com/rikenbit/rTensor
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:6d3aaf607b6deb3422f342d4da6f1a376030866227a67d732d6fe6aa1e028d56
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
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# drug-target-gene-module-visualization

## Summary

Renders scRank's drug-target-gene module views: init_mod modularizes the target-perturbed gene regulatory network (tpGRN) and plot_net(mode='heatmap') displays perturbed gene module activity per cell type, exposing the transcriptional modules (e.g., Smo-Cnbp in Node_B; module-2 containing SLC6A4 in Ex_9) that underlie in silico drug responsiveness.

## When to use

Use when scRank has ranked cell types for a drug target (e.g., Smo for vismodegib, SLC6A4 for fluoxetine) and you need to see which perturbed gene modules — especially the target-containing module — differ between top- and bottom-ranked cell types.

## When NOT to use

- Only the cell-type ranking is needed — obj@cell_type_rank from rank_celltype() already answers which cell type responds; module inspection adds mechanistic detail, not the ranking itself
- The drug is an agonist or has multiple targets — scRank's agonist handling is a simplifying assumption (out-edge weights set to maximum) and multi-target perturbation has only preliminary results (Figure S17)
- No tpGRN exists yet — init_mod requires the gene regulatory network reconstructed by Constr_net() from the expression profiles for the target gene

## Inputs

- scRank S4 object created with CreateScRank(), with the tpGRN built by Constr_net() for the target gene and ranking computed by rank_celltype() into obj@cell_type_rank
- Cell-type-labeled single-cell expression data underlying the object (e.g., vehicle-treated medulloblastoma scRNA-seq; untreated MDD DLPFC snRNA-seq from GSE144136)
- Drug target gene symbol ('Smo' for vismodegib, 'SLC6A4' for fluoxetine)

## Outputs

- Initialized modularization of the drug-target-gene subnetwork (from init_mod)
- Per-cell-type perturbed gene module heatmap plots from plot_net(mode='heatmap', celltype=...), e.g., exported as PNG
- Comparison of target-gene-containing module activity between top-ranked (drug-responsive) and bottom-ranked (drug-resistant) cell types

## How to apply

After Constr_net() builds the tpGRN for the specified drug target (e.g., 'Smo' or 'SLC6A4') and rank_celltype() populates obj@cell_type_rank, call init_mod(obj) to initialize modularization of the drug-target-gene subnetwork. Render module heatmaps per cell type with plot_net(obj, mode='heatmap', celltype=...) for the highest- and lowest-ranked types — Node_B vs Node_A for vismodegib/Smo in medulloblastoma, Ex_9 vs Inhib_5 for fluoxetine/SLC6A4 in the untreated MDD DLPFC data. Judge correctness by whether the module containing the drug target gene shows high activity in the drug-responsive cell type: the reported expectations are the Smo-Cnbp connection in Node_B (Figure 4C) and abnormal activity of module-2 containing SLC6A4 in Ex_9. The rationale is that the tpGRN models inhibitor perturbation as deletion of the drug target gene node with global and local effects, so per-cell-type module activity links the in silico perturbation score to a concrete transcriptional program. Complement with scRank_GSEA()/plot_drug_function() for functional impact in the highest-ranking cell type, and export heatmaps plus the plot_dim UMAP ranking as PNG/CSV artifacts.

## Related tools

- **scRank** (Provides init_mod() to modularize the drug-target-gene subnetwork and plot_net(mode='heatmap', celltype=...) to render perturbed gene module heatmaps per cell type) — https://github.com/ZJUFanLab/scRank
- **rTensor** (Dependent R package that must be installed for scRank (and thus its init_mod/plot_net workflow) to run) — https://github.com/rikenbit/rTensor

## Examples

```
init_mod(obj); plot_net(obj, mode = 'heatmap', celltype = 'Ex_9')
```

## Evaluation signals

- init_mod must run on an scRank object whose tpGRN was built for the stated target gene ('Smo' or 'SLC6A4') by Constr_net(); running it before network reconstruction should fail
- The heatmap of the top-ranked cell type should show high activity in the module containing the drug target gene — module-2 with SLC6A4 in Ex_9, or the Smo-Cnbp connection in Node_B (Figure 4C)
- The bottom-ranked cell type (Node_A for vismodegib; Inhib_5 for fluoxetine) should show comparatively low activity in the target-containing module
- Rendered per-cell-type module heatmaps should be consistent with the ordering stored in obj@cell_type_rank
- Exported plots must be per-cell-type and labeled with the cell type passed via plot_net(..., celltype=...)

## Limitations

- scRank primarily focuses on drugs acting as inhibitors; agonist perturbation is modeled by setting the target gene node's out-edge weights to their maximum value — a simplifying assumption
- Multi-target perturbation requires specifically tailored methodologies that are not yet developed; only preliminary results exist (Figure S17)
- Module heatmaps reflect a GRN reconstructed solely from untreated expression profiles; validation of the simulated tpGRN against GRNs from treated cells appears only in supplementary Figure S14

## Evidence
