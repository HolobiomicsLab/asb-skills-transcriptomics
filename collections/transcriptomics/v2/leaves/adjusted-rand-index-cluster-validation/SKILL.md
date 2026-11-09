---
name: adjusted-rand-index-cluster-validation
description: Use when assessing whether repeated runs of an unsupervised clustering
  method (e.g., RCC with ConsensusClusterPlus/k-means base) yield consistent cluster
  counts and concordant assignments on bulk or single-cell expression matrices.
when_to_use_negative:
- Input lacks a reference clustering or repeat-run assignments, so no concordance
  measure (ARI) can be computed
- The goal is supervised subtype classification against known labels rather than unsupervised
  stability assessment of RCC runs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RCC (Recursive Consensus Clustering)
  role: Unsupervised clustering algorithm whose repeat-run cluster counts and assignments
    are validated via ARI
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm (consensus clustering with k-means) used by RCC in the repeat-run
    stability configuration
- name: clue
  role: R package installed per the README that provides partition-comparison utilities
    for computing adjusted Rand index
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:298e1b0035c5d6aae9398231b688d1bb8f814bedf96e894db2f914e81e4a2f18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - RCC (Recursive Consensus Clustering)
  - RCC_clus
  - clue
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# adjusted-rand-index-cluster-validation

## Summary

Validates the stability and reproducibility of Recursive Consensus Clustering (RCC) runs by comparing cluster counts and adjusted Rand index (ARI) values across repeated RCC_clus executions on the paper's seven benchmark transcriptome datasets.

## When to use

Use when assessing whether repeated runs of an unsupervised clustering method (e.g., RCC with ConsensusClusterPlus/k-means base) yield consistent cluster counts and concordant assignments on bulk or single-cell expression matrices.

## When NOT to use

- Input lacks a reference clustering or repeat-run assignments, so no concordance measure (ARI) can be computed
- The goal is supervised subtype classification against known labels rather than unsupervised stability assessment of RCC runs

## Inputs

- Expression matrices for Ivy GAP, Human tissue, TCGA pan-cancer, Biase, Pollen, Darmanis, and Neftel datasets (R objects loaded in R)
- RCC config_file specifying repeat settings and ConsensusClusterPlus (k-means) as base algorithm
- Reference clustering assignments from a randomly selected RCC run

## Outputs

- Final cluster count per dataset (expected: 66, 144, 124, 6, 13, 43, 80)
- ARI values for each repeat vs. the reference run (expected range 0.5–1)
- rcc_benchmark_results.csv with cluster counts and ARI values
- Summary bar plot in the style of Figure 2A

## How to apply

Load each expression matrix (Ivy GAP, Human tissue, TCGA pan-cancer, Biase, Pollen, Darmanis, Neftel) in R and run RCC_clus(x, config_file) with the config set to the paper's repeat settings using ConsensusClusterPlus (k-means) as the base algorithm. Record the final number of clusters RCC identifies per dataset. Compute the adjusted Rand index (ARI) between each repeat and a randomly selected reference run — the `clue` package installed per the README provides ARI computation for cluster partitions. Benchmark the reproduced cluster counts against the reported values (66 Ivy GAP, 144 Human tissue, 124 TCGA pan-cancer, 6 Biase, 13 Pollen, 43 Darmanis, 80 Neftel) and confirm each ARI falls within the reported 0.5–1 range. Write results to rcc_benchmark_results.csv and render a summary bar plot in the style of Figure 2A.

## Related tools

- **RCC (Recursive Consensus Clustering)** (Unsupervised clustering algorithm whose repeat-run cluster counts and assignments are validated via ARI) — github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm (consensus clustering with k-means) used by RCC in the repeat-run stability configuration)
- **clue** (R package installed per the README that provides partition-comparison utilities for computing adjusted Rand index)

## Examples

```
RCC_clus(x, config_file)
```

## Evaluation signals

- Reproduced cluster counts exactly match the reported values: 66 (Ivy GAP), 144 (Human tissue), 124 (TCGA pan-cancer), 6 (Biase), 13 (Pollen), 43 (Darmanis), 80 (Neftel)
- Every per-dataset ARI between a repeat run and the randomly selected reference run falls within 0.5–1
- rcc_benchmark_results.csv exists with columns for dataset, cluster count, and ARI for all seven datasets
- Summary bar plot visually matches the style of Figure 2A
- RCC_clus was run with the config_file set to the paper's repeat settings (ConsensusClusterPlus, k-means base)

## Limitations

- One limitation of the RCC algorithm is computational time, dependent on the underlying base algorithm (consensus clustering with k-means), so repeated-run benchmarking is costly on large datasets
- Analysis of very large integrated datasets (millions of cells) is not yet demonstrated; combining with scanorama and geometric sketching is proposed as future work
- ARI values of 0.5 at the lower bound of the reported range indicate partial, not perfect, assignment concordance between runs

## Evidence

- [abstract] RCC abstract claim: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
- [methods] RCC_clus invocation signature: "RCC_clus(x, config_file)"
- [readme] R package dependency install including clue for partition comparison: "install.packages(c("devtools", "clv", "fields", "matrixStats", "data.table", "cluster", "clue", "circlize", "gdata"))"
- [discussion] RCC reproducibility via a few commands: "using our algorithm, RCC, available as an R package, with just a few commands we can recapitulate findings obtained through multi-step complex analysis"
