---
name: adjusted-rand-index-normalized-mutual-information-evaluation
description: Use when a set of scRNA-seq clustering methods (e.g., SC3, CIDR, monocle,
  pcaReduce, Rphenograph, Seurat, SHARP, SINCERA, RaceID, scEFSC) has been run on
  datasets with published cell-type labels and you need to quantify and compare clustering
  accuracy.
when_to_use_negative:
- The dataset has no published cell-type labels — ARI/NMI here are external-validation
  metrics against ground truth and cannot be computed without reference labels
- You only have one method's clustering and no comparison benchmark — a single ARI/NMI
  value cannot establish outperformance claims without per-dataset, multi-method tabulation
- The task is differential expression, GO enrichment, or KEGG analysis of a chosen
  clustering — those downstream steps need cluster identities, not label-agreement
  scores
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: scEFSC
  role: Ensemble feature-selection-based consensus clustering method whose labels
    are scored alongside nine baselines in the ARI/NMI benchmark
  repo: https://github.com/Conan-Bian/scEFSC
- name: SHARP
  role: Baseline clustering method; its ARI utility function is used to compare predicted
    labels with reference labels
  repo: https://github.com/shibiaowan/SHARP
- name: SC3
  role: Baseline clustering method run with the number of cell types as k before ARI/NMI
    scoring
  repo: https://github.com/hemberg-lab/SC3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:8e432c603ce211f40f24b498dbce684dfa1adb822b8773158895cda21593023d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - CIDR
  - Monocle
  - RaceID
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scEFSC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# adjusted-rand-index-normalized-mutual-information-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark clustering accuracy of single-cell RNA-seq clustering methods by computing Adjusted Rand Index (ARI) and Normalised Mutual Information (NMI) between each method's predicted cluster labels and published cell-type labels, then tabulating per-dataset, per-method scores to rank methods (e.g., scEFSC vs nine baselines on fourteen datasets).

## When to use

Use when a set of scRNA-seq clustering methods (e.g., SC3, CIDR, monocle, pcaReduce, Rphenograph, Seurat, SHARP, SINCERA, RaceID, scEFSC) has been run on datasets with published cell-type labels and you need to quantify and compare clustering accuracy.

## When NOT to use

- The dataset has no published cell-type labels — ARI/NMI here are external-validation metrics against ground truth and cannot be computed without reference labels
- You only have one method's clustering and no comparison benchmark — a single ARI/NMI value cannot establish outperformance claims without per-dataset, multi-method tabulation
- The task is differential expression, GO enrichment, or KEGG analysis of a chosen clustering — those downstream steps need cluster identities, not label-agreement scores

## Inputs

- Cluster label vector(s) predicted by each clustering method for every cell
- Published cell-type ground-truth labels for every cell
- Pre-processed scRNA-seq expression matrices (log2-normalized, low-expression genes filtered) for fourteen GEO/ArrayExpress/dbGaP datasets (e.g., GSE36552, GSE83139, GSE81861, GSE59739, GSE81252, GSE81608, GSE71585, GSE85241, E-MTAB-5061, GSE65525)

## Outputs

- Per-dataset, per-method ARI and NMI table (benchmark_ari_nmi_table.csv) covering ten methods x fourteen datasets
- Method ranking per dataset/metric identifying which method achieved best NMI/ARI

## How to apply

Run each clustering method on the log2-normalized, gene-filtered data (genes detected in <2% of cells removed; 5000 genes pre-selected by non-negative kernel autoencoder for scEFSC), supplying the true number of cell types as k where required (e.g., SC3). For each method and dataset, compute ARI and NMI between the method's predicted cluster labels and the published cell-type labels using an ARI/NMI utility such as the one shipped with SHARP. Tabulate per-dataset, per-method ARI and NMI values (e.g., benchmark_ari_nmi_table.csv), ensuring ten methods (scEFSC plus nine baselines) by fourteen datasets are covered. Rank methods per metric per dataset; validation here was that scEFSC achieved the best NMI on eleven of the fourteen datasets. Cells labeled ambiguously as 'abandoned' should be removed before scoring.

## Related tools

- **scEFSC** (Ensemble feature-selection-based consensus clustering method whose labels are scored alongside nine baselines in the ARI/NMI benchmark) — https://github.com/Conan-Bian/scEFSC
- **SHARP** (Baseline clustering method; its ARI utility function is used to compare predicted labels with reference labels) — https://github.com/shibiaowan/SHARP
- **SC3** (Baseline clustering method run with the number of cell types as k before ARI/NMI scoring) — https://github.com/hemberg-lab/SC3

## Examples

```
ARI(label, res)  # SHARP utility: compare predicted cluster labels `res` with published cell-type labels `label`
```

## Evaluation signals

- Output table contains ARI and NMI entries for ten methods (scEFSC plus nine baselines) across all fourteen datasets
- Reproduced result: scEFSC achieves the best NMI on eleven of the fourteen datasets
- NMI values fall in [0, 1]; ARI values fall below or equal 1 with 0 as the random baseline — negative ARI indicates worse-than-random label agreement
- Scores are computed strictly against the published cell-type labels, with cells ambiguously labeled 'abandoned' removed beforehand
- Per-dataset ARI/NMI values match the published benchmark table when the same preprocessing (log2 transform, <2%-of-cells gene filter, 5000-gene autoencoder pre-selection) is applied

## Limitations

- ARI/NMI require published ground-truth cell-type labels, which may themselves be imperfect or ambiguous (cells labeled 'abandoned' had to be removed)
- Some baselines require the true number of cell types as k (e.g., SC3), so the evaluation is not fully unsupervised
- Performance is dataset-dependent — no single algorithm is optimal on every dataset, which is why per-dataset tabulation is needed

## Evidence

- [other] Compute Adjusted Rand Index (ARI) and Normalised Mutual Information (NMI) between each method's cluster labels and the published cell-type labels for every dataset.: [evidence span withheld — anchor status: not_found]
- [results] scEFSC best NMI on 11/14 datasets: "of the ten single-cell clustering algorithms, scEFSC provided the best clustering results on eleven of the fourteen datasets"
- [discussion] ARI/NMI outperformance claim: "significantly outperformed other single-cell clustering algorithms in terms of ARI and NMI evaluation metrics"
- [readme] SHARP ARI utility: [evidence span withheld — anchor status: external_doc]
- [results] abandoned cells removed: "we removed cells labeled ambiguously as ”abandoned”"
