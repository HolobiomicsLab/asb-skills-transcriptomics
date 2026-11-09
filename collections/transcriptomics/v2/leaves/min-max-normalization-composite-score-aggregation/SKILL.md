---
name: min-max-normalization-composite-score-aggregation
description: Use when comparing multiple integration methods with metrics on different
  scales (NMI, ARI, ASW, PCR batch, kBET, LISI) and you need normalized composite
  bio-conservation, batch-correction, and combined scores.
when_to_use_negative:
- Only one method is being evaluated — min-max normalization across methods is undefined/degenerate
  without multiple competitors
- A metric cannot be computed for some methods (e.g., kBET for the healthy heart atlas
  dataset failed due to computational limitations), making cross-method normalization
  incomplete
tools:
- name: Scanpy
  role: Provides the AnnData-embedded integrated outputs from which Louvain clusters
    (resolution maximizing NMI) and metric inputs are derived before score aggregation
- name: scDREAMER
  role: One of the twelve methods whose metrics are min-max normalized and averaged
    into composite scores; achieved the highest combined composite score
  repo: https://github.com/Zafar-Lab/scDREAMER
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:8cee1c5b6c1ff8707d6d6e1861dcadabb7315e2bf1bac8504becd31554e8028c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# min-max-normalization-composite-score-aggregation

## Summary

Aggregates heterogeneous integration benchmark metrics into min-max-normalized composite scores across methods, enabling a single comparable ranking of twelve integration methods on the human pancreas task (16,382 cells, nine sequencing protocols).

## When to use

Use when comparing multiple integration methods with metrics on different scales (NMI, ARI, ASW, PCR batch, kBET, LISI) and you need normalized composite bio-conservation, batch-correction, and combined scores.

## When NOT to use

- Only one method is being evaluated — min-max normalization across methods is undefined/degenerate without multiple competitors
- A metric cannot be computed for some methods (e.g., kBET for the healthy heart atlas dataset failed due to computational limitations), making cross-method normalization incomplete

## Inputs

- Per-method metric table (12 methods x metrics: NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET, iLISI, cLISI, isolated label F1, isolated label ASW)
- Louvain cluster labels computed at the resolution maximizing NMI against ground-truth cell labels
- Ground-truth cell type and batch annotations from the pancreas AnnData object

## Outputs

- Composite bio-conservation score per method
- Composite batch-correction score per method
- Composite isolated-label score per method
- Combined composite score per method (e.g., written to pancreas_integration_metrics.csv)

## How to apply

First compute per-method bio-conservation metrics (NMI, ARI, cell-type ASW scaled as (ASW+1)/2), batch-correction metrics (batch ASW as 1-abs(ASW), PCR batch, graph connectivity, kBET with k=50, graph iLISI and graph cLISI via Dijkstra graph distances), and isolated-label metrics for the rarest-batch cell types. Then min-max normalize each metric across the twelve methods (scDREAMER, scVI, Scanorama, Harmony, Seurat, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML) so all values fall in [0,1] with higher being better. Average the normalized metrics into composite bio-conservation, composite batch-correction, and composite isolated-label scores, then average these into a combined composite score per method. The rationale is that raw metrics have incompatible ranges and directions, so normalization across methods is required before averaging into a fair combined ranking.

## Related tools

- **Scanpy** (Provides the AnnData-embedded integrated outputs from which Louvain clusters (resolution maximizing NMI) and metric inputs are derived before score aggregation)
- **scDREAMER** (One of the twelve methods whose metrics are min-max normalized and averaged into composite scores; achieved the highest combined composite score) — https://github.com/Zafar-Lab/scDREAMER

## Examples

```
norm = (df - df.min()) / (df.max() - df.min()); composite = norm.mean(axis=1)  # min-max normalize metrics across 12 methods, then average per composite group
```

## Evaluation signals

- Each metric is bounded as defined: NMI/ARI in [0,1], scaled cell-type ASW (ASW+1)/2 in [0,1], scaled kBET and batch ASW (1-abs(ASW)) in [0,1] with 1 indicating perfect mixing
- Composite scores must be computable for all twelve methods, confirming the results table is complete
- On the pancreas task, scDREAMER's combined composite score should be the highest, ~12.9% above the second-best method
- All normalized metric values fall within [0,1] after min-max scaling across methods
- Per-method results table (pancreas_integration_metrics.csv) contains one row per method with all four composite columns populated

## Limitations

- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large numbers of cells and batches, breaking complete normalization
- Composite scores depend on the method set being compared; adding or removing a method changes the min-max scaling and the combined ranking
- scDREAMER is a deep learning-based method that may require fine-tuning of some parameters for best performance, which can shift metric values between runs

## Evidence

- [methods] We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value.: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [other] batch ASW as 1-abs(ASW), PCR batch, graph connectivity, kBET with k=50, graph iLISI and graph cLISI via Dijkstra graph distances: [evidence span withheld — anchor status: not_found]
- [other] Min-max normalize each metric across the twelve methods and average into composite bio-conservation, composite batch-correction, composite isolated-label, and combined composite scores.: [evidence span withheld — anchor status: not_found]
- [methods] for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [abstract] scDREAMER and scDREAMER-Sup outperform state-of-the-art unsupervised and supervised integration methods respectively in batch-correction and conservation of biological variation: "scDREAMER and scDREAMER-Sup outperform state-of-the-art unsupervised and supervised integration methods respectively in batch-correction and conservation of biological variation"
