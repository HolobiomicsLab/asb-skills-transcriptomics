---
name: benchmark-comparison-table-generation
description: Use when a new clustering method must be benchmarked against existing
  ensemble and single-cell baselines across multiple datasets and an ARI/NMI comparison
  table with per-dataset rankings is the deliverable.
when_to_use_negative:
- The evaluation goal is a single method on a single dataset with no baselines to
  compare — a ranking table has no comparative meaning
- The reference labels are unavailable or ambiguous (e.g., cells labeled ambiguously
  as 'abandoned' must first be removed, otherwise ARI/NMI against labels is undefined
  or biased)
- The task is biological interpretation (differential expression, GO/KEGG enrichment)
  rather than quantitative benchmarking
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scEFSC
  role: Proposed ensemble feature-selection clustering method whose ARI/NMI values
    fill the comparison table; code and pipeline from its GitHub repository
  repo: https://github.com/Conan-Bian/scEFSC
- name: SAME-clustering
  role: Ensemble clustering baseline run on the same preprocessed fourteen datasets
    to populate comparison columns
- name: SHARP
  role: Single-cell clustering baseline; SHARP README shows ARI computation against
    reference labels as the evaluation pattern
  repo: github.com/shibiaowan/SHARP
- name: SC3
  role: Single-cell clustering baseline installed from GitHub for the benchmark
  repo: github.com/hemberg-lab/SC3
- name: CIDR
  role: Single-cell clustering baseline (version 0.1.5) run on the preprocessed datasets
  repo: github.com/VCCRI/CIDR
- name: Monocle
  role: Single-cell clustering baseline (R package version 2.18.0)
  repo: github.com/cole-trapnell-lab/monocle-release
- name: PhenoGraph
  role: Single-cell clustering baseline via the Rphenograph R package (version 0.99.1)
  repo: github.com/JinmiaoChenLab/Rphenograph
- name: Seurat
  role: Single-cell clustering baseline referenced among the compared algorithms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:8beb355d0cad91a3f2400ac6508fcfed2b04a761c3b7c2ed4ed6cd1e1f982553
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
  - Seurat
  - scEFSC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# benchmark-comparison-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a method-by-dataset comparison table of clustering performance (ARI and NMI against published cell-type labels) for a proposed method (scEFSC) versus ensemble baselines (SAME-clustering, scCCESS, RSEC) and single-cell clustering algorithms (SHARP, SC3, CIDR, Monocle, RaceID, PhenoGraph, Seurat) on fourteen published scRNA-seq datasets.

## When to use

Use when a new clustering method must be benchmarked against existing ensemble and single-cell baselines across multiple datasets and an ARI/NMI comparison table with per-dataset rankings is the deliverable.

## When NOT to use

- The evaluation goal is a single method on a single dataset with no baselines to compare — a ranking table has no comparative meaning
- The reference labels are unavailable or ambiguous (e.g., cells labeled ambiguously as 'abandoned' must first be removed, otherwise ARI/NMI against labels is undefined or biased)
- The task is biological interpretation (differential expression, GO/KEGG enrichment) rather than quantitative benchmarking

## Inputs

- Preprocessed scRNA-seq expression matrices for the fourteen public datasets (e.g., GSE36552, GSE83139, GSE81861, GSE59739, GSE81252, GSE81608, GSE71585, GSE85241, E-MTAB-5061, GSE65525) with rows as genes and columns as cells
- Published cell-type labels for each dataset (reference labels used for ARI/NMI)
- Clustering result label vectors for each method-dataset pair (from scEFSC and from SAME-clustering, scCCESS, RSEC and the nine single-cell clustering algorithms)

## Outputs

- ARI/NMI comparison table with one value per method-dataset combination
- Per-dataset ranking of scEFSC relative to the other methods
- Summary statement of wins (e.g., best NMI on 11 of 14 datasets; best against SAME and RSEC on all 14, best against scCCESS on 12 of 14)

## How to apply

