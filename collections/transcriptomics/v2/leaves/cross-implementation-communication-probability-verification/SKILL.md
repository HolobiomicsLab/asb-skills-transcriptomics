---
name: cross-implementation-communication-probability-verification
description: Use when a reimplemented ligand-receptor communication probability model
  (Hill/mass-action form) must be checked against CellChat's computeCommunProb reference
  outputs to confirm numerical and statistical agreement.
when_to_use_negative:
- The goal is primary cell-cell communication inference on a new dataset rather than
  validating a reimplementation — use CellChat's own pipeline directly.
- Input lacks pre-computed cell group labels; CellChat analysis requires clustering
  as a pre-requisite.
- Cross-method comparison against non-CellChat tools, since different ligand-receptor
  databases confound the comparison.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: Reference implementation providing computeCommunProb and computeCommunProbPathway
    outputs against which the reimplementation is numerically compared
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Curated ligand-receptor-cofactor database (2,021 validated molecular interactions)
    supplying the interaction universe and multimeric complex composition used in
    probability computation
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:a569250e67901d5b1b820968b26403ad2b682936031a05060cfc6f30ac1d7665
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - computeCommunProb
  - computeCommunProbPathway
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Cross-implementation communication probability verification

## Summary

Validates an independent reimplementation of CellChat's law-of-mass-action ligand-receptor communication probability model and permutation significance test by numerically comparing inferred probabilities against CellChat's computeCommunProb outputs on the same scRNA-seq dataset.

## When to use

Use when a reimplemented ligand-receptor communication probability model (Hill/mass-action form) must be checked against CellChat's computeCommunProb reference outputs to confirm numerical and statistical agreement.

## When NOT to use

- The goal is primary cell-cell communication inference on a new dataset rather than validating a reimplementation — use CellChat's own pipeline directly.
- Input lacks pre-computed cell group labels; CellChat analysis requires clustering as a pre-requisite.
- Cross-method comparison against non-CellChat tools, since different ligand-receptor databases confound the comparison.

## Inputs

- 10X Genomics mouse skin wound scRNA-seq count matrix from GEO accession GSE113854
- Pre-computed cell group labels/metadata for the scRNA-seq dataset
- CellChatDB ligand-receptor-cofactor database (with multimeric complex composition and cofactor annotations)
- Reference CellChat R package (computeCommunProb, computeCommunProbPathway)

## Outputs

- communication_probabilities.csv — agreement-checked table of source-target pair, pathway, probability, and p-value
- Reimplemented mass-action communication probability matrix
- Permutation-test significance calls (M=100 shuffles, p<0.05) for comparison against CellChat's significant-interaction set

## How to apply

Load the 10X Genomics mouse skin scRNA-seq test dataset (GEO accession GSE113854) with pre-computed cell group labels and the CellChatDB ligand-receptor-cofactor database. Compute the ensemble-average expression of signaling genes per cell group using the statistically robust trimean (mean of quartiles) method, and aggregate multi-subunit ligand/receptor complexes by geometric mean per CellChatDB annotations. Implement the mass-action communication probability as a Hill function with the default half-maximal activation parameter Kh=0.5 and Hill coefficient n, including agonist/antagonist cofactor modulation terms and the cell-population-size scaling factor ni*nj/n^2, mirroring the computeCommunProb argument surface (type, trim, population.size, Kh, n). Assess significance via a permutation test that randomly shuffles cell group labels M=100 times and calls interactions significant at p<0.05. Run the reference CellChat package (computeCommunProb and computeCommunProbPathway) on the same data, then compare the reimplemented probability matrix against CellChat's within a tight numerical tolerance and check that permutation-filtered significant interactions agree with CellChat's significant set at p<0.05. Write the agreement-checked table (source-target pair, pathway, probability, p-value) to communication_probabilities.csv.

## Related tools

- **CellChat** (Reference implementation providing computeCommunProb and computeCommunProbPathway outputs against which the reimplementation is numerically compared) — https://github.com/sqjin/CellChat
- **CellChatDB** (Curated ligand-receptor-cofactor database (2,021 validated molecular interactions) supplying the interaction universe and multimeric complex composition used in probability computation) — https://github.com/sqjin/CellChat

## Examples

```
cellchat <- createCellChat(object=data, meta=meta, group.by='labels'); cellchat <- computeCommunProb(cellchat, type='triMean', population.size=TRUE, Kh=0.5, nboot=100); cellchat <- computeCommunProbPathway(cellchat)
```

## Evaluation signals

- Reimplemented communication probabilities match CellChat's computeCommunProb outputs within a tight numerical tolerance across all cell group pairs
- Permutation-filtered significant interactions at p<0.05 agree with CellChat's significant-interaction set
- Ensemble-average expression reproduces the trimean (mean of quartiles) statistic and geometric-mean aggregation for multi-subunit complexes matches CellChatDB annotations
- Probability table schema contains source-target pair, pathway, probability, and p-value columns with no missing pairs relative to the reference matrix
- Permutation test uses exactly M=100 random group-label shuffles matching the CellChat default

## Limitations

- Hill parameters Kh and n lack explicit biological connections; the Hill function is used only as a nonlinear approximation of ligand-receptor interactions, so the reimplementation inherits arbitrary parameter choices
- The 'communication probability' only represents interaction strength and is not exactly a probability, so tolerance checks should be against interaction-strength values, not probabilistic guarantees
- mRNA levels are used to approximate protein levels, which are the true substrate of mass-action binding; this approximation error propagates into both implementations equally
- No ground truth exists for systematic evaluation of predicted cell-cell communication networks, so agreement is only cross-implementation, not accuracy verification

## Evidence

- [methods] Mass action Hill model with permutation test: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [methods] Permutation test parameters: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Trimean ensemble average: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [intro] Mass action models in CellChat: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models, along with differential expression analysis and statistical tests on cell groups,"
- [discussion] Probability is not a true probability: "The communication probability here only represents the interaction strength and is not exactly a probability."
- [discussion] Hill parameters lack biological grounding: "Although these parameters lack explicit biological connections in our current model, the Hill function can be considered as a nonlinear approximation of the ligand-receptor interactions."
