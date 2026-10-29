---
name: in-silico-drug-perturbation-scoring
description: Use when untreated single-cell transcriptomic data plus a known drug
  target gene must be converted into a ranking of drug-responsive cell populations,
  without disease-treatment paired samples.
when_to_use_negative:
- Paired disease-treatment datasets with drug-responsive labels exist — differential
  expression or separability-based methods (e.g., Augur) may be more appropriate since
  scRank is designed for untreated data
- The drug acts as an agonist — scRank's antagonist-mode edge deletion is the validated
  default; agonist modeling (setting out-edge weights of the target gene to maximum
  value) is only preliminary
- The drug has multiple targets requiring target-to-target interaction modeling —
  multi-target perturbation needs specifically tailored methodologies not yet developed
  in scRank
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRank
  role: Core package providing CreateScRank, Constr_net, rank_celltype, init_mod,
    plot_net, plot_dim for tpGRN construction and perturbation scoring
  repo: https://github.com/ZJUFanLab/scRank
- name: devtools
  role: Used to install scRank from GitHub via install_github
- name: rTensor
  role: Dependent package required for scRank installation and network construction
  repo: https://github.com/rikenbit/rTensor
- name: tensorly
  role: Python module integrated in the Constr_net() function for GRN reconstruction
- name: Seurat
  role: Used to load and handle scRNA-seq expression data and metadata prior to scRank
    input
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_008
  - task_002
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:4593e05e097c6ea7081681d5f55dcba9724c15fc2264aec0d8549dff3d82b376
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# in-silico-drug-perturbation-scoring

## Summary

Scores and ranks cell types by their responsiveness to an in silico drug perturbation of the target gene in cell-type-specific gene regulatory networks, using scRank's target-perturbed GRN (tpGRN) framework on untreated scRNA-seq data.

## When to use

Use when untreated single-cell transcriptomic data plus a known drug target gene must be converted into a ranking of drug-responsive cell populations, without disease-treatment paired samples.

## When NOT to use

- Paired disease-treatment datasets with drug-responsive labels exist — differential expression or separability-based methods (e.g., Augur) may be more appropriate since scRank is designed for untreated data
- The drug acts as an agonist — scRank's antagonist-mode edge deletion is the validated default; agonist modeling (setting out-edge weights of the target gene to maximum value) is only preliminary
- The drug has multiple targets requiring target-to-target interaction modeling — multi-target perturbation needs specifically tailored methodologies not yet developed in scRank

## Inputs

- Untreated disease-condition scRNA-seq expression matrix (genes x cells)
- Cell-type metadata / labels column
- Drug target gene name (e.g., Brd4, Smo, Ctsk) or drug name resolvable via utile_database
- Species specification ('mouse' or 'human')

## Outputs

- Ranked cell-type list of drug perturbation scores in obj@cell_type_rank (e.g., exported to CSV)
- Optionally, perturbed gene module heatmaps (plot_net) and dimension-reduction rank overlays (plot_dim)
- Ranked candidate target genes when run in gene-ranking mode within a specific cell type's GRN

## How to apply

Build an S4 object with CreateScRank(), supplying the expression matrix, cell-type metadata column, species ('mouse' or 'human'), and either the drug name (resolvable via utile_database) or the known target gene (e.g., Brd4, Smo, Ctsk). Reconstruct cell-type-specific GRNs with Constr_net(), which integrates the Python module 'tensorly'. Call rank_celltype() (default type = antagonist for inhibitors) to form the tpGRN by removing the target gene node's outgoing edges, quantify the global perturbation effect via manifold-alignment node distances, add the local effect via 2-hop network diffusion (n_hop parameter) from the target gene, and combine both into a per-cell-type perturbation score. Extract the ranking from obj@cell_type_rank and optionally visualize with plot_dim() or inspect perturbed modules after init_mod() and plot_net(mode = 'heatmap'). Validation is by confirming the ground-truth target cell type ranks first (e.g., Node_B for vismodegib/Smo; M0-5 macrophages for tanshinone IIA/Ctsk).

## Related tools

- **scRank** (Core package providing CreateScRank, Constr_net, rank_celltype, init_mod, plot_net, plot_dim for tpGRN construction and perturbation scoring) — https://github.com/ZJUFanLab/scRank
- **devtools** (Used to install scRank from GitHub via install_github)
- **rTensor** (Dependent package required for scRank installation and network construction) — https://github.com/rikenbit/rTensor
- **tensorly** (Python module integrated in the Constr_net() function for GRN reconstruction)
- **Seurat** (Used to load and handle scRNA-seq expression data and metadata prior to scRank input)

## Examples

```
obj <- CreateScRank(input=expr_matrix, meta=cell_meta, cell_type='celltype', species='mouse', target='Ctsk'); Constr_net(obj); rank_celltype(obj); obj@cell_type_rank
```

## Evaluation signals

- Ground-truth target cell type is ranked first in obj@cell_type_rank across synthetic SERGIO scenarios (gradient, binary, cell-type-specific module GRNs)
- For vismodegib/Smo in medulloblastoma, Node_B is top-ranked (drug-sensitive) and Node_A is bottom-ranked (drug-resistant), matching Figure 4B
- For tanshinone IIA/Ctsk in myocardial infarction, the M0-5 macrophage subpopulation is the top-ranked cell type in obj@cell_type_rank
- Gene-ranking mode within the M0-5 GRN recovers Ctsb, Ctsd, Fcgr1, Lgmn, Cd68 as top five targets with statistically significant post-treatment downregulation
- tpGRN simulation matches GRNs derived from treated cells (Figure S14 validation)

## Limitations

- scRank primarily focuses on drugs acting as inhibitors, partly due to the nature of available data; agonist testing is preliminary only
- Agonist modeling uses a simplifying assumption: setting the weight on edges emanating from the target gene node to their maximum possible value
- Multi-target drugs need specifically tailored methodologies not yet developed; only preliminary results exist (Figure S17)
- Drug resistance is considered only via an integrated score in untreated samples (Figure S16)
- Perturbation-score disparities versus beyondcell and scDEAL may be influenced by the quality and imbalance of the bulk datasets those methods utilize

## Evidence
