---
name: geometric-sketching-subsampling
description: Use when benchmarking stability of communication-inference or clustering
  results against reduced cell counts, e.g. re-running CellChat on geometric sketches
  at 90/80/70% and scoring TPR/FPR/ACC vs full-data predictions.
when_to_use_negative:
- The dataset lacks cell-cluster labels — cell clustering is a pre-requisite for communication
  analysis with CellChat, CellPhoneDB, iTALK and SingleCellSignalR, so sketches of
  unclustered data cannot be scored
- You have ground-truth validated interactions — direct evaluation against truth is
  preferable to self-referential sketch-vs-full-data comparison
- The tool's predictions depend on absolute cell counts per group rather than relative
  abundances, making sketch-level comparisons misleading
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellChat
  role: Communication inference re-run on each geometric sketch; its interactions
    at each subsampling level are scored as TPR/FPR/ACC against full-data predictions
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB v2.0
  role: Comparator tool run on the same clustered and subsampled datasets to contrast
    robustness (TPR/FPR/ACC) under subsampling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:ca6160eba5062ee3b272bad04e667638991b18aee6a5290d59ccb8cf9b20dbeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellPhoneDB
  - CellPhoneDB v2.0
  - SingleCellSignalR
  - iTALK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Geometric sketching subsampling for robustness benchmarking

## Summary

Subsample scRNA-seq datasets with geometric sketching at 90%, 80%, and 70% of cells, re-run cell-cell communication inference on each sketch, and score retained interactions against full-data predictions using TPR, FPR, and ACC. Used in the CellChat paper to show that CellChat and CellPhoneDB are relatively robust to subsampling.

## When to use

Use when benchmarking stability of communication-inference or clustering results against reduced cell counts, e.g. re-running CellChat on geometric sketches at 90/80/70% and scoring TPR/FPR/ACC vs full-data predictions.

## When NOT to use

- The dataset lacks cell-cluster labels — cell clustering is a pre-requisite for communication analysis with CellChat, CellPhoneDB, iTALK and SingleCellSignalR, so sketches of unclustered data cannot be scored
- You have ground-truth validated interactions — direct evaluation against truth is preferable to self-referential sketch-vs-full-data comparison
- The tool's predictions depend on absolute cell counts per group rather than relative abundances, making sketch-level comparisons misleading

## Inputs

- Preprocessed, QC-filtered and cell-clustered scRNA-seq datasets (mouse skin wound datasets GEO GSM3453535, GSM3453536, GSM3453537, GSM3453538)
- Full-data cell-cell communication predictions (e.g. CellChat with CellChatDB, CellPhoneDB v2.0) used as the reference for TPR/FPR/ACC
- Subsampling fractions: 90%, 80%, 70% of cells via geometric sketching

## Outputs

- Per-level TPR, FPR, and ACC values for each tool at 90/80/70% subsampling relative to full-data predictions
- Benchmark table (benchmark_metrics.csv) with columns for tool, dataset, metric, and value

## How to apply

Take each preprocessed, cell-clustered dataset (here the four mouse skin wound datasets GSM3453535-GSM3453538; clustering is a pre-requisite for CellChat and other tools) and generate geometric sketches retaining 90%, 80%, and 70% of cells. Re-run CellChat (with CellChatDB) on each sketch and compute true-positive rate, false-positive rate, and accuracy of interactions retained relative to the full-data predictions. Run CellPhoneDB v2.0 through the same sketch levels to obtain a comparator, since it predicts enriched signaling interactions by considering the minimum average expression. Assemble tool, dataset, metric, and value rows into a single benchmark table (benchmark_metrics.csv) and flag any metric deviating materially from the reported pattern: CellChat showing slightly higher TPR, lower FPR, and higher accuracy than CellPhoneDB with both methods relatively robust to subsampling.

## Related tools

- **CellChat** (Communication inference re-run on each geometric sketch; its interactions at each subsampling level are scored as TPR/FPR/ACC against full-data predictions) — https://github.com/sqjin/CellChat
- **CellPhoneDB v2.0** (Comparator tool run on the same clustered and subsampled datasets to contrast robustness (TPR/FPR/ACC) under subsampling)

## Examples

```
devtools::install_github("jinworks/CellChat"); # then re-run CellChat on each 90/80/70% geometric sketch and compute TPR/FPR/ACC vs full-data predictions
```

## Evaluation signals

- Reproduced values should match the reported pattern: under 90/80/70% subsampling CellChat shows slightly higher TPR, lower FPR, and higher accuracy than CellPhoneDB
- Both CellChat and CellPhoneDB should appear relatively robust to subsampling — TPR/FPR/ACC should degrade only modestly as sketch fraction drops from 90% to 70%
- benchmark_metrics.csv must contain rows for each tool × dataset × metric (TPR, FPR, ACC) × subsampling level (90/80/70)
- Any metric deviating materially from the reported robustness pattern should be flagged before accepting the reproduction
- Jaccard-style overlap between CellChat and CellPhoneDB predicted interactions on full data should land near the reported ~50% before sketching is attempted

## Limitations

- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth — sketch-based TPR/FPR/ACC measures agreement with full-data predictions, not biological correctness
- Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions, confounding cross-method robustness comparisons
- Overlap-based comparison assumes similar methods generate similar results regardless of accuracy

## Evidence

- [other] Subsampling TPR/FPR/ACC workflow: [evidence span withheld — anchor status: not_found]
- [other] Reported robustness finding: [evidence span withheld — anchor status: not_found]
- [discussion] Clustering prerequisite: "Cell clustering is a pre-requisite for cell–cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR."
- [discussion] Lack of ground truth: "systematic evaluation of predicted cell–cell communication networks is challenging due to the lack of ground truth"
- [methods] Install CellChat: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
