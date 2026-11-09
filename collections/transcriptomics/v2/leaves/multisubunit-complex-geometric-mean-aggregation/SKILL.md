---
name: multisubunit-complex-geometric-mean-aggregation
description: Use when computing ligand-receptor communication probabilities with CellChatDB
  and a ligand or receptor in the interaction is annotated as a multi-subunit complex
  whose subunits each have per-cell-group expression values.
when_to_use_negative:
- The interaction involves only single-gene ligands and single-chain receptors with
  no heteromeric annotation in CellChatDB — no aggregation is needed
- Analyzing secreted signaling described only at pathway level (computeCommunProbPathway
  aggregation), where per-subunit complex composition is not the target quantity
- The expression matrix is raw counts without per-cell-group normalization; trimean
  aggregation on unnormalized counts will not match CellChat's probabilities
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellChat
  role: 'Reference implementation: its computeCommunProb function applies geometric-mean
    aggregation over multi-subunit complex subunits and the resulting probability
    serves as the numerical comparison target'
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Curated database supplying the ligand-receptor-cofactor interaction list including
    multi-subunit complex composition annotations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:79313fdf82dd0e8014d322db754d4015caea65399e12b776f28897215a0fecd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - computeCommunProb
  - computeCommunProbPathway
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Multisubunit complex geometric mean aggregation

## Summary

CellChat represents multi-subunit ligand and receptor heteromeric complexes by aggregating the expression of their subunit genes via a geometric mean, so that complex-level abundance feeds the law-of-mass-action communication probability model rather than a single-gene proxy.

## When to use

Use when computing ligand-receptor communication probabilities with CellChatDB and a ligand or receptor in the interaction is annotated as a multi-subunit complex whose subunits each have per-cell-group expression values.

## When NOT to use

- The interaction involves only single-gene ligands and single-chain receptors with no heteromeric annotation in CellChatDB — no aggregation is needed
- Analyzing secreted signaling described only at pathway level (computeCommunProbPathway aggregation), where per-subunit complex composition is not the target quantity
- The expression matrix is raw counts without per-cell-group normalization; trimean aggregation on unnormalized counts will not match CellChat's probabilities

## Inputs

- scRNA-seq expression matrix (e.g. 10X Genomics mouse skin data, GEO GSE113854) with cell group labels
- CellChatDB ligand-receptor-cofactor annotations marking multi-subunit complex composition

## Outputs

- Complex-level aggregated expression values for multi-subunit ligands and receptors per cell group pair
- Ligand-receptor communication probabilities consumed by computeCommunProb (matrix of sender-receiver probabilities)

## How to apply

First compute the ensemble average expression of every signaling gene within each cell group using the statistically robust trimean (mean of quartiles) method. Then, for ligands or receptors annotated in CellChatDB as heteromeric complexes, aggregate the trimean-normalized expression of all subunit genes with a geometric mean to obtain a single complex-level abundance value; this respects the biology that a complex requires all subunits to be present. The resulting complex-level values feed the Hill-function law-of-mass-action model (Kh=0.5, Hill coefficient n) that computes the communication probability, optionally scaled by population size factor ni*nj/n^2. Correctness is judged by reproducing CellChat's computeCommunProb probability matrix within tight numerical tolerance.

## Related tools

- **CellChat** (Reference implementation: its computeCommunProb function applies geometric-mean aggregation over multi-subunit complex subunits and the resulting probability serves as the numerical comparison target) — https://github.com/sqjin/CellChat
- **CellChatDB** (Curated database supplying the ligand-receptor-cofactor interaction list including multi-subunit complex composition annotations)

## Examples

```
cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat@DB <- CellChatDB; cellchat <- computeCommunProb(cellchat, type = "trimean", population.size = TRUE)
```

## Evaluation signals

- Reimplemented communication probabilities match CellChat computeCommunProb outputs within a tight numerical tolerance across all cell group pairs
- Permutation-filtered significant interactions (M=100 shuffles, p<0.05) agree with CellChat's significant-interaction set
- Complex aggregation returns a value bounded between the minimum and maximum subunit expression values (geometric-mean invariant)
- Complexes with any subunit at (near-)zero ensemble expression yield near-zero aggregated complex expression
- Output probability table contains the expected schema: source-target pair, pathway, probability, p-value

## Limitations

- mRNA levels are used to approximate protein levels; ligand-receptor interactions and mass-action kinetics actually happen at the protein level and single-cell proteomic information is technically difficult to capture
- Mass action parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation of ligand-receptor interactions
- Communication probability represents interaction strength and is not exactly a probability

## Evidence

- [methods] Geometric mean aggregation for multi-subunit complexes: "We construct a database of interactions among ligands, receptors and their cofactors that accurately represent known heteromeric molecular complexes."
- [discussion] Trimean ensemble average expression: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [discussion] Law of mass action model: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] Multimeric structure consideration: "CellChatDB is the first manually curated signaling interaction database in mouse that considers multimeric structure."
- [intro] Mass action models in CellChat: "CellChat infers cell-state specific signaling communications within a given scRNA-seq data using mass action models"
