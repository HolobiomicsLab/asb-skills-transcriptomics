---
name: target-perturbed-grn-analysis
description: Use when you need to rank which cell type in an untreated diseased single-cell
  dataset is most responsive to a drug with a known target gene, especially when no
  paired treated data is available.
when_to_use_negative:
- The drug has no known or confidently assigned target gene and the mechanism of action
  is unknown (unless using scRank's target-prioritization mode).
- You require paired disease-vs-treated data analysis (DEG or separability methods
  like Augur, beyondcell, scDEAL) rather than untreated-only inference.
- The question concerns multi-target drug perturbation with intricate target-to-target
  interactions — only preliminary results exist and tailored methods are not yet developed.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scRank
  role: 'Core package: CreateScRank, Constr_net, rank_celltype, plot_dim, plot_net,
    scRank_GSEA implement the tpGRN ranking pipeline'
  repo: https://github.com/ZJUFanLab/scRank
- name: devtools
  role: Used to install scRank from GitHub
- name: rTensor
  role: Dependent R package required for tensor operations during GRN reconstruction
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for network reconstruction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:72b2482065b0cf95e1003a64cf8143d5bf118c13451db22560655b671bce155f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - devtools
  - rTensor
  - scRank
  - tensorly
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# target-perturbed-grn-analysis

## Summary

Rank drug-responsive cell types from untreated scRNA-seq data using scRank's target-perturbed gene regulatory network (tpGRN), which models an inhibitor's drug perturbation as deletion of the target gene node and scores the resulting network distortion per cell type.

## When to use

Use when you need to rank which cell type in an untreated diseased single-cell dataset is most responsive to a drug with a known target gene, especially when no paired treated data is available.

## When NOT to use

- The drug has no known or confidently assigned target gene and the mechanism of action is unknown (unless using scRank's target-prioritization mode).
- You require paired disease-vs-treated data analysis (DEG or separability methods like Augur, beyondcell, scDEAL) rather than untreated-only inference.
- The question concerns multi-target drug perturbation with intricate target-to-target interactions — only preliminary results exist and tailored methods are not yet developed.

## Inputs

- Untreated diseased single-cell expression matrix (scRNA-seq counts, genes x cells)
- Cell type labels for each cell
- Species identifier (human/mouse)
- Known drug target gene name (searchable via `utile_database`)

## Outputs

- Ranked cell types by perturbation score (`obj@cell_type_rank`)
- Cell-type-specific target-perturbed GRNs
- Perturbation scores visualized in dimension-reduction space (plot_dim) and network views (plot_net)
- GSEA functional impact results for top-ranked cell types

## How to apply

Build the scRank S4 object from the untreated scRNA-seq expression matrix and cell type labels with `CreateScRank`, supplying the drug's known target gene via the `target` argument (for agonists, out-edge weights of the target node are set to their maximum value). Reconstruct cell-type-specific gene regulatory networks with `Constr_net` (which integrates the Python module tensorly and uses `init_mod` for modularization). Estimate the extent of in silico drug perturbation per cell type with `rank_celltype`, treating perturbation as both global and local effects via network diffusion of target-node deletion. Extract the ranking from `obj@cell_type_rank`, optionally visualize with `plot_dim`/`plot_net` and follow up with `scRank_GSEA()` and `plot_drug_function()` to assess functional impact in the top-ranked cell type.

## Related tools

- **scRank** (Core package: CreateScRank, Constr_net, rank_celltype, plot_dim, plot_net, scRank_GSEA implement the tpGRN ranking pipeline) — https://github.com/ZJUFanLab/scRank
- **devtools** (Used to install scRank from GitHub)
- **rTensor** (Dependent R package required for tensor operations during GRN reconstruction) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for network reconstruction)

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(expr, meta, species="human", target="BRD4"); Constr_net(obj); rank_celltype(obj); obj@cell_type_rank
```

## Evaluation signals

- On SERGIO-simulated ground-truth datasets (stepwise gradient, binary module, cell-type-specific module GRNs with 4-6 cell types and 100 genes), the designated target cell type must be ranked first by scRank in all three scenarios.
- In real datasets (medulloblastoma, major depressive disorder), scRank's top-ranked drug-responsive cell types should be consistent with published literature.
- The output `obj@cell_type_rank` must contain a perturbation score and rank for every input cell type.
- Simulated tpGRNs should match GRNs derived from actually treated cells (Figure S14 validation).
- The rank should be robust across different clustering methodologies (Figure S15 supplementary benchmark).

## Limitations

- scRank primarily focuses on drugs acting as inhibitors due to data availability; agonist tests are preliminary and rely on the simplifying assumption of setting target-gene out-edge weights to the maximum value.
- Multi-target drugs require specifically tailored methodologies that have not been developed; only preliminary results exist.
- Performance comparison outcomes for competing methods (beyondcell, scDEAL) are influenced by the quality and imbalance of the bulk datasets they utilized.

## Evidence

- [methods] CreateScRank method: [evidence span withheld — anchor status: not_found]
