---
name: hill-function-ligand-receptor-probability-modeling
description: Use when inferring sender-receptor communication probabilities from scRNA-seq
  cell groups and you need a mass-action probability model plus a permutation-based
  significance filter over a curated ligand-receptor-cofactor database.
when_to_use_negative:
- Input lacks cell cluster/group labels — cell clustering is a pre-requisite for CellChat
  analysis
- You need spatially resolved communication probabilities — mass-action modeling here
  is not yet validated for spatial transcriptomics integration
- Your goal requires a true probability or biologically calibrated Hill parameters
  — Kh and n lack explicit biological connections and the value represents interaction
  strength only
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellChat
  role: Reference implementation whose computeCommunProb/computeCommunProbPathway
    functions compute the mass-action Hill-function communication probabilities and
    pathway-level aggregation used as ground truth for validation
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Curated database supplying the ligand-receptor pairs, heteromeric complex
    composition, and cofactors consumed by the probability model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:b0efc0abc2d4b345085fc16cf55ed732efd14eee544406221059724a8a5ee384
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - computeCommunProb
  - computeCommunProbPathway
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hill-function-ligand-receptor-probability-modeling

## Summary

Quantifies the communication probability of each ligand-receptor interaction between cell groups with a law-of-mass-action Hill function (Kh=0.5) and flags significant interactions via a permutation test (M=100 label shuffles, p<0.05), as implemented in CellChat's computeCommunProb.

## When to use

Use when inferring sender-receptor communication probabilities from scRNA-seq cell groups and you need a mass-action probability model plus a permutation-based significance filter over a curated ligand-receptor-cofactor database.

## When NOT to use

- Input lacks cell cluster/group labels — cell clustering is a pre-requisite for CellChat analysis
- You need spatially resolved communication probabilities — mass-action modeling here is not yet validated for spatial transcriptomics integration
- Your goal requires a true probability or biologically calibrated Hill parameters — Kh and n lack explicit biological connections and the value represents interaction strength only

## Inputs

- scRNA-seq expression matrix with cell group labels (e.g. 10X Genomics mouse skin data, GEO GSE113854)
- CellChatDB ligand-receptor-cofactor database including multi-subunit complex annotations
- Optionally pre-computed differentially expressed signaling genes (Wilcoxon rank sum test, 0.05 level)

## Outputs

- Communication probability matrix over sender-receiver cell group pairs (per interaction and per pathway via computeCommunProbPathway)
- Table of significant interactions with p-values from the permutation test (e.g. communication_probabilities.csv with source, target, pathway, probability, p-value)

## How to apply

Compute the ensemble-average expression of each signaling gene per cell group using the statistically robust trimean method, aggregating multi-subunit ligand/receptor complexes by geometric mean from CellChatDB annotations. Model each interaction's probability with a Hill function under the law of mass action using default Kh=0.5 and Hill coefficient n, including agonist/antagonist cofactor modulation and the population-size scaling factor ni*nj/n^2. Assess significance with a permutation test that randomly permutes cell group labels M=100 times and retain interactions at p<0.05. Validate by running CellChat's computeCommunProb and computeCommunProbPathway on the same input and comparing probability matrices within a tight numerical tolerance.

## Related tools

- **CellChat** (Reference implementation whose computeCommunProb/computeCommunProbPathway functions compute the mass-action Hill-function communication probabilities and pathway-level aggregation used as ground truth for validation) — https://github.com/sqjin/CellChat
- **CellChatDB** (Curated database supplying the ligand-receptor pairs, heteromeric complex composition, and cofactors consumed by the probability model)

## Examples

```
cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat <- computeCommunProb(cellchat, type = "trimean", population.size = TRUE, Kh = 0.5, n = 1)
```

## Evaluation signals

- Reimplemented Hill-function probabilities match CellChat computeCommunProb outputs within a tight numerical tolerance across all cell group pairs
- Permutation-filtered significant interactions (p<0.05, M=100 shuffles) agree with CellChat's significant-interaction set
- Multi-subunit complexes reproduce CellChat's geometric-mean aggregation of subunit average expressions rather than a single-gene minimum
- Pathway-level probabilities from computeCommunProbPathway are consistent with the per-interaction matrix after summing within pathway

## Limitations

- Hill parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation of ligand-receptor interactions
- mRNA levels approximate protein levels, so the model misses protein-level regulation; communication probability represents interaction strength, not an exact probability
- Systematic evaluation lacks ground truth, and overlap-based cross-method comparisons are limited since similar methods produce similar results

## Evidence

- [discussion] Hill function under mass action: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] Permutation significance test: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Trimean ensemble average expression: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [intro] Mass action models at intro level: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models, along with differential expression analysis and statistical tests on cell groups,"
- [discussion] Parameter caveat: "Although these parameters lack explicit biological connections in our current model, the Hill function can be considered as a nonlinear approximation of the ligand-receptor interactions."
