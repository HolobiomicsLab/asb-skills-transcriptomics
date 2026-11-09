---
name: geometric-mean-expression-scoring
description: 'Use when ground-truth (supervised) CCI labels must be constructed for
  training a cell-cell interaction model: candidates come from a shared LRIDB over
  clustered scRNA-seq data (e.g. panc8 with Human labels, label_mode True) and no
  trusted label file exists yet.'
when_to_use_negative:
- A trusted ground-truth CCI label file already exists — pass it directly with label_mode
  True rather than re-deriving labels by majority vote.
- Running DeepCCI in label-free mode (label_mode False) — there are no cluster labels
  to score interactions against, so supervised label construction is inapplicable.
- A single-method analysis (e.g. only CellChat) is the goal — this skill specifically
  aggregates four scorers by majority vote, which requires all benchmark method calls.
tools:
- name: DeepCCI
  role: Supplies the geometric-mean L-R expression scoring over the shared LRIDB and
    the Feature.py/Interaction_inference.py pipeline that consumes the consensus labels
  repo: https://github.com/JiangBioLab/DeepCCI
- name: CellChat
  role: Benchmark statistical method contributing one vote to the majority consensus
    on significant interactions
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: Benchmark statistical method contributing one vote to the majority consensus
    on significant interactions
- name: SingleCellSignalR
  role: Benchmark statistical method contributing one vote to the majority consensus
    on significant interactions
- name: Seurat
  role: R package used to load the panc8 expression matrix and benchmark cell labels
    before scoring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:063677422928a849223ad8ccd416815158c77f7a7a1e91299f8c7cad54255ae2
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# geometric-mean-expression-scoring

## Summary

Score candidate ligand–receptor pairs from a shared LRIDB as the geometric mean of ligand and receptor expression products across cell-cluster pairs, then threshold (LRscore ≥ 0.5 or P-value ≤ 0.01) and majority-vote with CellChat, CellPhoneDB, and SingleCellSignalR calls to produce consensus positive/negative CCI labels that supervise DeepCCI's interaction model.

## When to use

Use when ground-truth (supervised) CCI labels must be constructed for training a cell-cell interaction model: candidates come from a shared LRIDB over clustered scRNA-seq data (e.g. panc8 with Human labels, label_mode True) and no trusted label file exists yet.

## When NOT to use

- A trusted ground-truth CCI label file already exists — pass it directly with label_mode True rather than re-deriving labels by majority vote.
- Running DeepCCI in label-free mode (label_mode False) — there are no cluster labels to score interactions against, so supervised label construction is inapplicable.
- A single-method analysis (e.g. only CellChat) is the goal — this skill specifically aggregates four scorers by majority vote, which requires all benchmark method calls.

## Inputs

- Preprocessed scRNA-seq expression matrix with cell-cluster labels (e.g. panc8, Human, prepared via python Feature.py --label_mode True --species Human)
- Shared ligand–receptor interaction database (LRIDB) candidate pair list
- Per-method interaction calls from CellChat, CellPhoneDB, and SingleCellSignalR (LR scores and/or P-values)

## Outputs

- cci_labels.csv — consensus positive/negative CCI label table over all candidate L-R–cluster-pair interactions
- Per-method significance masks after thresholding at LRscore ≥ 0.5 or P-value ≤ 0.01

## How to apply

Load the scRNA-seq matrix (e.g. panc8) with benchmark cell labels in R/Seurat (label_mode True, species Human), then compute ligand–receptor interaction probability for every cell-population pair as the geometric mean of ligand and receptor expression products over candidate pairs in the shared LRIDB — this is DeepCCI's base statistical scorer. In parallel run CellChat, CellPhoneDB, and SingleCellSignalR on the same expression data and LRIDB. Apply the unified cutoff (LRscore ≥ 0.5 or P-value ≤ 0.01) to each method's calls to mark significant interactions, then take a majority vote across the four scorers (geometric-mean scoring, CellChat, CellPhoneDB, SingleCellSignalR) to assign each candidate interaction a positive or negative label. Rationale: DeepCCI argues purely statistical strategies ignore latent information in sparse, heterogeneous scRNA-seq data, so consensus labels rather than any single method supervise the GCN via Interaction_inference.py.

## Related tools

- **DeepCCI** (Supplies the geometric-mean L-R expression scoring over the shared LRIDB and the Feature.py/Interaction_inference.py pipeline that consumes the consensus labels) — https://github.com/JiangBioLab/DeepCCI
- **CellChat** (Benchmark statistical method contributing one vote to the majority consensus on significant interactions) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (Benchmark statistical method contributing one vote to the majority consensus on significant interactions)
- **SingleCellSignalR** (Benchmark statistical method contributing one vote to the majority consensus on significant interactions)
- **Seurat** (R package used to load the panc8 expression matrix and benchmark cell labels before scoring)

## Examples

```
python Feature.py --label_mode True --species Human; python Interaction_inference.py --device cuda
```

## Evaluation signals

- cci_labels.csv exists and covers every candidate interaction from the shared LRIDB with a binary positive/negative label.
- Each method's calls were thresholded with the unified cutoff (LRscore ≥ 0.5 or P-value ≤ 0.01) before voting; verify per-method masks against this cutoff.
- Consensus labels agree with the majority (≥ 2–3 of the 4 scorers: geometric-mean score, CellChat, CellPhoneDB, SingleCellSignalR) on spot-checked interactions.
- The label table loads cleanly as training labels for Interaction_inference.py and the inference run completes (python Interaction_inference.py --device cuda).
- Scoring was computed on the same expression data and same LRIDB across all four methods, so votes refer to identical candidate pairs.

## Limitations

- Label quality depends on majority vote over statistical methods that DeepCCI itself criticizes for ignoring latent information in sparse, heterogeneous scRNA-seq data — noisy minority signals may be lost or spurious ones retained.
- Thresholds are a unified cutoff (LRscore ≥ 0.5 or P-value ≤ 0.01); they may not be optimal for datasets with different expression sparsity or cluster resolution.
- Requires cell-cluster labels and a species-appropriate LRIDB (e.g. species Human); the label-free workflow (label_mode False) cannot use this skill.

## Evidence

- [methods] Interactions passing each statistical method's threshold (LRscore ≥ 0.5 or P-value ≤ 0.01) were selected as the true labels of the model: [evidence span withheld — anchor status: not_found]
- [methods] Compute ligand–receptor interaction probabilities for every pair of cell populations as the geometric mean of ligand and receptor expression products over the candidate pairs in the shared LRIDB.: [evidence span withheld — anchor status: not_found]
- [methods] Take a majority vote across the methods (geometric-mean scoring, CellChat, CellPhoneDB, SingleCellSignalR) to assign each candidate interaction a positive or negative label.: [evidence span withheld — anchor status: not_found]
- [readme] python Feature.py --label_mode True --species Human: "python Feature.py --label_mode True --species Human"
- [abstract] current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity: "current computational methods based on statistical strategies ignore some key latent information contained in scRNA-seq data with high sparsity and heterogeneity"
