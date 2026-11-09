---
name: confidence-threshold-application
description: Use when you have per-cell label predictions from multiple annotation
  tools and need to separate high-confidence (majority-agreed) from unconfident cells
  before propagating labels.
when_to_use_negative:
- You need the final propagated annotations for all cells — the simple consensus threshold
  only splits confident vs. unconfident cells and does not label the unconfident ones
  (use scSHARP's GCN label propagation for that).
- You have predictions from fewer than three tools, making a 0.51 majority fraction
  degenerate or unavailable.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R4scSHARP
  role: Runs the five component classification tools (SCINA, ScType, scSorter, SingleR,
    scPred) on a gene expression matrix and returns their predictions for consensus
    aggregation
  repo: https://github.com/W-Holtz/R4scSHARP
- name: scSHARP
  role: Consumes the confident labels from the 0.51 majority-vote consensus and propagates
    them to unconfident cells via a semi-supervised GCN
  repo: https://github.com/lewinsohndp/scSHARP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:42e5733efff31042c84f1b40fc0c3a9fa052c6145773aad93144d6e12760638d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/W-Holtz/R4scSHARP
  tools:
  - Python
  - R
  - R4scSHARP
  - conda
  - scSHARP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# confidence-threshold-application

## Summary

Applies a 0.51 simple-majority vote over five scRNA-seq annotation tools (SCINA, ScType, scSorter, SingleR, scPred) to flag cells whose labels are not confidently agreed upon, quantifying the unconfident fraction per dataset. These confident/unconfident labels then feed scSHARP's GCN label propagation.

## When to use

Use when you have per-cell label predictions from multiple annotation tools and need to separate high-confidence (majority-agreed) from unconfident cells before propagating labels.

## When NOT to use

- You need the final propagated annotations for all cells — the simple consensus threshold only splits confident vs. unconfident cells and does not label the unconfident ones (use scSHARP's GCN label propagation for that).
- You have predictions from fewer than three tools, making a 0.51 majority fraction degenerate or unavailable.

## Inputs

- Per-tool cell-type label predictions for the same cells from five tools (SCINA, ScType, scSorter, SingleR, scPred)
- RNA gene expression matrix (raw counts) fed to the component tools
- Confidence threshold parameter (0.51 simple-majority fraction)

## Outputs

- Per-cell binary confident/unconfident label assignments
- Per-dataset counts of unconfidently labeled cells
- Per-dataset unconfident fraction (e.g., 15.6%, 8.5%, 6.6%, 3.0%), typically written to a CSV summary table

## How to apply

Run each of the five component tools (SCINA, ScType, scSorter, SingleR, scPred, provided via the R4scSHARP package) independently on the same gene expression matrix to produce per-tool cell-type labels. Aggregate the five predictions per cell and compute the fraction of tools agreeing on a label; mark a cell as unconfident when fewer than a 0.51 fraction of tools agree (i.e., simple majority of 3/5 required). Count unconfident cells per dataset and compute the unconfident fraction (unconfident cells / total cells), writing per-dataset counts to a CSV summary. In the paper this yielded 15.6% unconfident cells in the de.facScale 0.7 Splatter simulation, 8.5% at de.facScale 0.8, 6.6% (46 cells) in the mouse testis dataset (Jung et al. 2019), and 3.0% (2310 cells) in the human PBMC dataset (Zheng et al. 2017). The retained confident labels serve as the seed set for downstream GCN label propagation.

## Related tools

- **R4scSHARP** (Runs the five component classification tools (SCINA, ScType, scSorter, SingleR, scPred) on a gene expression matrix and returns their predictions for consensus aggregation) — https://github.com/W-Holtz/R4scSHARP
- **scSHARP** (Consumes the confident labels from the 0.51 majority-vote consensus and propagates them to unconfident cells via a semi-supervised GCN) — https://github.com/lewinsohndp/scSHARP

## Examples

```
library(devtools); install_github("W-Holtz/R4scSHARP"); library(R4scSHARP)  # then run_r4scsharp to obtain per-tool predictions for majority-vote consensus
```

## Evaluation signals

- Unconfident fraction per dataset should reproduce the reported values when the 0.51 threshold is applied correctly: 156 cells/15.6% (de.facScale 0.7 simulation), 85/8.5% (de.facScale 0.8), 46/6.6% (testis), 2310/3.0% (PBMC).
- A cell is unconfident only when strictly fewer than 0.51 of the five tools agree — i.e., cells with 3/5 or more agreeing tools must be labeled confident.
- Every cell must have exactly one label or one unconfident flag per dataset; unconfident count + confident count = total cells.
- Component predictions must come from the same cells and gene expression matrix across all five tools so the per-cell vote is valid.
- Summary CSV should contain per-dataset counts and fractions (unconfident cells / total cells) matching the source card values.

## Limitations

- The simple consensus improves label reliability only for cells the tools already agree on; the paper reports GCN label propagation improves accuracy over this simple consensus algorithm.
- Unconfident fractions vary strongly with dataset difficulty (3.0% for PBMC vs. 15.6% for the hardest simulation), so expected counts must be benchmarked per dataset.
- Component tools require R (via R4scSHARP, including SingleR, scSorter, scPred, SCINA dependencies), so the consensus step is not fully runnable from Python alone without R installed.

## Evidence

- [abstract] Confident label assignment via tool consensus: "using state-of-the-art tools to find cells with highly confident label assignments through consensus"
- [readme] R4scSHARP runs the five tools and returns predictions: [evidence span withheld — anchor status: external_doc]
- [readme] Component tool suite names: [evidence span withheld — anchor status: external_doc]
- [methods] Component predictions feed the GCN: "GCN model class propagates labels from component tool predictions"
- [results] Reported benchmark result context: [evidence span withheld — anchor status: not_found]
