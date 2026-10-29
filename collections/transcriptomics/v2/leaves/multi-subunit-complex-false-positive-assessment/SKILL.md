---
name: multi-subunit-complex-false-positive-assessment
description: Use when benchmarking ligand-receptor inference tools on scRNA-seq data
  and you need to measure how often a tool predicts single-gene interactions for receptors
  that are known heteromeric complexes, or when choosing a database that accounts
  for multimeric receptor structure.
when_to_use_negative:
- The tool under test already supports multi-subunit receptor complexes (e.g., CellChat
  with CellChatDB), so the single-ligand/single-receptor restriction itself would
  not yield a meaningful false-positive measurement
- The scRNA-seq data are not yet cell-clustered — clustering is a pre-requisite for
  communication analysis with CellChat, CellPhoneDB, iTALK and SingleCellSignalR
- The task is validating biological ground truth of communications rather than comparing
  tool behavior; the article notes systematic evaluation is challenging due to the
  lack of ground truth
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellChat
  role: Runs on the same clustered datasets as the comparison tools and provides CellChatDB,
    whose multimeric-complex annotations are the reference for flagging single-subunit
    mispredictions
  repo: https://github.com/sqjin/CellChat
- name: SingleCellSignalR
  role: Comparison tool whose one-ligand/one-receptor predictions are assessed for
    a 10.6% multi-subunit false-positive rate
- name: iTALK
  role: Comparison tool whose one-ligand/one-receptor predictions are assessed for
    a 14.3% multi-subunit false-positive rate
- name: CellPhoneDB v2.0
  role: Comparison tool run on the same clustered datasets; its enriched signaling
    predictions are overlapped with CellChat results (~50% overlap) in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:cc7bd304639843a89d40c8cf15bef92180a573014925dad85aee5456af341575
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# multi-subunit-complex-false-positive-assessment

## Summary

Quantifies the false-positive rate incurred by cell-cell communication tools that model only one-ligand/one-receptor gene pairs, by flagging predicted interactions whose receptors are actually multi-subunit complexes. CellChat reports 10.6% (SingleCellSignalR) and 14.3% (iTALK) of such mispredictions on mouse skin scRNA-seq data.

## When to use

Use when benchmarking ligand-receptor inference tools on scRNA-seq data and you need to measure how often a tool predicts single-gene interactions for receptors that are known heteromeric complexes, or when choosing a database that accounts for multimeric receptor structure.

## When NOT to use

- The tool under test already supports multi-subunit receptor complexes (e.g., CellChat with CellChatDB), so the single-ligand/single-receptor restriction itself would not yield a meaningful false-positive measurement
- The scRNA-seq data are not yet cell-clustered — clustering is a pre-requisite for communication analysis with CellChat, CellPhoneDB, iTALK and SingleCellSignalR
- The task is validating biological ground truth of communications rather than comparing tool behavior; the article notes systematic evaluation is challenging due to the lack of ground truth

## Inputs

- Mouse skin scRNA-seq datasets from GEO (GSM3453535, GSM3453536, GSM3453537, GSM3453538) with cell-cluster labels
- Predicted ligand-receptor interaction lists from SingleCellSignalR and iTALK
- CellChatDB ligand-receptor/cofactor interaction database with multimeric (heteromeric complex) annotations

## Outputs

- Per-tool multi-subunit false-positive rates (e.g., 10.6% SingleCellSignalR, 14.3% iTALK)
- List of interaction pairs wrongly predicted as single-subunit
- benchmark_metrics.csv table with columns tool, dataset, metric, value

## How to apply

On each preprocessed and cell-clustered scRNA-seq dataset (e.g., the four mouse skin wound datasets GSM3453535-GSM3453538 from GEO), run the comparison tools — SingleCellSignalR, iTALK, and CellChat with CellChatDB — ensuring cell clustering is done first since it is a pre-requisite for all these tools. Restrict the SingleCellSignalR and iTALK predicted interactions to one-ligand/one-receptor gene pairs, then cross-reference each predicted receptor against the multimeric-composition annotations in CellChatDB (the first manually curated mouse signaling database that considers multimeric structure, containing 2,021 validated molecular interactions) to identify cases where a single-gene receptor is actually a multi-subunit complex. The fraction of such wrongly predicted single-subunit interactions is the tool's false-positive rate attributable to ignoring complex composition (reported: 10.6% for SingleCellSignalR and 14.3% for iTALK). Record these rates, together with CellChat-CellPhoneDB prediction overlap (~50% Jaccard-style overlap) and subsampling TPR/FPR/ACC at 90/80/70% geometric sketching, in a benchmark table (benchmark_metrics.csv) with columns tool, dataset, metric, value, and flag any metric deviating materially from the reported values.

## Related tools

- **CellChat** (Runs on the same clustered datasets as the comparison tools and provides CellChatDB, whose multimeric-complex annotations are the reference for flagging single-subunit mispredictions) — https://github.com/sqjin/CellChat
- **SingleCellSignalR** (Comparison tool whose one-ligand/one-receptor predictions are assessed for a 10.6% multi-subunit false-positive rate)
- **iTALK** (Comparison tool whose one-ligand/one-receptor predictions are assessed for a 14.3% multi-subunit false-positive rate)
- **CellPhoneDB v2.0** (Comparison tool run on the same clustered datasets; its enriched signaling predictions are overlapped with CellChat results (~50% overlap) in the benchmark)

## Examples

```
devtools::install_github("jinworks/CellChat")
```

## Evaluation signals

- Reproduced multi-subunit false-positive rates match reported 10.6% (SingleCellSignalR) and 14.3% (iTALK) within a tolerance band; deviations are flagged
- Every single-gene receptor prediction is cross-referenced against CellChatDB multimeric annotations before counting — none counted without a database lookup
- benchmark_metrics.csv has the required schema (tool, dataset, metric, value) and covers all four datasets (GSM3453535-GSM3453538)
- CellChat-CellPhoneDB prediction overlap reproduces the reported ~50% figure
- Subsampling benchmark covers all three geometric-sketch levels (90/80/70%) with TPR, FPR and ACC computed for each

## Limitations

- Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions, confounding cross-method comparison
- Overlap-based comparison assumes similar methods generate similar results regardless of accuracy
- Systematic evaluation of predicted cell-cell communication networks is challenging due to the lack of ground truth
- mRNA levels are used to approximate protein levels, and single-cell proteomic information is technically difficult to capture

## Evidence

- [intro] Single-gene mispredictions arise from ignoring complex structure: "these and other similar methods usually use only one ligand/one receptor gene pairs, often neglecting that many receptors function as multi-subunit complexes"
- [discussion] Cell clustering pre-requisite for all benchmarked tools: "Cell clustering is a pre-requisite for cell–cell communication analysis with CellChat and other tools, such as CellPhoneDB, iTALK and SingleCellSignalR."
- [discussion] CellChatDB uniquely encodes multimeric structure, enabling the reference annotation: "CellChatDB is the first manually curated signaling interaction database in mouse that considers multimeric structure."
- [discussion] Lack of ground truth limits validation framing: "systematic evaluation of predicted cell–cell communication networks is challenging due to the lack of ground truth7."
- [readme] Tool installation for reproducing the assessment: "CellChat v1/v2 R package can be easily installed from Github using devtools: devtools::install_github("jinworks/CellChat")"
