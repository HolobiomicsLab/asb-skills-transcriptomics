---
name: lineage-marker-dynamics-analysis
description: Use when you have CellRouter subpopulations and reconstructed trajectories
  (e.g., from subpopulation 20 HSCs to neutrophil/erythroid) and need to score marker
  genes or regulators along each trajectory to confirm lineage identity and transition-specific
  dynamics.
when_to_use_negative:
- When the goal is only to detect branching structure de novo without scoring gene
  dynamics — CellRouter's flow-network reconstruction or Monocle 2/Wishbone alone
  suffice and marker scoring adds no validation value
- When trajectories come from tools that do not define subpopulations (e.g., linear
  Waterfall/Monocle 1 orderings), since transition-specific scoring here is defined
  between CellRouter subpopulations
- When cell identities are unknown a priori and no lineage marker genes can be specified
  to validate the trajectories
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellRouter
  role: Reconstructs high-resolution multi-state trajectories between subpopulations
    via flow networks and scores transition-specific gene expression and regulators
    by integrating GRNs with trajectories
  repo: https://github.com/edroaldo/cellrouter
- name: R
  role: Execution environment; required R packages listed as reshape, reshape2, pheatmap,
    tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd
- name: igraph
  role: Community detection to partition the kNN graph into subpopulations whose transitions
    the marker dynamics are scored along
- name: ggplot2
  role: Visualization of marker gene dynamics along trajectories
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:db99f95d177b276dea8d67e8c19795d1ed6376f9941b8db1cfe5ab6514a9f38d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - Rtsne
  - cccd
  - ggplot2
  - igraph
  - mclust
  - pheatmap
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# lineage-marker-dynamics-analysis

## Summary

Score lineage-specific marker gene expression dynamics along CellRouter-reconstructed multi-state trajectories (e.g., Klf1/Gata1/Gfi1b on the erythroid, Ctsg/Gfi1 on the GM, Ccl3 on the LMPP path) to validate that reconstructed cell-state transitions terminate in the expected mature lineages and to rank transition-specific regulators such as Satb1, Irf8, Notch1, and Esr1.

## When to use

Use when you have CellRouter subpopulations and reconstructed trajectories (e.g., from subpopulation 20 HSCs to neutrophil/erythroid) and need to score marker genes or regulators along each trajectory to confirm lineage identity and transition-specific dynamics.

## When NOT to use

- When the goal is only to detect branching structure de novo without scoring gene dynamics — CellRouter's flow-network reconstruction or Monocle 2/Wishbone alone suffice and marker scoring adds no validation value
- When trajectories come from tools that do not define subpopulations (e.g., linear Waterfall/Monocle 1 orderings), since transition-specific scoring here is defined between CellRouter subpopulations
- When cell identities are unknown a priori and no lineage marker genes can be specified to validate the trajectories

## Inputs

- CellRouter trajectory assignments from the flow-network reconstruction over the cccd/igraph kNN graph (source state, e.g., subpopulation 2 long-term HSCs)
- Mouse myeloid progenitor single-cell RNA-seq count data (Paul et al., Cell 2015; GSE76983 / GSE81682) with t-SNE or 3D diffusion map embeddings
- List of lineage-specific marker genes and candidate regulators (e.g., Klf1, Gata1, Gfi1b, Ctsg, Gfi1, Ccl3, Cebpe, Mxd1, Rnf10, Scna, Satb1, Irf8, Notch1, Esr1)

## Outputs

- Trajectory assignments table with per-cell lineage paths written to cellrouter_trajectories.csv
- Marker gene expression dynamics along each trajectory (erythroid, GM, LMPP, neutrophil) confirming lineage-specific upregulation patterns
- Ranked transition-specific regulators at the gene and network level (e.g., Satb1, Irf8, Notch1, Esr1 for the lymphoid trajectory)

## How to apply

After CellRouter partitions the kNN graph into subpopulations (e.g., 15 subpopulations from 3D diffusion components on BloodNet/GSE81682, or 24 on GSE76983) and its flow-network algorithm reconstructs trajectories from a designated source state such as subpopulation 2 (long-term HSCs), extract per-trajectory cell orderings and score lineage marker genes along each path. Validate the trajectory by checking that terminal subpopulations match mature lineages and that known markers show lineage-specific dynamics: Klf1, Gata1, and Gfi1b rising along erythroid; Ctsg and Gfi1 along granulocyte-monocyte; Ccl3 along LMPP. For transition specificity, confirm expected temporal ordering such as Cebpe upregulated early and Mxd1 later on the neutrophil trajectory, and Gata1 earlier than Scna during erythroid differentiation. Finally, rank transition-specific regulators (e.g., Satb1, Irf8, Notch1, Esr1 for the lymphoid trajectory) at the gene and network level using CellRouter's GRN-integrated scoring, and write assignments and dynamics to cellrouter_trajectories.csv.

## Related tools

- **CellRouter** (Reconstructs high-resolution multi-state trajectories between subpopulations via flow networks and scores transition-specific gene expression and regulators by integrating GRNs with trajectories) — https://github.com/edroaldo/cellrouter
- **R** (Execution environment; required R packages listed as reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd)
- **igraph** (Community detection to partition the kNN graph into subpopulations whose transitions the marker dynamics are scored along)
- **ggplot2** (Visualization of marker gene dynamics along trajectories)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- Each reconstructed trajectory terminates in the expected mature lineage population (e.g., erythroid, granulocyte-monocyte, LMPP) on the BloodNet/GSE81682 run
- Lineage-specific markers show correct dynamics: Gata1/Klf1 rising along erythroid, Ctsg/Gfi1 along GM, Ccl3 along LMPP
- Transition-specific timing is reproduced: Cebpe upregulated early and Mxd1 later on the neutrophil trajectory; Gata1 upregulated earlier than Scna during erythroid differentiation
- Lymphoid regulators Satb1, Irf8, Notch1, and Esr1 rank among the top transition-specific regulators at the gene and network level
- Trajectory assignments and gene dynamics table is written to cellrouter_trajectories.csv with the expected 15 (GSE81682) or 24 (GSE76983) subpopulations recovered

## Limitations

- Genes identified by CellRouter for generating engraftable HSC-like cells require thorough functional characterization that extends beyond the manuscript's scope
- CellRouter was tested only in Unix-based systems/Mac, not Windows, so downstream scoring must run in a compatible environment
- Validating trajectories depends on a priori knowledge of lineage markers; without known markers (e.g., Gata1, Cebpe, Ccl3), the scoring cannot confirm lineage identity

## Evidence

- [readme] CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions: "CellRouter integrates subpopulation identification, multi-state trajectories, and gene regulatory networks (GRNs) to provide new insights into cell-state transitions"
- [methods] Identification of genes and regulators driving transitions: "Identification of genes implicated in the dynamic biological process under study, such as differentiation, and identification of regulators driving or mediating cell-fate transitions at the gene and"
- [discussion] GRN-integrated scoring: "CellRouter addresses many of these challenges by integrating GRNs with multi-state transition trajectories and scoring schemes to capture transition-specific gene expression"
- [readme] Source class and libdir setup: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