Run the proposed method (scEFSC: log2 transformation when expression values exceed 100, removal of genes detected in <2% of cells, non-negative kernel autoencoder pre-selection of 5000 genes, Low Variance/Laplacian Score/SPEC/MCFS feature selections, nine clustering algorithms, removal of least-diverse labels, wMetaC weighted meta-clustering) and each baseline on the same preprocessed datasets. For every method-dataset pair, compute ARI and NMI against the previously published cell-type labels. Assemble the comparison table including each dataset's ranking of the proposed method relative to the other methods. Validate that the table is complete for all method-dataset combinations and that summary claims (e.g., best NMI on eleven of fourteen datasets among ten algorithms) can be verified from the table entries.

## Related tools

- **scEFSC** (Proposed ensemble feature-selection clustering method whose ARI/NMI values fill the comparison table; code and pipeline from its GitHub repository) — https://github.com/Conan-Bian/scEFSC
- **SAME-clustering** (Ensemble clustering baseline run on the same preprocessed fourteen datasets to populate comparison columns)
- **SHARP** (Single-cell clustering baseline; SHARP README shows ARI computation against reference labels as the evaluation pattern) — github.com/shibiaowan/SHARP
- **SC3** (Single-cell clustering baseline installed from GitHub for the benchmark) — github.com/hemberg-lab/SC3
- **CIDR** (Single-cell clustering baseline (version 0.1.5) run on the preprocessed datasets) — github.com/VCCRI/CIDR
- **Monocle** (Single-cell clustering baseline (R package version 2.18.0)) — github.com/cole-trapnell-lab/monocle-release
- **PhenoGraph** (Single-cell clustering baseline via the Rphenograph R package (version 0.99.1)) — github.com/JinmiaoChenLab/Rphenograph
- **Seurat** (Single-cell clustering baseline referenced among the compared algorithms)

## Examples

```
res = SHARP(scExp, rN.seed=2103); ARI(label, res)
```

## Evaluation signals

- The comparison table contains both ARI and NMI entries for every method-dataset combination (ten algorithms × fourteen datasets), with no missing cells
- Summary statistics computed from the table reproduce the reported finding: scEFSC achieves the best NMI on eleven of the fourteen datasets among the ten compared single-cell clustering algorithms
- Per-dataset rankings are consistent with the reported head-to-head results: scEFSC best against SAME and RSEC on all 14 datasets and against scCCESS on 12 of 14
- Metrics are computed against the previously published cell-type labels, and cells labeled ambiguously as 'abandoned' are excluded before scoring
- Preprocessing invariants hold before scoring: log2 (base 2) transformation applied when gene expression values exceed 100, and genes detected in less than 2% of cells removed

## Limitations

- The underlying scEFSC benchmark is limited to fourteen specific published scRNA-seq datasets; rankings may not generalize to other datasets or technologies
- No quantitative ARI/NMI results appear in some article sections (e.g., discussion only claims superiority in terms of ARI and NMI without numbers), so table values must come from the results section or repository
- Ambiguously labeled cells ('abandoned') must be removed first or reference-based metrics will be distorted; the article also notes it is difficult for users to decide which clustering algorithm is most appropriate for a given dataset since each has strengths and weaknesses on particular datasets

## Evidence

- [results] scEFSC achieves the best NMI on eleven of the fourteen datasets: "of the ten single-cell clustering algorithms, scEFSC provided the best clustering results on eleven of the fourteen datasets"
- [results] Preprocessing threshold applied before benchmarking: "performed a logarithmic transformation (base 2) to rescale the data if its gene expression values were higher than 100"
- [results] Ambiguous cells must be excluded before computing reference-based metrics: "we removed cells labeled ambiguously as ”abandoned”"
- [readme] ARI against reference labels is the evaluation pattern for each method-dataset pair: [evidence span withheld — anchor status: external_doc]
- [abstract] Rationale for benchmarking against multiple algorithms on fourteen datasets: "a systematic and extensive performance evaluation of four feature selection methods and nine scRNA-seq clustering algorithms on fourteen real single-cell RNA-seq datasets"
