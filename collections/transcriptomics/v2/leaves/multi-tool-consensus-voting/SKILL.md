---
name: multi-tool-consensus-voting
description: Use when multiple independent scRNA-seq cell-type annotation tools have
  each produced per-cell label predictions on the same dataset and you need a consensus
  confidence mask separating confidently labeled cells from unconfident ones.
when_to_use_negative:
- You need higher annotation accuracy than simple majority voting provides — the paper
  shows its GCN label propagation method 'improves accuracy over a simple consensus
  algorithm'
- You have only one annotator tool's predictions, since a majority vote over a single
  labeler is degenerate
- Input is not a raw-count single-cell expression matrix (the README instructs 'Please
  input raw counts')
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: R4scSHARP
  role: R package whose main function run_r4scsharp runs the five cell classification
    tools on an RNA gene expression matrix and returns predictions in a succinct file
  repo: https://github.com/W-Holtz/R4scSHARP
- name: scSHARP
  role: Downstream pipeline that consumes the consensus confident-label mask and propagates
    labels with a semi-supervised GCN
  repo: https://github.com/lewinsohndp/scSHARP
- name: SingleR
  role: One of the five component annotators voted into the consensus
- name: SCINA
  role: One of the five component annotators voted into the consensus
- name: ScType
  role: One of the five component annotators voted into the consensus
- name: scSorter
  role: One of the five component annotators voted into the consensus
- name: scPred
  role: One of the five component annotators voted into the consensus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:fa984322d6ba312461ccbf38b7ff1ec6c065d04771682d28ef8a5930b5b66b09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/W-Holtz/R4scSHARP
  tools:
  - Python
  - R
  - R4scSHARP
  - SCINA
  - ScType
  - SingleR
  - conda
  - scPred
  - scSHARP
  - scSorter
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# multi-tool-consensus-voting

## Summary

Aggregate per-cell cell-type predictions from five independent single-cell annotation tools (SCINA, ScType, scSorter, SingleR, scPred) via simple majority voting, marking cells as unconfident when fewer than a 0.51 fraction of tools agree, to quantify how many cells can be confidently labeled before downstream label propagation.

## When to use

Use when multiple independent scRNA-seq cell-type annotation tools have each produced per-cell label predictions on the same dataset and you need a consensus confidence mask separating confidently labeled cells from unconfident ones.

## When NOT to use

- You need higher annotation accuracy than simple majority voting provides — the paper shows its GCN label propagation method 'improves accuracy over a simple consensus algorithm'
- You have only one annotator tool's predictions, since a majority vote over a single labeler is degenerate
- Input is not a raw-count single-cell expression matrix (the README instructs 'Please input raw counts')

## Inputs

- Gene expression matrix with raw counts (e.g., Splatter simulations at de.facScale 0.7 and 0.8, mouse testis Jung et al. 2019, human PBMC Zheng et al. 2017)
- Per-tool cell-type label prediction files from SCINA, ScType, scSorter, SingleR, and scPred

## Outputs

- Per-cell consensus label with a confidence flag (confident vs. unconfident at the 0.51 majority threshold)
- Per-dataset CSV summary table of unconfident cell counts and fractions (e.g., 156/15.6%, 85/8.5%, 46/6.6%, 2310/3.0%)

## How to apply

Run each of the five component tools independently on the same gene expression matrix (raw counts) using the R4scSHARP package, whose run_r4scsharp function returns all tool predictions in a single succinct file. Aggregate the five per-cell predictions and compute a simple majority-vote consensus: a cell is confidently labeled when at least a 0.51 fraction of tools agree on the same label, otherwise it is marked unconfident. Count unconfident cells per dataset and divide by total cells to obtain the unconfident fraction. These confident labels are the seeds that scSHARP's downstream GCN label propagation step spreads to the unconfident cells, so the consensus mask directly determines the semi-supervised training set.

## Related tools

- **R4scSHARP** (R package whose main function run_r4scsharp runs the five cell classification tools on an RNA gene expression matrix and returns predictions in a succinct file) — https://github.com/W-Holtz/R4scSHARP
- **scSHARP** (Downstream pipeline that consumes the consensus confident-label mask and propagates labels with a semi-supervised GCN) — https://github.com/lewinsohndp/scSHARP
- **SingleR** (One of the five component annotators voted into the consensus)
- **SCINA** (One of the five component annotators voted into the consensus)
- **ScType** (One of the five component annotators voted into the consensus)
- **scSorter** (One of the five component annotators voted into the consensus)
- **scPred** (One of the five component annotators voted into the consensus)

## Examples

```
library(R4scSHARP); preds <- run_r4scsharp(counts_matrix, labels=marker_genes)
```

## Evaluation signals

- Unconfident fractions on the four benchmarks should be comparable to reported values: 15.6% (de.facScale 0.7 simulation), 8.5% (de.facScale 0.8 simulation), 6.6% / 46 cells (testis), 3.0% / 2310 cells (PBMC)
- Every cell must carry exactly one consensus label or an unconfident flag — no missing cells between tool outputs
- A cell is confident only when the agreeing-tool fraction is >= 0.51 of the five tools
- The counts/fractions CSV should match the total cell counts of each input dataset (unconfident divided by total cells)

## Limitations

- Simple majority voting is only a baseline: the paper's GCN method 'improves accuracy over a simple consensus algorithm and the average of the underlying tools'
- Cells with no tool agreement become unconfident and get no label from this step alone — they must be handled by downstream label propagation
- Component tools require separate R dependency installs (SingleR via BiocManager, scPred via devtools from GitHub, etc.), which can complicate reproducibility

## Evidence

- [results] Applying the 0.51 majority-vote confidence threshold yields 15.6% unconfidently labeled cells in the de.facScale 0.7 simulation, 8.5% in the de.facScale 0.8 simulation, 6.6% (46 cells) in the testis dataset, and 3.0% (2310 cells) in the PBMC dataset.: [evidence span withheld — anchor status: not_found]
- [abstract] using state-of-the-art tools to find cells with highly confident label assignments through consensus: "using state-of-the-art tools to find cells with highly confident label assignments through consensus"
- [readme] the main function, run_r4scsharp, is used to run up to five cell classification tools on a RNA gene expression matrix, then return the tool predictions in a succinct file.: [evidence span withheld — anchor status: external_doc]
- [methods] Our package utilizes a suite of cutting edge single cell classification tools which we have compiled as an R package: "Our package utilizes a suite of cutting edge single cell classification tools which we have compiled as an R package"
