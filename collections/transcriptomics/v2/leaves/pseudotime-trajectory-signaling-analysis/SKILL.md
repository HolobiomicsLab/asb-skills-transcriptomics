---
name: pseudotime-trajectory-signaling-analysis
description: Use when scRNA-seq cells form continuous pseudotemporal trajectories
  (e.g. E14.5 embryonic skin dermal/epidermal states) and you need cell-state-specific
  ligand-receptor signaling rather than cluster-level communication.
when_to_use_negative:
- Cells do not form continuous trajectories or pseudotime ordering is unavailable
  — standard cluster-to-cluster CellChat analysis suffices.
- Input lacks cell-group labels or has not passed QC — CellChat requires prior cell
  clustering and quality-filtered expression data.
- The goal is spatially-resolved communication; this label-free workflow does not
  model spatial proximity (use SpatialCellChat instead).
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: R package that infers, visualizes and analyzes intercellular communication
    from scRNA-seq data, including continuous states along pseudotime trajectories
  repo: https://github.com/sqjin/CellChat
- name: createCellChat
  role: Constructs the CellChat object from filtered count matrices and metadata in
    secreted-signaling (label-free) mode
  repo: https://github.com/jinworks/CellChat
- name: computeCommunProb
  role: Computes ligand-receptor communication probabilities with the Hill-function
    mass action model and permutation significance testing (M=100, p<0.05)
  repo: https://github.com/jinworks/CellChat
- name: filterCommunication
  role: Filters the inferred communication network to retain significant interactions
  repo: https://github.com/jinworks/CellChat
- name: netAnalysis_computeCentrality
  role: Computes signaling roles (out-degree, in-degree, flow betweenness, information
    centrality) along the pseudotime trajectories
  repo: https://github.com/jinworks/CellChat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:f63f8119ff1eb54d9f7527c1cdf50b05d14525fb4f6695ff497e7ec43e23a1ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - computeCommunProb
  - createCellChat
  - filterCommunication
  - netAnalysis_computeCentrality
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pseudotime-trajectory-signaling-analysis

## Summary

Infer ligand-receptor communication along continuous cell-state pseudotime trajectories (not just discrete clusters) using CellChat, e.g. predicting 88 significant interactions across 22 signaling pathways along E14.5 embryonic mouse skin dermal/epidermal trajectories.

## When to use

Use when scRNA-seq cells form continuous pseudotemporal trajectories (e.g. E14.5 embryonic skin dermal/epidermal states) and you need cell-state-specific ligand-receptor signaling rather than cluster-level communication.

## When NOT to use

- Cells do not form continuous trajectories or pseudotime ordering is unavailable — standard cluster-to-cluster CellChat analysis suffices.
- Input lacks cell-group labels or has not passed QC — CellChat requires prior cell clustering and quality-filtered expression data.
- The goal is spatially-resolved communication; this label-free workflow does not model spatial proximity (use SpatialCellChat instead).

## Inputs

- scRNA-seq count matrices (10X Genomics, GEO accessions e.g. GSM3453537, GSM3453538)
- Cell metadata with cell group labels and pseudotime ordering
- CellChatDB mouse ligand-receptor database (secreted-signaling subset)

## Outputs

- Table of significant ligand-receptor interactions (e.g. 88 interactions, cellchat_e145_significant_interactions.csv)
- Pathway-level aggregated communication probabilities (e.g. 22 signaling pathways including WNT, ncWNT, TGFb, PDGF, NGF, FGF, SEMA3)
- Centrality scores assigning dominant senders/receivers/mediators/influencers along pseudotime

## How to apply

Download the raw count matrices (e.g. GSM3453537 and GSM3453538 from GEO) and apply QC filters (UMI count 2500-50000, genes >= 1000, mitochondrial fraction < 20%). Build a CellChat object with createCellChat using the mouse CellChatDB in secreted-signaling (label-free) mode, then identify overexpressed signaling genes across cell groups with the Wilcoxon rank sum test at significance level 0.05 via identifyOverExpressedGenes and identifyOverExpressedInteractions. Compute communication probabilities with computeCommunProb using the Hill-function mass action model (trimean-of-quartiles ensemble average expression) and call significant interactions by permutation testing with M=100 group-label shuffles at p<0.05. Filter the network with filterCommunication, aggregate to pathway level with computeCommunProbPathway, and compute signaling roles (dominant senders, receivers, mediators, influencers) with netAnalysis_computeCentrality along the pseudotime trajectories.

## Related tools

- **CellChat** (R package that infers, visualizes and analyzes intercellular communication from scRNA-seq data, including continuous states along pseudotime trajectories) — https://github.com/sqjin/CellChat
- **createCellChat** (Constructs the CellChat object from filtered count matrices and metadata in secreted-signaling (label-free) mode) — https://github.com/jinworks/CellChat
- **computeCommunProb** (Computes ligand-receptor communication probabilities with the Hill-function mass action model and permutation significance testing (M=100, p<0.05)) — https://github.com/jinworks/CellChat
- **filterCommunication** (Filters the inferred communication network to retain significant interactions) — https://github.com/jinworks/CellChat
- **netAnalysis_computeCentrality** (Computes signaling roles (out-degree, in-degree, flow betweenness, information centrality) along the pseudotime trajectories) — https://github.com/jinworks/CellChat

## Examples

```
devtools::install_github("jinworks/CellChat"); cellchat <- createCellChat(object=data, meta=meta, group.by="labels"); cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- computeCommunProb(cellchat)
```

## Evaluation signals

- Reproduction should confirm 88 significant ligand-receptor interactions across 22 signaling pathways for the E14.5 embryonic skin pseudotemporal trajectories.
- Wnt6-Fzd10/Lrp6 and Wnt6-Fzd2/Lrp6 should emerge as dominant canonical WNT pairs in the significant interaction table.
- FIB-E-derived ncWNT signaling must be present among the inferred pathways.
- All reported interactions must pass the permutation test (M=100 group-label shuffles) at p<0.05 after filterCommunication.
- The significant interaction table exports cleanly to cellchat_e145_significant_interactions.csv with expected ligand/receptor/pathway columns.

## Limitations

- Communication probability represents interaction strength and is not exactly a probability.
- mRNA levels are used to approximate protein levels since single-cell proteomic information is difficult to capture.
- Hill-function parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation.
- No ground truth is available for systematic evaluation of predicted cell-cell communication networks.

## Evidence

- [intro] CellChat infers cell-state specific signaling communications using mass action models and can handle both discrete cell groups and continuous states along pseudotime trajectories: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models, along with differential expression analysis and statistical tests on cell groups,"
- [discussion] Embryonic skin QC thresholds: "we removed the cells with the amount of UMI count less than 2500 and greater than 50000, as well as the cells with the number of genes less than 1000 and the fraction of mitochondrial counts greater"
- [discussion] Permutation significance test parameters: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [methods] createCellChat function signature: "`createCellChat(object, meta, group.by, datatype, coordinates, spatial.factors, assay, do.sparse)`"
- [methods] CellChat installation from GitHub: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
