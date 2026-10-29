---
name: multi-method-cell-cell-communication-benchmarking
description: Use when comparing performance of cell-cell communication inference tools
  (CellChat, CellPhoneDB, iTALK, SingleCellSignalR) on identical scRNA-seq datasets
  to quantify false-positive rates and robustness to cell subsampling.
when_to_use_negative:
- When a ground-truth set of validated cell-cell interactions is available — overlap-based
  comparison assumes similar methods generate similar results regardless of accuracy,
  so it is not a true accuracy measure
- When comparing tools that use different ligand-receptor databases if database choice
  alone is not the variable under study, since differing databases could contribute
  to the variety of predicted interactions and confound the comparison
- When the input lacks cell clustering labels, which are a pre-requisite for CellChat,
  CellPhoneDB, iTALK and SingleCellSignalR
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: Primary communication inference tool benchmarked; re-run on geometric sketches
    at 90/80/70% cells to compute TPR/FPR/ACC
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB v2.0
  role: Comparison method predicting enriched signaling interactions by considering
    the minimum average expression; overlap with CellChat computed
- name: SingleCellSignalR
  role: Comparison method restricted to one-ligand/one-receptor gene pairs to quantify
    multi-subunit false-positive rate
- name: iTALK
  role: Comparison method restricted to one-ligand/one-receptor gene pairs to quantify
    multi-subunit false-positive rate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:d1cffeef9b94ff49d5a5c121bd3230c213cb4faf57d7aa12867fdaee84f74f20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellPhoneDB
  - CellPhoneDB v2.0
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  - iTALK
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- cell-cell-communication-method-benchmarking
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# multi-method-cell-cell-communication-benchmarking

## Summary

Benchmark CellChat against SingleCellSignalR, iTALK and CellPhoneDB v2.0 on the same clustered scRNA-seq datasets (mouse skin wound, GSM3453535-GSM3453538), quantifying multi-subunit false-positive rates, CellChat-CellPhoneDB overlap, and TPR/FPR/ACC under 90/80/70% geometric sketch subsampling.

## When to use

Use when comparing performance of cell-cell communication inference tools (CellChat, CellPhoneDB, iTALK, SingleCellSignalR) on identical scRNA-seq datasets to quantify false-positive rates and robustness to cell subsampling.

## When NOT to use

- When a ground-truth set of validated cell-cell interactions is available — overlap-based comparison assumes similar methods generate similar results regardless of accuracy, so it is not a true accuracy measure
- When comparing tools that use different ligand-receptor databases if database choice alone is not the variable under study, since differing databases could contribute to the variety of predicted interactions and confound the comparison
- When the input lacks cell clustering labels, which are a pre-requisite for CellChat, CellPhoneDB, iTALK and SingleCellSignalR

## Inputs

- Four mouse skin wound scRNA-seq datasets from GEO: GSM3453535, GSM3453536, GSM3453537, GSM3453538
- Preprocessed datasets with QC and cell clustering labels
- CellChatDB ligand-receptor interaction database

## Outputs

- benchmark_metrics.csv with columns for tool, dataset, metric, and value
- Multi-subunit false-positive rates for SingleCellSignalR (10.6%) and iTALK (14.3%)
- CellChat-CellPhoneDB prediction overlap (~50%)
- TPR/FPR/ACC at 90/80/70% subsampling levels

## How to apply

Preprocess the four mouse skin wound datasets (GSM3453535-GSM3453538) with QC and cell clustering, since clustering is a pre-requisite for all four tools. Run CellChat with CellChatDB, and run SingleCellSignalR and iTALK restricting predictions to one-ligand/one-receptor gene pairs to measure the fraction of interactions wrongly predicted as single-subunit when the receptor is a multi-subunit complex (reported: 10.6% SingleCellSignalR, 14.3% iTALK). Run CellPhoneDB v2.0 — which predicts enriched interactions using the minimum average expression — on the same clustered data and compute the Jaccard-style overlap with CellChat predictions (~50% expected). Subsample each dataset with geometric sketching at 90%, 80% and 70% of cells, re-run CellChat on each sketch, and compute TPR, FPR and ACC relative to full-data predictions, then assemble everything into benchmark_metrics.csv (tool, dataset, metric, value).

## Related tools

- **CellChat** (Primary communication inference tool benchmarked; re-run on geometric sketches at 90/80/70% cells to compute TPR/FPR/ACC) — https://github.com/sqjin/CellChat
- **CellPhoneDB v2.0** (Comparison method predicting enriched signaling interactions by considering the minimum average expression; overlap with CellChat computed)
- **SingleCellSignalR** (Comparison method restricted to one-ligand/one-receptor gene pairs to quantify multi-subunit false-positive rate)
- **iTALK** (Comparison method restricted to one-ligand/one-receptor gene pairs to quantify multi-subunit false-positive rate)

## Examples

```
devtools::install_github("jinworks/CellChat"); cellchat <- createCellChat(object, meta = meta, group.by = "labels"); cellchat <- computeCommunProb(cellchat)
```

## Evaluation signals

- Multi-subunit false-positive rate reproduced near 10.6% for SingleCellSignalR and 14.3% for iTALK
- CellChat-CellPhoneDB predicted interaction overlap approximately 50%
- TPR, FPR and ACC computed at all three subsampling levels (90%, 80%, 70%) and populated for every dataset
- benchmark_metrics.csv schema check: columns for tool, dataset, metric, and value present
- CellChat shows slightly higher TPR, lower FPR and higher accuracy versus CellPhoneDB under subsampling, with both methods relatively robust to subsampling

## Limitations

- Systematic evaluation is hampered because no ground truth exists for predicted cell-cell communication networks
- Different ligand-receptor databases used by each method could contribute to the variety of predicted interactions, confounding cross-method comparison
- Overlap-based comparison is limited: similar methods tend to generate similar results regardless of accuracy

## Evidence

- [discussion] Cell clustering is a pre-requisite for cell-cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR.: "Cell clustering is a pre-requisite for cell–cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR."
- [intro] CellPhoneDB v2.0 predicts enriched signaling interactions between two cell populations by considering the minimum average: "CellPhoneDB v2.0 has been developed, which predicts enriched signaling interactions between two cell populations by considering the minimum average"
- [intro] these and other similar methods usually use only one ligand/one receptor gene pairs, often neglecting that many receptors function as multi-subunit complexes: "these and other similar methods usually use only one ligand/one receptor gene pairs, often neglecting that many receptors function as multi-subunit complexes"
- [discussion] systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth: "systematic evaluation of predicted cell–cell communication networks is challenging due to the lack of ground truth"
- [discussion] Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions.: "Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions."
