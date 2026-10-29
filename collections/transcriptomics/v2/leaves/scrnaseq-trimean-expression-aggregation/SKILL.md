---
name: scrnaseq-trimean-expression-aggregation
description: Use when preparing per-cell-group average expression of ligands, receptors,
  and cofactors from scRNA-seq data before inferring cell-cell communication probabilities
  with CellChat or a reimplementation of its mass-action model.
when_to_use_negative:
- Input is spatial transcriptomics data needing single-cell-resolution communication
  (use CellChat v3/SpatialCellChat instead)
- Only differential expression of signaling genes is needed rather than group-level
  averaged expression feeding interaction inference
- Cell grouping or clustering is not yet established — CellChat and related tools
  require pre-defined cell groups as a prerequisite
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: R package implementing the trimean averaging ('trim' argument), geometric-mean
    complex aggregation, and downstream computeCommunProb mass-action model; used
    as the reference implementation to validate reimplementations
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated database (2,021 validated interactions) supplying ligand-receptor
    pairs, cofactors, and multi-subunit complex composition used to select genes for
    aggregation
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:330236bb6f2e0347234d980e784bdbf32da36ab7f1d73257c24d9940431719b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - computeCommunProb
  - computeCommunProbPathway
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scrnaseq-trimean-expression-aggregation

## Summary

Computes the statistically robust ensemble-average expression of signaling genes per cell group using the trimean (mean of quartiles), with geometric-mean aggregation for multi-subunit ligand/receptor complexes from CellChatDB, as the input layer for CellChat's mass-action communication probability model.

## When to use

Use when preparing per-cell-group average expression of ligands, receptors, and cofactors from scRNA-seq data before inferring cell-cell communication probabilities with CellChat or a reimplementation of its mass-action model.

## When NOT to use

- Input is spatial transcriptomics data needing single-cell-resolution communication (use CellChat v3/SpatialCellChat instead)
- Only differential expression of signaling genes is needed rather than group-level averaged expression feeding interaction inference
- Cell grouping or clustering is not yet established — CellChat and related tools require pre-defined cell groups as a prerequisite

## Inputs

- scRNA-seq expression matrix with cell group labels (e.g., 10X Genomics mouse skin data from GEO GSE113854)
- CellChatDB ligand-receptor-cofactor database annotations, including multi-subunit complex composition

## Outputs

- Per-cell-group trimean-averaged expression matrix for signaling genes (senders: ligands; receivers: receptors/cofactors)
- Geometric-mean-aggregated expression values for multi-subunit complexes
- Downstream communication probability table (source-target pair, pathway, probability, p-value), e.g., communication_probabilities.csv

## How to apply

After cell grouping and CellChatDB annotation, compute each signaling gene's average expression within every cell group using the trimean (mean of first and third quartiles) rather than the arithmetic mean, because the trimean is statistically robust to outlier cells in sparse scRNA-seq data. For heteromeric ligand or receptor complexes annotated in CellChatDB (e.g., multi-subunit receptors), aggregate subunit expressions with the geometric mean so a complex requires all subunits to be expressed. These averaged expressions feed the law-of-mass-action Hill function (default Kh=0.5, with agonist/antagonist cofactor modulation and optional population-size scaling ni*nj/n^2) inside computeCommunProb, and the resulting probabilities are significance-tested by permuting cell group labels M=100 times at p<0.05. Validate a reimplementation by numerically comparing its communication probability matrix against CellChat's computeCommunProb output on the same dataset (e.g., the 10X mouse skin data, GEO GSE113854) within a tight tolerance.

## Related tools

- **CellChat** (R package implementing the trimean averaging ('trim' argument), geometric-mean complex aggregation, and downstream computeCommunProb mass-action model; used as the reference implementation to validate reimplementations) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated database (2,021 validated interactions) supplying ligand-receptor pairs, cofactors, and multi-subunit complex composition used to select genes for aggregation) — https://github.com/sqjin/CellChat

## Examples

```
library(CellChat); cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat@data <- object@assays$RNA@data; cellchat <- computeCommunProb(cellchat, type = "triMean", population.size = TRUE)
```

## Evaluation signals

- Reimplemented communication probabilities match CellChat computeCommunProb outputs within a tight numerical tolerance across all source-target cell group pairs
- Permutation-filtered significant interactions (M=100 shuffles, p<0.05) agree with CellChat's significant-interaction set
- Geometric-mean aggregation of a multi-subunit complex yields zero when any subunit's averaged expression is zero
- Trimean values are stable to outlier cells compared with arithmetic means on the same group
- Output table contains schema columns source, target, pathway, probability, p-value for every tested cell group pair

## Limitations

- mRNA levels are used to approximate protein levels, but ligand-receptor interactions and mass action occur at the protein level; single-cell proteomics remains technically difficult
- Hill parameters Kh and n lack explicit biological connections; the Hill function is only a nonlinear approximation of ligand-receptor interactions
- The communication probability only represents interaction strength and is not exactly a probability

## Evidence

- [discussion] we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method: "we calculated the ensemble average expression of signaling genes in a given cell group using a statistically robust mean method"
- [discussion] mass action modeling quote: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] permutation test quote: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [intro] complex composition quote: [evidence span withheld — anchor status: not_found]
- [methods] github install quote: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
