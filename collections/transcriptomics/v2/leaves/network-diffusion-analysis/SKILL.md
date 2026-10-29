---
name: network-diffusion-analysis
description: Use when cell-type-specific GRNs are already built (e.g. via scRank Constr_net
  on untreated scRNA-seq like GSE110894) and you must rank cell types by responsiveness
  to a drug with a known target gene (e.g. Brd4).
when_to_use_negative:
- 'The drug acts as an agonist or has multiple targets: scRank primarily focuses on
  inhibitor drugs, and multi-target perturbation requires specifically tailored methodologies
  not yet developed (agonist support is a simplifying assumption setting out-edge
  weights to their maximum value).'
- No prior knowledge of the drug target exists, or the task requires disease-treatment
  paired data comparison of differential expression rather than target-based in silico
  perturbation.
- GRNs have not yet been reconstructed — run CreateScRank and Constr_net first; rank_celltype
  expects a populated S4 object.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scRank
  role: Implements rank_celltype() to build the target-perturbed GRN (antagonist edge
    deletion), run manifold alignment and n-hop network diffusion, and output cell-type
    perturbation rankings in obj@cell_type_rank
  repo: https://github.com/ZJUFanLab/scRank
- name: R
  role: Runtime environment (R >4.0) in which scRank is installed and the tpGRN scoring
    workflow is executed
- name: devtools
  role: Used to install scRank from GitHub (devtools::install_github("ZJUFanLab/scRank"))
- name: rTensor
  role: Dependent package required for scRank installation supporting tensor operations
    in network construction
  repo: https://github.com/rikenbit/rTensor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:ed97890d5ff2d6fd805cdadbd17b7fc22319ffe5400609b5065013310252d91c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - devtools
  - rTensor
  - scRank
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# network-diffusion-analysis

## Summary

Score in silico drug perturbations on cell-type-specific gene regulatory networks using scRank's rank_celltype, which removes the drug target gene's outgoing edges to form a target-perturbed GRN (tpGRN) and combines global manifold-alignment node distances with local 2-hop network diffusion to rank drug-responsive cell types.

## When to use

Use when cell-type-specific GRNs are already built (e.g. via scRank Constr_net on untreated scRNA-seq like GSE110894) and you must rank cell types by responsiveness to a drug with a known target gene (e.g. Brd4).

## When NOT to use

- The drug acts as an agonist or has multiple targets: scRank primarily focuses on inhibitor drugs, and multi-target perturbation requires specifically tailored methodologies not yet developed (agonist support is a simplifying assumption setting out-edge weights to their maximum value).
- No prior knowledge of the drug target exists, or the task requires disease-treatment paired data comparison of differential expression rather than target-based in silico perturbation.
- GRNs have not yet been reconstructed — run CreateScRank and Constr_net first; rank_celltype expects a populated S4 object.

## Inputs

- scRank S4 object with cell-type-specific GRNs built by Constr_net and an assigned drug target (e.g. Brd4)
- Untreated scRNA-seq expression data under disease condition (e.g. GSE110894 BET inhibitor resistant vs sensitive leukaemic cells)
- n_hop parameter (default 2) defining the local diffusion neighborhood

## Outputs

- Per-cell-type perturbation scores ranking drug responsiveness
- cell_type_rank.csv written from the obj@cell_type_rank slot
- Dimension-reduction and network visualizations via plot_dim / plot_net

## How to apply

Load the scRank S4 object (created with CreateScRank and populated by Constr_net) that carries the cell-type-specific GRNs, with the drug target already assigned. Call rank_celltype(obj), which by default (type = antagonist) forms the tpGRN by deleting the perturbed target gene node's outgoing edges, modeling drug (inhibitor) perturbation as target-node deletion producing both global and local effects. Quantify the global effect as manifold-alignment node distances between the untreated and target-perturbed networks, and the local effect via network diffusion over the neighborhood defined by the n_hop parameter (default 2-hop diffusion from the target gene). Combine global and local effects into a single per-cell-type perturbation score used to rank cell types. Extract the ranking from the obj@cell_type_rank slot (e.g. write to cell_type_rank.csv) and optionally visualize with plot_dim or plot_net. Optionally follow up with scRank_GSEA and plot_drug_function to assess functional impact of the drug in the highest-ranking cell types.

## Related tools

- **scRank** (Implements rank_celltype() to build the target-perturbed GRN (antagonist edge deletion), run manifold alignment and n-hop network diffusion, and output cell-type perturbation rankings in obj@cell_type_rank) — https://github.com/ZJUFanLab/scRank
- **R** (Runtime environment (R >4.0) in which scRank is installed and the tpGRN scoring workflow is executed)
- **devtools** (Used to install scRank from GitHub (devtools::install_github("ZJUFanLab/scRank")))
- **rTensor** (Dependent package required for scRank installation supporting tensor operations in network construction) — https://github.com/rikenbit/rTensor

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- rank_celltype(obj); write.csv(obj@cell_type_rank, "cell_type_rank.csv")
```

## Evaluation signals

- The S4 object's @cell_type_rank slot is populated with a finite perturbation score for every cell type and can be written non-empty to cell_type_rank.csv
- The tpGRN differs from the original GRN exactly by removal (antagonist) of the target gene node's outgoing edges
- Ranking remains robust across clustering methodologies, as the article reports adaptability in supplementary benchmark tests (Figure S15)
- Simulated tpGRN vs GRNs derived from treated cells validation (Figure S14) confirms the perturbation accurately simulates drug effect
- Top-ranked cell types should be biologically consistent with literature for the drug/disease (as shown for medulloblastoma, major depressive disorder, and tanshinone IIA-responsive macrophages in myocardial infarction)

## Limitations

- scRank primarily focuses on drugs acting as inhibitors, partly due to the nature of the available data; agonist support is preliminary and uses the simplifying assumption of setting the target gene's out-edge weights to their maximum value.
- Extension to multi-target drugs (e.g. target-to-target interactions) has only preliminary results (Figure S17) and requires tailored methodologies not yet developed.
- Validation of tpGRN simulation against GRNs from treated cells is only in supplementary Figure S14; drug resistance scoring via an integrated untreated-sample score is only in supplementary Figure S16.

## Evidence
