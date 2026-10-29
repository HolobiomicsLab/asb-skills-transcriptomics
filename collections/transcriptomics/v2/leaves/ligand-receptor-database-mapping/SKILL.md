---
name: ligand-receptor-database-mapping
description: Use when inferring cell-cell communication from scRNA-seq and interactions
  must be restricted to literature-supported ligand-receptor pairs, or when comparing
  tools whose one-ligand/one-receptor assumptions mis-handle multi-subunit receptor
  complexes.
when_to_use_negative:
- The goal is discovering novel, uncurated interactions — CellChatDB mapping only
  recovers literature-supported pairs, so de novo interaction discovery is out of
  scope
- The analysis requires protein-level validation — mRNA levels are only used to approximate
  protein levels, and single-cell proteomic information is not capturable at present
- Ground-truth evaluation of predicted networks is needed — no ground truth exists
  for systematic evaluation of predicted cell-cell communication networks
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: CellChat
  role: Maps inferred communications onto CellChatDB and aggregates ligand-receptor
    pairs into signaling pathways
  repo: https://github.com/sqjin/CellChat
- name: SingleCellSignalR
  role: Comparator whose one-ligand/one-receptor outputs are screened for multi-subunit
    false positives (10.6%)
- name: iTALK
  role: Comparator whose one-ligand/one-receptor outputs are screened for multi-subunit
    false positives (14.3%)
- name: CellPhoneDB v2.0
  role: Comparator predicting enriched interactions via minimum average expression,
    used for overlap analysis (~50%)
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:a25b069d40d6e154c2bd4428b9c86e39ed88674d42ec3c8778eb2f11220e918f
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
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# ligand-receptor-database-mapping

## Summary

Map predicted cell-cell signaling interactions to a curated ligand-receptor database (CellChatDB, built on KEGG) that accounts for multi-subunit receptor complexes and cofactors, enabling principled pathway-level classification and cross-tool benchmarking of communication inference.

## When to use

Use when inferring cell-cell communication from scRNA-seq and interactions must be restricted to literature-supported ligand-receptor pairs, or when comparing tools whose one-ligand/one-receptor assumptions mis-handle multi-subunit receptor complexes.

## When NOT to use

- The goal is discovering novel, uncurated interactions — CellChatDB mapping only recovers literature-supported pairs, so de novo interaction discovery is out of scope
- The analysis requires protein-level validation — mRNA levels are only used to approximate protein levels, and single-cell proteomic information is not capturable at present
- Ground-truth evaluation of predicted networks is needed — no ground truth exists for systematic evaluation of predicted cell-cell communication networks

## Inputs

- Preprocessed, cell-clustered scRNA-seq count matrix plus cell metadata (e.g., mouse skin wound data from GSM3453535-GSM3453538)
- CellChatDB resource (secreted signaling / ECM-receptor / cell-cell contact subsets), updateable via updateCellChatDB
- Predicted interaction lists from SingleCellSignalR, iTALK, or CellPhoneDB v2.0 restricted to one-ligand/one-receptor gene pairs

## Outputs

- Pathway-aggregated ligand-receptor interaction table with communication probabilities (computeCommunProbPathway output)
- Benchmark metrics (multi-subunit false-positive rates per tool; CellChat-CellPhoneDB overlap fraction) assemblable into benchmark_metrics.csv

## How to apply

Build inference on CellChatDB, a manually curated mouse database of 2,021 validated molecular interactions (60% paracrine/autocrine, 21% ECM-receptor, 19% cell-cell contact) largely curated from KEGG, which represents heteromeric complexes and cofactors rather than only one-ligand/one-receptor pairs. Assign interactions to functionally related signaling pathways via CellChatDB's pathway classification, then aggregate ligand-receptor probabilities with computeCommunProbPathway. Keep the database current with updateCellChatDB before analysis. When benchmarking against SingleCellSignalR or iTALK, restrict their predictions to one-ligand/one-receptor gene pairs to quantify the fraction of interactions wrongly predicted as single-subunit when the receptor is actually a multi-subunit complex (reported 10.6% for SingleCellSignalR and 14.3% for iTALK); note that differing ligand-receptor databases across methods can itself confound comparisons.

## Related tools

- **CellChat** (Maps inferred communications onto CellChatDB and aggregates ligand-receptor pairs into signaling pathways) — https://github.com/sqjin/CellChat
- **SingleCellSignalR** (Comparator whose one-ligand/one-receptor outputs are screened for multi-subunit false positives (10.6%))
- **iTALK** (Comparator whose one-ligand/one-receptor outputs are screened for multi-subunit false positives (14.3%))
- **CellPhoneDB v2.0** (Comparator predicting enriched interactions via minimum average expression, used for overlap analysis (~50%))

## Examples

```
library(CellChat); data(CellChatDB); cellchat <- createCellChat(data, meta = meta, group.by = "labels"); cellchat@DB <- CellChatDB
```

## Evaluation signals

- Multi-subunit false-positive rate for SingleCellSignalR restricted to one-ligand/one-receptor pairs reproduces ~10.6%
- Multi-subunit false-positive rate for iTALK reproduces ~14.3%
- Jaccard-style overlap between CellChat and CellPhoneDB v2.0 predicted interactions is approximately 50%
- All mapped interactions trace to entries in CellChatDB (2,021 validated molecular interactions) with assigned pathway labels
- Material deviation from any reported benchmark value triggers a flag in the assembled metrics table

## Limitations

- Different ligand-receptor databases used by each method confound cross-tool performance comparisons
- Overlap-based comparison assumes similar methods generate similar results regardless of accuracy
- CellChatDB at publication time was mouse-focused; human orthologue mapping requires tools such as biomaRt

## Evidence

- [intro] CellChatDB is a manually curated database accounting for ligand-receptor complex composition and cofactors, containing 2021 validated molecular interactions: [evidence span withheld — anchor status: not_found]
- [intro] Existing methods neglect multi-subunit complexes: "these and other similar methods usually use only one ligand/one receptor gene pairs, often neglecting that many receptors function as multi-subunit complexes"
- [methods] Database update function: "A function named `updateCellChatDB` is also provided for easily updating CellChatDB."
- [discussion] Cross-method comparison confounded by databases: "Different ligand-receptor databases used in each method could contribute to the variety of predicted interactions."
- [discussion] Overlap comparison caveat: "Similar methods tend to generate similar results regardless of accuracy"
