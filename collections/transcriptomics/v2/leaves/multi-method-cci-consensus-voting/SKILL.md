---
name: multi-method-cci-consensus-voting
description: Use when you need reliable positive/negative labels for candidate L-R
  interactions between cell clusters from a labeled scRNA-seq matrix (e.g. panc8)
  to supervise a deep-learning CCI model, and no single statistical method's calls
  should be trusted alone.
when_to_use_negative:
- You only have one statistical method available (e.g. CellChat alone) — the skill
  requires a majority vote across CellChat, CellPhoneDB, and SingleCellSignalR plus
  geometric-mean scoring, so a single method cannot produce consensus labels.
- Your goal is the final CCI prediction rather than training labels — use DeepCCI's
  Interaction_inference.py on a trained model instead, with these labels serving only
  as supervision.
- Data is not scRNA-seq gene-expression with assignable cell clusters, so ligand/receptor
  expression products over the LRIDB cannot be computed per cluster pair.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: Framework whose interaction model consumes the consensus cci_labels.csv as
    supervision; contributes the geometric-mean LRscore component of the vote
  repo: github.com/JiangBioLab/DeepCCI
- name: CellChat
  role: Benchmark statistical method producing one of the per-method significance
    calls in the majority vote
  repo: sqjin/CellChat
- name: CellPhoneDB
  role: Benchmark statistical method producing one of the per-method significance
    calls in the majority vote
- name: SingleCellSignalR
  role: Benchmark statistical method producing one of the per-method significance
    calls in the majority vote
- name: Seurat
  role: R package used to load the panc8 expression matrix with its benchmark cell
    labels before computing LR probabilities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:e4e38f131c60ab088ab5313405ed55e5ef153ef6be99788199bce7091f050cb7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellChat
  - CellPhoneDB
  - DeepCCI
  - Python
  - R
  - Seurat
  - SingleCellSignalR
  - igraph
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# multi-method-cci-consensus-voting

## Summary

Assigns positive/negative ground-truth labels to candidate ligand–receptor-mediated cell–cell interactions by thresholding calls from CellChat, CellPhoneDB, and SingleCellSignalR (plus DeepCCI's geometric-mean scoring over a shared LRIDB) and taking a majority vote across methods. These consensus labels then supervise DeepCCI's GCN-based interaction model training.

## When to use

Use when you need reliable positive/negative labels for candidate L-R interactions between cell clusters from a labeled scRNA-seq matrix (e.g. panc8) to supervise a deep-learning CCI model, and no single statistical method's calls should be trusted alone.

## When NOT to use

- You only have one statistical method available (e.g. CellChat alone) — the skill requires a majority vote across CellChat, CellPhoneDB, and SingleCellSignalR plus geometric-mean scoring, so a single method cannot produce consensus labels.
- Your goal is the final CCI prediction rather than training labels — use DeepCCI's Interaction_inference.py on a trained model instead, with these labels serving only as supervision.
- Data is not scRNA-seq gene-expression with assignable cell clusters, so ligand/receptor expression products over the LRIDB cannot be computed per cluster pair.

## Inputs

- scRNA-seq expression matrix (e.g. panc8 benchmark dataset) with cell-type labels
- Shared ligand–receptor interaction database (LRIDB) listing candidate L-R pairs
- Per-method interaction results from CellChat, CellPhoneDB, and SingleCellSignalR (LRscore / P-value calls)
- Geometric-mean LR scores computed from ligand and receptor expression products

## Outputs

- cci_labels.csv: consensus positive/negative CCI label table per candidate interaction
- Per-method binary significance calls at LRscore ≥ 0.5 or P-value ≤ 0.01

## How to apply

Load the scRNA-seq expression matrix with benchmark cell labels in R/Seurat (label_mode True, species Human) and compute ligand–receptor interaction probabilities for every pair of cell populations as the geometric mean of ligand and receptor expression products over the candidate pairs in the shared LRIDB. Run the three benchmark methods CellChat, CellPhoneDB, and SingleCellSignalR on the same expression data and the same LRIDB. Threshold each method's calls at the unified cutoff of LRscore ≥ 0.5 or P-value ≤ 0.01 to mark significant interactions. Take a majority vote across the four scorers (geometric-mean scoring, CellChat, CellPhoneDB, SingleCellSignalR) to assign each candidate interaction a positive or negative label, and write the consensus table to cci_labels.csv for downstream training in Interaction_inference.py. The rationale is that no single statistical method captures all significant CCIs, so agreement across independent methods defines the true labels.

## Related tools

- **DeepCCI** (Framework whose interaction model consumes the consensus cci_labels.csv as supervision; contributes the geometric-mean LRscore component of the vote) — github.com/JiangBioLab/DeepCCI
- **CellChat** (Benchmark statistical method producing one of the per-method significance calls in the majority vote) — sqjin/CellChat
- **CellPhoneDB** (Benchmark statistical method producing one of the per-method significance calls in the majority vote)
- **SingleCellSignalR** (Benchmark statistical method producing one of the per-method significance calls in the majority vote)
- **Seurat** (R package used to load the panc8 expression matrix with its benchmark cell labels before computing LR probabilities)

## Examples

```
cd Interaction_model; python Feature.py --label_mode True --species Human; python Interaction_inference.py --device cuda
```

## Evaluation signals

- Every interaction labeled positive must satisfy the unified cutoff in at least the majority of scorers: LRscore ≥ 0.5 or P-value ≤ 0.01 from CellChat, CellPhoneDB, or SingleCellSignalR, and/or the geometric-mean LR score.
- cci_labels.csv must contain a binary positive/negative label for every candidate LR pair evaluated over the shared LRIDB, with no missing labels.
- Interaction_inference.py must run without label-schema errors, confirming the consensus table is consumable as supervision for the GCN interaction model.
- Cross-check: the fraction of consensus-positive labels should reflect agreement among multiple independent methods rather than a single tool's calls; large deviations per method suggest an incorrect thresholding step.
- The same expression matrix and LRIDB must have been used for all three benchmark methods — rerunning a method on different inputs invalidates the vote.

## Limitations

- The consensus depends on current statistical methods, which per the article 'ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity' — labels are only as good as the underlying tool calls.
- Requires three external R tools (CellChat, CellPhoneDB, SingleCellSignalR) plus Seurat/igraph installed via devtools; any method failing to run breaks the majority vote.
- The unified cutoff (LRscore ≥ 0.5 or P-value ≤ 0.01) is a fixed convention from the DeepCCI paper; datasets with different sparsity or annotation depth may need recalibration.

## Evidence

- [methods] Unified cutoff applied to each method's calls: [evidence span withheld — anchor status: not_found]
- [methods] Majority vote across three benchmark methods on the same inputs: [evidence span withheld — anchor status: not_found]
- [readme] Framework whose interaction model the labels supervise: "DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data."
- [readme] Feature preprocessing that consumes the cell labels and species setting: "cd Interaction_model
python Feature.py --label_mode True --species Human"
- [abstract] Motivation for combining multiple methods: "current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity"
