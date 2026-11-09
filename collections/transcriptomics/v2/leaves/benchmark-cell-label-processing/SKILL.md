---
name: benchmark-cell-label-processing
description: Use when you need ground-truth positive/negative cell–cell interaction
  labels from scRNA-seq data (e.g. panc8 with benchmark cell labels) to supervise
  training of an interaction inference model via a majority vote of statistical L-R
  methods.
when_to_use_negative:
- You already have a validated CCI ground-truth label table — regenerating labels
  by majority vote adds redundant statistical runs.
- The scRNA-seq dataset is non-Human and lacks benchmark cell labels, since the labeled
  feature preprocessing in the README is specified for --species Human with cell labels.
- You need de novo discovery of novel interactions rather than labels for supervising
  a model — the three benchmark methods plus LRIDB only score candidate L-R pairs
  already in the database.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellChat
  role: Benchmark statistical method whose significant L-R calls are thresholded and
    included in the majority vote
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: Benchmark statistical method contributing thresholded interaction calls to
    the consensus labels
- name: SingleCellSignalR
  role: Benchmark statistical method contributing thresholded interaction calls to
    the consensus labels
- name: Seurat
  role: R package used to load the panc8 expression matrix and benchmark cell labels
- name: igraph
  role: R package used in data processing for the interaction-labeling pipeline
- name: DeepCCI
  role: Framework whose Interaction_model preprocessing (Feature.py --label_mode True)
    and Interaction_inference.py consume the consensus labels
  repo: https://github.com/JiangBioLab/DeepCCI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:ce6b73784e6321e8679e81fcb1227e8e7d16124b67dbd90413d36a8f6432ad07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellPhoneDB
  - DeepCCI
  - Python
  - R
  - Seurat
  - SingleCellSignalR
  - igraph
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# benchmark-cell-label-processing

## Summary

Reconstructs DeepCCI's ground-truth interaction labeling step: candidate ligand–receptor interactions from the shared LRIDB are scored by CellChat, CellPhoneDB, and SingleCellSignalR, thresholded at LRscore >= 0.5 or P-value <= 0.01, and combined by majority vote to produce positive/negative CCI labels that supervise the GCN interaction model.

## When to use

Use when you need ground-truth positive/negative cell–cell interaction labels from scRNA-seq data (e.g. panc8 with benchmark cell labels) to supervise training of an interaction inference model via a majority vote of statistical L-R methods.

## When NOT to use

- You already have a validated CCI ground-truth label table — regenerating labels by majority vote adds redundant statistical runs.
- The scRNA-seq dataset is non-Human and lacks benchmark cell labels, since the labeled feature preprocessing in the README is specified for --species Human with cell labels.
- You need de novo discovery of novel interactions rather than labels for supervising a model — the three benchmark methods plus LRIDB only score candidate L-R pairs already in the database.

## Inputs

- panc8 scRNA-seq expression matrix with benchmark cell labels (Seurat/R, label_mode True, species Human)
- Shared ligand–receptor interaction database (LRIDB) defining candidate L-R pairs
- Interaction probabilities from CellChat, CellPhoneDB, and SingleCellSignalR run on the same expression data

## Outputs

- cci_labels.csv: consensus positive/negative CCI label table for candidate ligand–receptor interactions, used as training labels by Interaction_inference.py

## How to apply

Load the panc8 scRNA-seq expression matrix with its benchmark cell labels in R (Seurat, label_mode True, species Human). Compute ligand–receptor interaction probabilities for every pair of cell populations as the geometric mean of ligand and receptor expression products over candidate pairs in the shared LRIDB. Run CellChat, CellPhoneDB, and SingleCellSignalR on the same expression data with the same LRIDB, then threshold each method's calls at the unified cutoff of LRscore >= 0.5 or P-value <= 0.01. Take a majority vote across the four scorers (geometric-mean scoring plus the three benchmark methods) to assign each candidate interaction a positive or negative label, and write the consensus table to cci_labels.csv for use by Interaction_inference.py. The rationale is that interactions agreed upon by multiple statistical methods are reliable enough to serve as 'true labels' of the deep learning model.

## Related tools

- **CellChat** (Benchmark statistical method whose significant L-R calls are thresholded and included in the majority vote) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (Benchmark statistical method contributing thresholded interaction calls to the consensus labels)
- **SingleCellSignalR** (Benchmark statistical method contributing thresholded interaction calls to the consensus labels)
- **Seurat** (R package used to load the panc8 expression matrix and benchmark cell labels)
- **igraph** (R package used in data processing for the interaction-labeling pipeline)
- **DeepCCI** (Framework whose Interaction_model preprocessing (Feature.py --label_mode True) and Interaction_inference.py consume the consensus labels) — https://github.com/JiangBioLab/DeepCCI

## Examples

```
cd Interaction_model && python Feature.py --label_mode True --species Human
```

## Evaluation signals

- cci_labels.csv exists and every candidate LRIDB interaction carries a binary positive/negative label.
- Each labeled interaction satisfies at least one benchmark cutoff: LRscore >= 0.5 or P-value <= 0.01 (positives) or fails the majority (negatives).
- Positive labels agree with the majority vote across the four scorers (geometric mean, CellChat, CellPhoneDB, SingleCellSignalR) on the same LRIDB.
- Feature.py was run with --label_mode True --species Human and the processed feature file reflects the labeled mode before Interaction_inference.py runs.
- The consensus label file loads without error as supervision input to Interaction_inference.py (python Interaction_inference.py --device cuda).

## Limitations

- Label quality is bounded by agreement among only three benchmark methods plus geometric-mean scoring; interactions where the methods disagree fall to the negative class.
- The labeled preprocessing path in the README assumes Human species and cell labels; other species/label modes are not covered by this labeling workflow.
- Thresholds (LRscore >= 0.5, P-value <= 0.01) are unified cutoffs applied to all methods; methods with different native scales may be inconsistently binarized.

## Evidence

- [other] Interactions passing each statistical method's threshold (LRscore ≥ 0.5 or P-value ≤ 0.01) were selected as the true labels of the model, and the majority vote across the methods was used to select the significant interactions.: [evidence span withheld — anchor status: not_found]
- [intro] DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data.: "DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data."
- [readme] devtools::install_github("sqjin/CellChat"): "devtools::install_github("sqjin/CellChat")"
- [readme] python Feature.py --label_mode True --species Human: "python Feature.py --label_mode True --species Human"
- [readme] python Interaction_inference.py --device cuda: "python Interaction_inference.py --device cuda"
