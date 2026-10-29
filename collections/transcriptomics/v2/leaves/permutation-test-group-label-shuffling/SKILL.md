---
name: permutation-test-group-label-shuffling
description: Use when filtering inferred ligand-receptor interactions in scRNA-seq
  cell-cell communication analysis, i.e., after computing communication probabilities
  and needing a nonparametric null model to call interactions significant at p < 0.05.
when_to_use_negative:
- When the goal is differential expression or pathway enrichment testing rather than
  filtering pairwise cell-cell communication probabilities — the permutation applies
  specifically to cell group labels in the communication probability model
- When communication probabilities have not yet been computed; the null model must
  use the same mass-action model parameters (type, trim, population.size, Kh, n) as
  the forward calculation
- When analyzing spatially resolved communication at single-cell resolution without
  discrete group labels to permute — use Spatial CellChat approaches instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: Reference implementation providing computeCommunProb and computeCommunProbPathway
    for both the mass-action probability model and the permutation significance test;
    used as ground truth for numerical comparison
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Curated ligand-receptor-cofactor database supplying the interaction set (including
    multi-subunit complexes) whose probabilities are permuted and tested
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:6625095d9112e2c95f56dd9f1fc01b609283dc053be38f6e90e73c0fc80ec9ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - computeCommunProb
  - computeCommunProbPathway
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# permutation-test-group-label-shuffling

## Summary

CellChat's permutation-based significance test for ligand-receptor interactions: cell group labels are randomly shuffled 100 times to recompute communication probabilities under the null, retaining interactions with p < 0.05. This is the statistical filter that turns mass-action probability scores into a validated interaction set.

## When to use

Use when filtering inferred ligand-receptor interactions in scRNA-seq cell-cell communication analysis, i.e., after computing communication probabilities and needing a nonparametric null model to call interactions significant at p < 0.05.

## When NOT to use

- When the goal is differential expression or pathway enrichment testing rather than filtering pairwise cell-cell communication probabilities — the permutation applies specifically to cell group labels in the communication probability model
- When communication probabilities have not yet been computed; the null model must use the same mass-action model parameters (type, trim, population.size, Kh, n) as the forward calculation
- When analyzing spatially resolved communication at single-cell resolution without discrete group labels to permute — use Spatial CellChat approaches instead

## Inputs

- Normalized scRNA-seq expression matrix (e.g., 10X Genomics mouse skin data from GEO GSE113854)
- Pre-computed cell group labels per cell (metadata used both for observed probabilities and for shuffling)
- CellChatDB ligand-receptor-cofactor database annotations
- Observed mass-action communication probability matrix (Hill function, Kh=0.5)

## Outputs

- Permutation p-value for each source-target ligand-receptor pair (M=100 shuffles)
- Filtered significant-interaction set at p < 0.05
- communication_probabilities.csv with columns: source-target pair, pathway, probability, p-value

## How to apply

After computing ligand-receptor communication probabilities with the law-of-mass-action Hill-function model (Kh=0.5, trimean ensemble averages, geometric-mean aggregation for multi-subunit complexes), assess significance by randomly permuting the group labels of cells across the dataset — CellChat's default is M=100 permutations — and recalculating the interaction probability for each shuffle to build an empirical null distribution. Call an interaction significant when its observed probability exceeds the permuted null at p < 0.05. When reimplementing, mirror the computeCommunProb argument surface (type, trim, population.size, Kh, n) so the null model matches the forward model, including the population.size scaling factor ni*nj/n^2. Validate by running CellChat's computeCommunProb and computeCommunProbPathway on the same input and checking that the permutation-filtered significant-interaction set agrees with CellChat's at p < 0.05. Write the agreement-checked table (source-target pair, pathway, probability, p-value) to communication_probabilities.csv.

## Related tools

- **CellChat** (Reference implementation providing computeCommunProb and computeCommunProbPathway for both the mass-action probability model and the permutation significance test; used as ground truth for numerical comparison) — https://github.com/sqjin/CellChat
- **CellChatDB** (Curated ligand-receptor-cofactor database supplying the interaction set (including multi-subunit complexes) whose probabilities are permuted and tested)

## Examples

```
R: cellchat <- computeCommunProb(cellchat, type = "triMean", population.size = TRUE, trim = NULL)  # then inspect cellchat@opt$pvalues for M=100 permutation p < 0.05 filtering
```

## Evaluation signals

- Reimplemented communication probabilities match CellChat computeCommunProb outputs within a tight numerical tolerance across all cell group pairs
- Permutation-filtered significant interactions at p < 0.05 agree with CellChat's significant-interaction set
- Every retained interaction has a p-value computed from M=100 label shuffles (empirical null of size 100)
- communication_probabilities.csv contains exactly the four required columns (source-target pair, pathway, probability, p-value) with no missing significant pairs
- Interactions failing the p < 0.05 threshold are excluded from the final probability table

## Limitations

- Communication probability represents interaction strength only and is not exactly a probability
- Mass action parameters Kh and n lack explicit biological connections; the Hill function is used only as a nonlinear approximation
- mRNA levels are used to approximate protein levels, since ligand-receptor interactions and law-of-mass-action happen at the protein level
- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth

## Evidence

- [discussion] The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] Law of mass action modeling: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] Trimean ensemble average: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [discussion] Probability is interaction strength not probability: "The communication probability here only represents the interaction strength and is not exactly a probability."
- [discussion] Hill parameters lack biological connection: "Although these parameters lack explicit biological connections in our current model, the Hill function can be considered as a nonlinear approximation of the ligand-receptor interactions."
