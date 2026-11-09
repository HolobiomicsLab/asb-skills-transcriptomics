---
name: benchmark-score-table-aggregation
description: Use when cluster label assignments from multiple scRNA-seq clustering
  methods must be scored against ground-truth cell-type annotations and compiled into
  a per-dataset, per-method ARI/AMI/NMI comparison table.
when_to_use_negative:
- The run involves a single clustering method without ground-truth cell-type labels,
  so no supervised metric like ARI/AMI/NMI can be computed
- The goal is the CASCC method run itself (attractor feature selection, cluster-k
  prediction, k-means) rather than cross-method benchmark scoring
- Input datasets are outside the 15 Krzak et al. 2019 evaluation datasets when the
  goal is reproducing the published Fig. 1b / Supplementary Table S3 numbers
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CASCC
  role: Clustering method under evaluation; run.CASCC produces the cluster labels
    scored against ground truth
  repo: github.com/LingyiC/CASCC
- name: aricode
  role: Computes ARI, AMI, and NMI for every dataset-method pair from predicted vs.
    ground-truth labels
- name: Seurat 4.3.0
  role: Pinned dependency and baseline clustering method (Seurat V4) in the benchmark
    environment
- name: cafr
  role: Dependency providing the adaptive attractor algorithm used by CASCC's co-expression
    feature selection
  repo: github.com/weiyi-bitw/cafr
- name: CASCC_benchmark
  role: Repository supplying the benchmark code and the 15 evaluation datasets with
    ground-truth labels
  repo: https://github.com/LingyiC/CASCC_benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:1187140af4a5419dda1dcdb6993c4d2f82489815933fe8d593a86485322b33f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASCC
  - CASCC_benchmark
  - R
  - Seurat 4.3.0
  - aricode
  - cafr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-score-table-aggregation

## Summary

Aggregates per-dataset, per-method clustering scores (ARI, AMI, NMI) from the CASCC benchmark into a single score table, enabling direct comparison of CASCC against CIDR, RaceID3, SC3, Seurat V4, SIMLR, and TSCAN on the 15 Krzak et al. 2019 scRNA-seq datasets.

## When to use

Use when cluster label assignments from multiple scRNA-seq clustering methods must be scored against ground-truth cell-type annotations and compiled into a per-dataset, per-method ARI/AMI/NMI comparison table.

## When NOT to use

- The run involves a single clustering method without ground-truth cell-type labels, so no supervised metric like ARI/AMI/NMI can be computed
- The goal is the CASCC method run itself (attractor feature selection, cluster-k prediction, k-means) rather than cross-method benchmark scoring
- Input datasets are outside the 15 Krzak et al. 2019 evaluation datasets when the goal is reproducing the published Fig. 1b / Supplementary Table S3 numbers

## Inputs

- Cluster label vectors (predicted) from CASCC::run.CASCC and the six baseline methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN)
- Ground-truth cell-type annotation vectors for the 15 real scRNA-seq evaluation datasets from the LingyiC/CASCC_benchmark repository
- Normalized log-transformed scRNA-seq count matrices for each dataset

## Outputs

- benchmark_ari_ami_nmi_scores.csv — per-dataset, per-method table of ARI, AMI, and NMI values
- Mean ARI summary per method for comparison against Fig. 1b and Supplementary Table S3

## How to apply

Load each of the 15 real scRNA-seq evaluation datasets (with ground-truth labels) from the LingyiC/CASCC_benchmark repository and run every method under the pinned environment recorded in the paper's sessionInfo (R 4.0.5, Seurat 4.3.0, cafr 0.312). For each dataset-method pair, compare predicted cluster labels to ground-truth annotations using the aricode package to compute ARI, AMI, and NMI. Aggregate all metrics into a per-dataset, per-method score table saved as benchmark_ari_ami_nmi_scores.csv. CASCC labels come from CASCC::run.CASCC on the normalized log-transformed count matrix, whose pipeline combines adaptive attractor-based feature selection, Ward.D2 hierarchical clustering with the silhouette index for cluster-number prediction, and k-means with attractor-modified centers. Validate the table by checking that computed values reproduce the per-dataset scores in Fig. 1b and Supplementary Table S3, where CASCC improved mean ARI by 34% over SC3.

## Related tools

- **CASCC** (Clustering method under evaluation; run.CASCC produces the cluster labels scored against ground truth) — github.com/LingyiC/CASCC
- **aricode** (Computes ARI, AMI, and NMI for every dataset-method pair from predicted vs. ground-truth labels)
- **Seurat 4.3.0** (Pinned dependency and baseline clustering method (Seurat V4) in the benchmark environment)
- **cafr** (Dependency providing the adaptive attractor algorithm used by CASCC's co-expression feature selection) — github.com/weiyi-bitw/cafr
- **CASCC_benchmark** (Repository supplying the benchmark code and the 15 evaluation datasets with ground-truth labels) — https://github.com/LingyiC/CASCC_benchmark

## Examples

```
aricode::ARI(truth_labels, cascc_labels)
```

## Evaluation signals

- Computed per-dataset ARI/AMI/NMI values for CASCC and each baseline reproduce the scores reported in Fig. 1b and Supplementary Table S3 of the paper
- CASCC's mean ARI across the 15 datasets exceeds the second-best method (SC3) by ~34%
- The output CSV contains rows covering all 15 datasets × 7 methods (CASCC plus CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) with three metrics each
- All runs execute under the recorded sessionInfo environment (R 4.0.5, Seurat 4.3.0, cafr 0.312) so scores are reproducible

## Limitations

- Reproduction depends on the pinned environment (R 4.0.5, Seurat 4.3.0, cafr); version drift in any dependency can change clustering labels and thus scores
- Findings cover only the 15 real scRNA-seq evaluation datasets used by Krzak et al. 2019; no claim of generalization beyond those datasets is supported here
- Fig. 1b and Supplementary Table S3 are the validation reference, but they are not machine-readable from the paper itself, so verification requires manual comparison

## Evidence

- [readme] Benchmarking link: "# Benchmarking
https://github.com/LingyiC/CASCC_benchmark"
- [methods] Pinned environment: "R version 4.0.5 (2021-03-31)"
- [intro] Seurat version pin: "remotes::install_version("Seurat", "4.3.0") # dependency"
- [intro] CASCC method description: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [methods] CASCC invocation: "res <- CASCC::run.CASCC(Data_PDAC_peng_2k)"
- [readme] Benchmark repo purpose: [evidence span withheld — anchor status: external_doc]
