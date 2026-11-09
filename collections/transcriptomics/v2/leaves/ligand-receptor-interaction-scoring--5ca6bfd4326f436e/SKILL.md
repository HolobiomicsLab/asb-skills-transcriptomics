---
name: ligand-receptor-interaction-scoring
description: Use when labeled ground-truth CCI interactions are needed to supervise
  training of an interaction-inference model (e.g., DeepCCI's Interaction_inference.py),
  i.e., a scRNA-seq matrix with benchmark cell labels (e.g., panc8, Human) plus a
  ligand–receptor interaction database.
when_to_use_negative:
- The workflow already provides preprocessed DeepCCI feature files (Feature.py output);
  scoring would duplicate the label-generation step.
- You have no cell-type labels and only want unsupervised cluster discovery — that
  path uses DeepCCI's Cluster model, not supervised interaction labeling.
- You lack all three benchmark methods or their thresholds and cannot form a majority
  vote; a single-method call is not the consensus labeling this skill defines.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: Framework whose interaction model consumes the consensus CCI labels produced
    by this scoring/majority-vote step
  repo: https://github.com/JiangBioLab/DeepCCI
- name: CellChat
  role: Benchmark statistical method contributing votes for significant L-R interactions
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: Benchmark statistical method contributing votes for significant L-R interactions
- name: SingleCellSignalR
  role: Benchmark statistical method contributing votes for significant L-R interactions
- name: Seurat
  role: R package used to load the panc8 expression matrix with benchmark cell labels
  repo: https://github.com/satijalab/seurat-data
- name: R
  role: Environment for data processing and running the Seurat-based label-generation
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:fc6312219bbbbf44c66c7d1abbc455a1547b27658c6855c2f3918a4c58bb142e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - CellChat
  - CellPhoneDB
  - DeepCCI
  - Python
  - R
  - Seurat
  - SingleCellSignalR
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ligand-receptor-interaction-scoring

## Summary

Score all candidate ligand–receptor interactions between cell clusters (as geometric means of ligand/receptor expression products against a shared LRIDB) and derive consensus significant-interaction labels by majority vote across CellChat, CellPhoneDB, SingleCellSignalR, and geometric-mean scoring, using the unified cutoff LRscore ≥ 0.5 or P-value ≤ 0.01. These labels supervise DeepCCI's interaction model training.

## When to use

Use when labeled ground-truth CCI interactions are needed to supervise training of an interaction-inference model (e.g., DeepCCI's Interaction_inference.py), i.e., a scRNA-seq matrix with benchmark cell labels (e.g., panc8, Human) plus a ligand–receptor interaction database.

## When NOT to use

- The workflow already provides preprocessed DeepCCI feature files (Feature.py output); scoring would duplicate the label-generation step.
- You have no cell-type labels and only want unsupervised cluster discovery — that path uses DeepCCI's Cluster model, not supervised interaction labeling.
- You lack all three benchmark methods or their thresholds and cannot form a majority vote; a single-method call is not the consensus labeling this skill defines.

## Inputs

- scRNA-seq expression matrix with benchmark cell-type labels (e.g., panc8, loaded in R/Seurat with label_mode True and species Human)
- Shared ligand–receptor interaction database (LRIDB) of candidate L-R pairs
- Per-method significance calls from CellChat, CellPhoneDB, and SingleCellSignalR (LRscores and/or P-values)

## Outputs

- cci_labels.csv: consensus positive/negative CCI label table per candidate L-R interaction, used as supervised training labels for the DeepCCI interaction model

## How to apply

Load the scRNA-seq expression matrix with its benchmark cluster labels in R/Seurat (label_mode True, species Human) and compute ligand–receptor interaction probabilities for every pair of cell populations as the geometric mean of ligand and receptor expression products over candidate pairs in the shared LRIDB. Run CellChat, CellPhoneDB, and SingleCellSignalR on the same expression data and LRIDB, then threshold each method's calls at the unified cutoff LRscore >= 0.5 or P-value <= 0.01 to mark significant interactions. Take a majority vote across the four scoring methods (geometric-mean, CellChat, CellPhoneDB, SingleCellSignalR) to assign each candidate interaction a positive or negative label. Write the consensus CCI label table to cci_labels.csv for use as training labels in Interaction_inference.py.

## Related tools

- **DeepCCI** (Framework whose interaction model consumes the consensus CCI labels produced by this scoring/majority-vote step) — https://github.com/JiangBioLab/DeepCCI
- **CellChat** (Benchmark statistical method contributing votes for significant L-R interactions) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (Benchmark statistical method contributing votes for significant L-R interactions)
- **SingleCellSignalR** (Benchmark statistical method contributing votes for significant L-R interactions)
- **Seurat** (R package used to load the panc8 expression matrix with benchmark cell labels) — https://github.com/satijalab/seurat-data
- **R** (Environment for data processing and running the Seurat-based label-generation workflow)

## Examples

```
cd Interaction_model && python Feature.py --label_mode True --species Human
```

## Evaluation signals

- Each method's calls are thresholded at the unified cutoff: significant iff LRscore >= 0.5 or P-value <= 0.01.
- The consensus label for each candidate interaction is the majority vote across four methods (geometric-mean scoring, CellChat, CellPhoneDB, SingleCellSignalR).
- Output is a binary positive/negative label table (cci_labels.csv) covering every candidate L-R pair in the shared LRIDB, with no missing candidate interactions.
- Labels are computed on the same expression data and same LRIDB as used downstream by Interaction_inference.py, so pair keys are consistent for supervised training.

## Limitations

- Only the majority-vote consensus (not individual method calls) is used as the true label, so disagreements among the three statistical methods and geometric-mean scoring suppress interactions that fewer than a majority support.
- The unified cutoff (LRscore >= 0.5 or P-value <= 0.01) is a fixed choice; interactions just outside the cutoff are labeled negative regardless of method agreement.
- Statistical methods on sparse, heterogeneous scRNA-seq data can miss latent information — the article notes current statistical strategies 'ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity', which is why these labels only supervise rather than replace the model.

## Evidence

- [methods] Interactions passing each statistical method's threshold (LRscore ≥ 0.5 or P-value ≤ 0.01) were selected as the true labels of the model, and the majority vote across the methods was used to select the significant interactions.: [evidence span withheld — anchor status: not_found]
- [full_text] Run the three benchmark methods CellChat, CellPhoneDB, and SingleCellSignalR on the same expression data with the same LRIDB.: [evidence span withheld — anchor status: not_found]
- [full_text] Write the consensus positive/negative CCI label table to cci_labels.csv for use as training labels in the interaction inference step (Interaction_inference.py).: [evidence span withheld — anchor status: not_found]
- [abstract] current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity: "current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data"
- [readme] python Feature.py --label_mode True --species Human: "python Feature.py --label_mode True --species Human"
