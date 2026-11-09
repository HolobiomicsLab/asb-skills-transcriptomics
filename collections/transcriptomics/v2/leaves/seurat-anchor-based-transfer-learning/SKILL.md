---
name: seurat-anchor-based-transfer-learning
description: Use when benchmarking protein imputation from paired RNA/ADT CITE-seq
  data against a non-neural transfer-learning baseline, or when comparing predictions
  in holdout (10%/90%) and out-of-cell-type splits.
when_to_use_negative:
- You need accurate imputation for cell types absent from but related to the training
  data — the paper shows Seurat v3 is vastly outperformed by cTP-net in the out-of-cell-type
  scenario
- You want a denoising-then-prediction pipeline with SAVER-X preprocessing — that
  is part of the cTP-net workflow, not the Seurat anchor transfer baseline
- Input expression is TPM/RPKM or non-UMI data — the paper's imputation framework
  applies only to count-based (UMI) input
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Seurat v3
  role: Anchor-based transfer-learning baseline used to impute surface protein abundances
    from transcriptomes for comparison against cTP-net
  repo: https://github.com/zhouzilu/cTPnet
- name: cTP-net
  role: Deep neural network method whose imputation results are compared side-by-side
    with Seurat v3 in the benchmark heatmap
  repo: https://github.com/zhouzilu/cTPnet
- name: R
  role: Language in which the Seurat v3 anchor-transfer workflow and comparison plots
    are executed
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:5f841156abec937ab6c9e82e14f34457cf575c85ede11dcb01dad5dc892de676
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - Seurat v3
  - cTP-net
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Seurat v3 anchor-based transfer learning

## Summary

Benchmark baseline method that imputes surface protein (ADT) abundances from scRNA-seq transcriptomes using Seurat v3's anchor-based transfer-learning workflow, used in the cTP-net paper as the comparator in holdout and out-of-cell-type validation scenarios on CITE-PBMC data.

## When to use

Use when benchmarking protein imputation from paired RNA/ADT CITE-seq data against a non-neural transfer-learning baseline, or when comparing predictions in holdout (10%/90%) and out-of-cell-type splits.

## When NOT to use

- You need accurate imputation for cell types absent from but related to the training data — the paper shows Seurat v3 is vastly outperformed by cTP-net in the out-of-cell-type scenario
- You want a denoising-then-prediction pipeline with SAVER-X preprocessing — that is part of the cTP-net workflow, not the Seurat anchor transfer baseline
- Input expression is TPM/RPKM or non-UMI data — the paper's imputation framework applies only to count-based (UMI) input

## Inputs

- CITE-PBMC paired RNA/ADT count matrix pre-split into random holdout scheme (10% testing / 90% training)
- CITE-PBMC paired RNA/ADT counts partitioned by cell type for out-of-cell-type benchmarking
- Cell-type annotations for PBMC cells
- CITE-seq-measured ADT protein abundance matrix (ground truth)

## Outputs

- Seurat-imputed surface protein abundance matrix per split
- Per-protein, per-cell-type Pearson correlation table (Seurat vs. measured)
- Comparison heatmap of imputation correlations for cTP-net vs. Seurat v3 (Fig. 2a / Supplementary Fig. 5 equivalent)
- Within-cell-type scatter plots of measured vs. imputed protein abundance (Fig. 2b-c equivalent)

## How to apply

Load the CITE-PBMC paired RNA/ADT counts pre-split either into the random holdout scheme (10% testing / 90% training) or the out-of-cell-type scheme (partition cells by cell type and iteratively hold out all cells of one type). Run the Seurat v3 anchor-based transfer-learning workflow on each split to impute surface protein abundances from transcriptomes, using the same training cells as cTP-net for a fair comparison. Compute per-protein, per-cell-type Pearson correlations between Seurat-imputed and CITE-seq-measured protein levels, restricting comparisons to the same cells that were in the cTP-net validation set to account for variations between subsets. Render a comparison heatmap of imputation correlations for cTP-net vs. Seurat v3 across protein targets and splits (Fig. 2a, Supplementary Fig. 5) plus within-cell-type scatter plots of measured vs. imputed abundance (Fig. 2b-c). The key decision point is the validation scenario: expect Seurat v3 to perform comparably to cTP-net on random holdout but to degrade sharply when the held-out cell type is absent from training.

## Related tools

- **Seurat v3** (Anchor-based transfer-learning baseline used to impute surface protein abundances from transcriptomes for comparison against cTP-net) — https://github.com/zhouzilu/cTPnet
- **cTP-net** (Deep neural network method whose imputation results are compared side-by-side with Seurat v3 in the benchmark heatmap) — https://github.com/zhouzilu/cTPnet
- **R** (Language in which the Seurat v3 anchor-transfer workflow and comparison plots are executed) — https://github.com/zhouzilu/cTPnet

## Examples

```
library(Seurat); anchors <- FindTransferAnchors(reference = pbmc_ref, query = pbmc_query, dims = 1:30); imputed <- TransferData(anchorset = anchors, refdata = pbmc_ref$ADT)
```

## Evaluation signals

- In the random holdout scenario, Seurat v3 per-protein Pearson correlations should be comparable to cTP-net (cTP-net slightly better)
- In the out-of-cell-type scenario, cTP-net correlation entries in the comparison heatmap should substantially exceed the Seurat v3 entries, matching the paper's conclusion
- Correlation comparisons must be restricted to the same cells used in the cTP-net validation set to control for subset variation
- Per-protein, per-cell-type Pearson correlations of Seurat-imputed vs. CITE-seq-measured levels should be finite and reported for every protein target and split
- Seurat-imputed values should be generated on identical training/testing cell assignments as the cTP-net runs

## Limitations

- Anchor-based transfer degrades substantially when predicting cell types absent from the training data (the paper's out-of-cell-type scenario)
- Benchmark comparisons are limited to the same cells as the validation set due to variation between subsets
- cTP-net (and by extension the paired benchmark) can only apply to count-based expression input (UMI-based); accuracy on TPM/RPKM CITE-seq data is unknown

## Evidence

- [discussion] Benchmarking Seurat v3 anchor transfer as baseline: "We compared cTP-net with an anchor-based transfer-learning method developed in Seurat v3"
- [discussion] Same-cell comparison restriction: "we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets."
- [discussion] Out-of-cell-type split design: "instead of selecting the test set cells randomly, we partition the cells by their cell types"
- [discussion] Generalization claim for cTP-net: "the predictions generalize to cell types that are absent from, but related to those in the training data"
- [discussion] UMI-only applicability: "cTP-net can only apply to count-based expression input (UMI-based)."
