---
name: cross-method-prediction-overlap-analysis
description: Use when benchmarking ligand-receptor inference tools on the same scRNA-seq
  dataset, e.g. comparing CellChat vs CellPhoneDB/SingleCellSignalR/iTALK predictions,
  overlap, or robustness to cell subsampling.
when_to_use_negative:
- When a true ground-truth set of validated cell-cell interactions exists — direct
  accuracy evaluation is preferable to overlap-based comparison
- When the goal is to evaluate CellChat's own scalability or accuracy, not to rank
  competing communication-inference tools on identical inputs
- When tools are run on differently curated ligand-receptor databases without restriction,
  since database differences confound cross-method comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellChat
  role: Primary communication-inference tool benchmarked against alternatives; run
    on full and geometric-sketch subsampled datasets using CellChatDB
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB v2.0
  role: Comparator tool; predicts enriched signaling interactions by considering the
    minimum average expression, used for overlap computation
- name: SingleCellSignalR
  role: Comparator tool; restricted to one-ligand/one-receptor pairs to measure multi-subunit
    false positives
- name: iTALK
  role: Comparator tool; restricted to one-ligand/one-receptor pairs to measure multi-subunit
    false positives
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:800611736dc6872fef0157c4efd25a20ffc7484faafd3d1d4f020ceeaa50add5
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-method-prediction-overlap-analysis

## Summary

A benchmarking skill that compares cell-cell communication predictions from multiple tools (CellChat, CellPhoneDB v2.0, SingleCellSignalR, iTALK) applied to the same clustered scRNA-seq data, quantifying overlap, false-positive rates on multi-subunit receptors, and TPR/FPR/ACC under cell subsampling.

## When to use

Use when benchmarking ligand-receptor inference tools on the same scRNA-seq dataset, e.g. comparing CellChat vs CellPhoneDB/SingleCellSignalR/iTALK predictions, overlap, or robustness to cell subsampling.

## When NOT to use

- When a true ground-truth set of validated cell-cell interactions exists — direct accuracy evaluation is preferable to overlap-based comparison
- When the goal is to evaluate CellChat's own scalability or accuracy, not to rank competing communication-inference tools on identical inputs
- When tools are run on differently curated ligand-receptor databases without restriction, since database differences confound cross-method comparison

## Inputs

- Clustered mouse skin wound scRNA-seq datasets (GEO GSM3453535, GSM3453536, GSM3453537, GSM3453538)
- CellChatDB v1 interaction database
- CellChat interaction predictions per dataset
- CellPhoneDB v2.0 predicted enriched interactions per dataset
- SingleCellSignalR and iTALK ligand-receptor pair predictions per dataset
- Geometric-sketch subsampled versions of each dataset (90%, 80%, 70% of cells)

## Outputs

- Multi-subunit false-positive rates per tool (targets: 10.6% SingleCellSignalR, 14.3% iTALK)
- CellChat-CellPhoneDB interaction overlap fraction (target ~50%)
- TPR, FPR, ACC of CellChat at 90/80/70% subsampling levels
- benchmark_metrics.csv with columns tool, dataset, metric, value

## How to apply

Start from identically preprocessed and clustered datasets, since cell clustering is a prerequisite for all four tools. Run CellChat with CellChatDB and run SingleCellSignalR and iTALK on each dataset, restricting their predicted interactions to one-ligand/one-receptor gene pairs so the fraction of interactions wrongly predicted as single-subunit for true multi-subunit complexes can be quantified (reported ~10.6% for SingleCellSignalR, ~14.3% for iTALK). Run CellPhoneDB v2.0, which predicts enriched interactions using the minimum average expression, and compute a Jaccard-style overlap of its predictions with CellChat's (~50% reported). For robustness, subsample cells with geometric sketching at 90%, 80%, and 70%, re-run CellChat on each sketch, and compute TPR, FPR, and ACC of retained interactions relative to full-data predictions. Assemble metrics into benchmark_metrics.csv with columns tool, dataset, metric, value, and flag metrics deviating materially from reported values. Rationale: lack of ground truth forces comparison-based evaluation, and different ligand-receptor databases across methods confound cross-method performance comparison.

## Related tools

- **CellChat** (Primary communication-inference tool benchmarked against alternatives; run on full and geometric-sketch subsampled datasets using CellChatDB) — https://github.com/sqjin/CellChat
- **CellPhoneDB v2.0** (Comparator tool; predicts enriched signaling interactions by considering the minimum average expression, used for overlap computation)
- **SingleCellSignalR** (Comparator tool; restricted to one-ligand/one-receptor pairs to measure multi-subunit false positives)
- **iTALK** (Comparator tool; restricted to one-ligand/one-receptor pairs to measure multi-subunit false positives)

## Examples

```
devtools::install_github("jinworks/CellChat"); cellchat <- createCellChat(object, meta, group.by = "labels")
```

## Evaluation signals

- Multi-subunit false-positive rate for SingleCellSignalR ≈ 10.6% and for iTALK ≈ 14.3% after restricting to one-ligand/one-receptor pairs
- CellChat vs CellPhoneDB v2.0 predicted-interaction overlap ≈ 50% (Jaccard-style) on the same clustered datasets
- TPR, FPR, and ACC computed for CellChat at all three sketching levels (90%, 80%, 70%) relative to full-data predictions
- CellChat TPR and ACC higher and FPR lower than CellPhoneDB's across subsampling levels, with both methods relatively robust to subsampling
- benchmark_metrics.csv contains rows for every tool × dataset × metric combination with no missing values

## Limitations

- No ground truth exists for systematic evaluation of predicted cell-cell communication networks, so comparisons are overlap-based and cannot directly establish accuracy
- Similar methods tend to generate similar results regardless of accuracy, limiting the informativeness of overlap metrics
- Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions, confounding cross-method performance comparison

## Evidence

- [discussion] prerequisite: "Cell clustering is a pre-requisite for cell–cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR."
- [discussion] db confound: "Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions."
- [discussion] overlap caveat: "Similar methods tend to generate similar results regardless of accuracy"
- [discussion] no ground truth: "systematic evaluation of predicted cell–cell communication networks is challenging due to the lack of ground truth7."
- [intro] cellphonedb method: "CellPhoneDB v2.0 has been developed, which predicts enriched signaling interactions between two cell populations by considering the minimum average"
- [intro] single-pair limitation: "these and other similar methods usually use only one ligand/one receptor gene pairs, often neglecting that many receptors function as multi-subunit complexes"
