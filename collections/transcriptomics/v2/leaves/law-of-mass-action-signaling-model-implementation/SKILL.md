---
name: law-of-mass-action-signaling-model-implementation
description: 'Use when reconstructing or validating CellChat''s computeCommunProb
  step: you have clustered scRNA-seq data, a ligand-receptor-cofactor database (CellChatDB),
  and need communication probabilities plus p-values between cell groups.'
when_to_use_negative:
- You need spatially resolved communication inference at single-cell resolution —
  CellChat v3 (SpatialCellChat) is designed for that, not the mass-action group-level
  model
- The dataset lacks cell group annotations and you have not performed cell clustering,
  which is a prerequisite for CellChat communication analysis
- Your ligand-receptor pairs come from a different curated database and you expect
  cross-method agreement — different databases confound comparisons of predicted interactions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellChat
  role: Reference R package providing computeCommunProb and computeCommunProbPathway,
    whose mass-action probability outputs and significant-interaction set serve as
    the numerical validation target for the reimplementation
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated ligand-receptor-cofactor database supplying the interaction
    definitions, multi-subunit complex composition, and cofactor (agonist/antagonist)
    annotations used by the probability model
  repo: https://github.com/sqjin/CellChat
- name: CellChat v2
  role: Current maintained version of CellChat installable via devtools; hosts the
    updated implementation of the communication probability inference
  repo: https://github.com/jinworks/CellChat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:35b4498e6303a2c1abad85ae5b1eb3e1b5929851ffebabec9777450b5778158c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChat v2
  - CellChatDB
  - computeCommunProb
  - computeCommunProbPathway
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# law-of-mass-action-signaling-model-implementation

## Summary

Implements CellChat's mass-action ligand-receptor communication probability model, combining trimean-averaged gene expression of ligands/receptors/cofactors in a Hill function, and identifies significant interactions via a permutation test that shuffles cell group labels.

## When to use

Use when reconstructing or validating CellChat's computeCommunProb step: you have clustered scRNA-seq data, a ligand-receptor-cofactor database (CellChatDB), and need communication probabilities plus p-values between cell groups.

## When NOT to use

- You need spatially resolved communication inference at single-cell resolution — CellChat v3 (SpatialCellChat) is designed for that, not the mass-action group-level model
- The dataset lacks cell group annotations and you have not performed cell clustering, which is a prerequisite for CellChat communication analysis
- Your ligand-receptor pairs come from a different curated database and you expect cross-method agreement — different databases confound comparisons of predicted interactions

## Inputs

- Pre-processed scRNA-seq expression matrix with cell group labels (e.g., 10X Genomics mouse skin data, GEO accession GSE113854)
- CellChatDB ligand-receptor-cofactor database annotations (including multi-subunit complex composition)
- Sender and receiver cell group assignments (meta.data grouping variable)

## Outputs

- Communication probability matrix between all cell group pairs (source-target)
- Permutation p-value for each inferred ligand-receptor interaction (significant at p<0.05)
- Agreement-checked probability table (source-target pair, pathway, probability, p-value), e.g., communication_probabilities.csv

## How to apply

Compute the ensemble average expression of every signaling gene per cell group using the statistically robust trimean (mean of quartiles), applying geometric-mean aggregation for multi-subunit ligand/receptor complexes from CellChatDB. Model each ligand-receptor interaction's communication probability as a Hill function under the law of mass action, with default half-maximal activation parameter Kh=0.5, Hill coefficient n, agonist/antagonist cofactor modulation terms, and cell-population-size scaling factor ni*nj/n^2 (mirroring computeCommunProb's type, trim, population.size, Kh, n arguments). Assess significance with a permutation test that randomly permutes cell group labels M=100 times and calls interactions significant at p<0.05. Validate the reimplementation by comparing its probability matrix against CellChat's computeCommunProb and computeCommunProbPathway outputs within a tight numerical tolerance.

## Related tools

- **CellChat** (Reference R package providing computeCommunProb and computeCommunProbPathway, whose mass-action probability outputs and significant-interaction set serve as the numerical validation target for the reimplementation) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated ligand-receptor-cofactor database supplying the interaction definitions, multi-subunit complex composition, and cofactor (agonist/antagonist) annotations used by the probability model) — https://github.com/sqjin/CellChat
- **CellChat v2** (Current maintained version of CellChat installable via devtools; hosts the updated implementation of the communication probability inference) — https://github.com/jinworks/CellChat

## Examples

```
cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat <- identifyAvergingFeatures(cellchat); cellchat <- computeCommunProb(cellchat, type = "trimean", population.size = TRUE)
```

## Evaluation signals

- Reimplemented communication probabilities match CellChat's computeCommunProb outputs within a tight numerical tolerance across all cell group pairs
- Permutation-filtered significant interactions (p<0.05) agree with CellChat's significant-interaction set
- Pathway-level probabilities aggregated via computeCommunProbPathway are consistent with the per-interaction probability matrix
- Output table communication_probabilities.csv has the required schema: source-target pair, pathway, probability, p-value
- Probabilities are bounded in [0, 1] and scaled by the population-size factor ni*nj/n^2 when population.size=TRUE

## Limitations

- The communication probability only represents interaction strength and is not exactly a probability
- Hill parameters Kh and n lack explicit biological connections; the Hill function is used only as a nonlinear approximation of ligand-receptor interactions
- mRNA levels are used to approximate protein levels, although ligand-receptor interactions and mass-action kinetics happen at the protein level
- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth

## Evidence

- [discussion] Hill function under law of mass action: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] Permutation significance test: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Ensemble average expression via trimean: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [intro] CellChat mass-action inference of cell-state specific signaling: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models, along with differential expression analysis and statistical tests on cell groups"
- [methods] Reference implementation installation: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
