---
name: clustering-agreement-metrics-ari-nmi
description: Use when predicted cluster labels (e.g., from scEFSC, scDHA, SC3, Seurat)
  must be quantitatively scored against published cell-type labels of a scRNA-seq
  dataset to compare clustering methods by ARI and NMI.
when_to_use_negative:
- Ground-truth cell-type labels are unavailable for the dataset, since both ARI and
  NMI require a reference labeling
- Comparing raw expression matrices or feature subsets rather than discrete cluster
  label assignments
- A single clustering result must be assessed without any external reference (use
  internal validity indices instead)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scEFSC
  role: Produces the predicted cluster labels whose ARI/NMI are scored against published
    cell-type labels
  repo: https://github.com/Conan-Bian/scEFSC
- name: SHARP
  role: Provides an ARI implementation (ARI(label, res)) for checking clustering performance
    against a reference label
  repo: https://github.com/shibiaowan/SHARP
- name: SC3
  role: One of the nine baseline clustering algorithms whose labels are scored by
    ARI/NMI in the benchmark
  repo: https://github.com/hemberg-lab/SC3
- name: Seurat
  role: Baseline clustering method (PCA + SNN graph + Louvain) evaluated via ARI/NMI
    in the comparison
- name: CIDR
  role: Baseline clustering method (v0.1.5, dropout imputation via dimensionality
    reduction) scored by ARI/NMI
  repo: https://github.com/VCCRI/CIDR
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:458f7496c396dbc67dcbd56d74c70f2bc9e079b3176058b483a8f9e7dfe41168
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
  - PhenoGraph
  - RaceID
  - SAME-clustering
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scDHA
  - scEFSC
  - wMetaC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# clustering-agreement-metrics-ari-nmi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) to compare predicted single-cell cluster labels against previously published cell-type labels, the two metrics scEFSC used to benchmark clustering methods across fourteen scRNA-seq datasets.

## When to use

Use when predicted cluster labels (e.g., from scEFSC, scDHA, SC3, Seurat) must be quantitatively scored against published cell-type labels of a scRNA-seq dataset to compare clustering methods by ARI and NMI.

## When NOT to use

- Ground-truth cell-type labels are unavailable for the dataset, since both ARI and NMI require a reference labeling
- Comparing raw expression matrices or feature subsets rather than discrete cluster label assignments
- A single clustering result must be assessed without any external reference (use internal validity indices instead)

## Inputs

- Predicted cluster label vector for each cell (e.g., wMetaC meta-clustering output of scEFSC)
- Reference vector of previously published cell-type labels for the same cells
- Number of cell types k used as the clustering parameter

## Outputs

- ARI and NMI values per method-dataset pair
- Comparison table (e.g., ari_nmi_comparison.csv or ablation_ari_nmi_table.csv) with ARI and NMI columns across fourteen datasets
- Per-dataset ranking of methods by ARI and NMI

## How to apply

After obtaining final cluster labels (e.g., scEFSC's wMetaC weighted meta-clustering output with ward.D linkage and delta=0.01), compute both ARI and NMI against the previously published cell-type labels for each dataset-method pair. Set k to the number of known cell types before clustering so agreement metrics are meaningful. Assemble the per-dataset ARI/NMI values into a comparison table (e.g., ari_nmi_comparison.csv) covering all fourteen datasets, then rank methods per dataset and per metric. Ground the ARI computation in a reference implementation such as SHARP's ARI function, which returns Rand, HA, MA, FM and Jaccard statistics. Validate by checking the table reproduces expected patterns, e.g., scEFSC attaining the best NMI on eleven of fourteen datasets among ten single-cell clustering algorithms.

## Related tools

- **scEFSC** (Produces the predicted cluster labels whose ARI/NMI are scored against published cell-type labels) — https://github.com/Conan-Bian/scEFSC
- **SHARP** (Provides an ARI implementation (ARI(label, res)) for checking clustering performance against a reference label) — https://github.com/shibiaowan/SHARP
- **SC3** (One of the nine baseline clustering algorithms whose labels are scored by ARI/NMI in the benchmark) — https://github.com/hemberg-lab/SC3
- **Seurat** (Baseline clustering method (PCA + SNN graph + Louvain) evaluated via ARI/NMI in the comparison)
- **CIDR** (Baseline clustering method (v0.1.5, dropout imputation via dimensionality reduction) scored by ARI/NMI) — https://github.com/VCCRI/CIDR

## Examples

```
res = SHARP(scExp); ARI(label, res)
```

## Evaluation signals

- The results table contains ARI and NMI values for every method-dataset combination across all fourteen datasets
- Reproduction of the reported pattern: scEFSC provides the best NMI on eleven of the fourteen datasets among ten single-cell clustering algorithms
- Reproduction of reported ARI gains: scEFSC ARI >= 0.10 higher than scDHA on GSE81252, GSE81608, GSE85241, GSE60361 and phs000833.v3.p1
- In the ablation table, the full multi-feature-selection scEFSC achieves equal or better ARI/NMI than each single-feature-selection variant (Laplacian Score, Low Variance, SPEC, MCFS)
- ARI computation cross-checked against SHARP's ARI function output (Rand, HA, MA, FM, Jaccard) on a known example

## Limitations

- ARI and NMI are only meaningful relative to previously published labels; datasets lacking reliable cell-type annotations cannot be scored this way
- Both metrics require setting k equal to the number of known cell types, so performance is conditioned on knowing the true cluster count
- The article notes it is a challenge for users to decide which clustering algorithm is most appropriate, since each algorithm has strengths and weaknesses on particular datasets

## Evidence

- [abstract] a systematic and extensive performance evaluation of four feature selection methods and nine scRNA-seq clustering algorithms on fourteen real single-cell RNA-seq datasets: "a systematic and extensive performance evaluation of four feature selection methods and nine scRNA-seq clustering algorithms on fourteen real single-cell RNA-seq datasets"
- [readme] you can check the perfornance using the ARI (adjusted Rand index) by comparing your prediction results with the given reference clustering label of the example: [evidence span withheld — anchor status: external_doc]
