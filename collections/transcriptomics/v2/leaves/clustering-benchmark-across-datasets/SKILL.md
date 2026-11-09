---
name: clustering-benchmark-across-datasets
description: Use when you need to demonstrate that an unsupervised clustering method
  yields consistent cluster counts and concordant assignments across repeated runs
  and across multiple bulk or single-cell transcriptome datasets.
when_to_use_negative:
- The input is supervised classification or the sample annotations are known and fixed
  — this skill evaluates unsupervised clustering stability, not predictive accuracy.
- The dataset contains millions of cells and needs integration/subsampling first —
  the paper notes the next step is analysis of large integrated datasets, and RCC's
  computational time is a stated limitation.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: RCC (Recursive Consensus Clustering)
  role: Unsupervised clustering algorithm under benchmark; run repeatedly on each
    dataset to test cluster-count and assignment stability
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: RCC_clus
  role: R function invoked as RCC_clus(x, config_file) to execute RCC clustering on
    each expression matrix
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm (with k-means) used by RCC to divide the data during recursive
    consensus clustering
- name: R
  role: Language/environment in which the RCC package, ARI computation, CSV export,
    and bar plot rendering are performed
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:1cfed01dad0ad00fd00e833b3872ed395e48781c138b52470838ac2ac2d39d0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - RCC (Recursive Consensus Clustering)
  - RCC_clus
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clustering-benchmark-across-datasets

## Summary

Benchmark a clustering algorithm's run-to-run stability and cluster-count reproducibility by repeating it on multiple transcriptomic datasets (bulk and single-cell) and scoring agreement between runs with the adjusted Rand index (ARI). This skill operationalizes the RCC paper's stability evaluation across Ivy GAP, Human tissue, TCGA pan-cancer, Biase, Pollen, Darmanis and Neftel datasets.

## When to use

Use when you need to demonstrate that an unsupervised clustering method yields consistent cluster counts and concordant assignments across repeated runs and across multiple bulk or single-cell transcriptome datasets.

## When NOT to use

- The input is supervised classification or the sample annotations are known and fixed — this skill evaluates unsupervised clustering stability, not predictive accuracy.
- The dataset contains millions of cells and needs integration/subsampling first — the paper notes the next step is analysis of large integrated datasets, and RCC's computational time is a stated limitation.

## Inputs

- Expression matrices for seven transcriptomic datasets: Ivy GAP, Human tissue, TCGA pan-cancer, Biase, Pollen, Darmanis, Neftel (bulk and single-cell)
- RCC config_file specifying repeat settings with ConsensusClusterPlus (k-means) as base algorithm
- Reference cluster assignments (e.g., a randomly selected run's clustering) for ARI computation

## Outputs

- Per-dataset cluster counts from repeated RCC runs
- Pairwise adjusted Rand index (ARI) values between each run and a randomly selected run
- rcc_benchmark_results.csv with cluster counts and ARI values for all seven datasets
- Summary bar plot in the style of Figure 2A

## How to apply

Load each expression matrix (e.g., Ivy GAP, Human tissue, TCGA pan-cancer, Biase, Pollen, Darmanis, Neftel) in R and run RCC_clus(x, config_file) on each, setting the config_file to the repeat settings with ConsensusClusterPlus (k-means) as the base algorithm, since RCC uses ConsensusClusterPlus (CCP) as the base algorithm for dividing the data. Record the final number of clusters identified by RCC per dataset. Compute the adjusted Rand index (ARI) of each run's clusters against a randomly selected reference run to quantify run-to-run concordance. Compare reproduced cluster counts and ARI values against reported benchmarks (66 Ivy GAP, 144 Human tissue, 124 TCGA pan-cancer, 6 Biase, 13 Pollen, 43 Darmanis, 80 Neftel; ARI range 0.5–1). Write results to a CSV and render a summary bar plot styled after the paper's Figure 2A; the benchmark is validated when reproduced cluster counts match reported values and each dataset's ARI falls within the 0.5–1 range.

## Related tools

- **RCC (Recursive Consensus Clustering)** (Unsupervised clustering algorithm under benchmark; run repeatedly on each dataset to test cluster-count and assignment stability) — github.com/MSCTR/RecursiveConsensusClustering
- **RCC_clus** (R function invoked as RCC_clus(x, config_file) to execute RCC clustering on each expression matrix) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm (with k-means) used by RCC to divide the data during recursive consensus clustering)
- **R** (Language/environment in which the RCC package, ARI computation, CSV export, and bar plot rendering are performed) — https://github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
library(RCC); rcc <- RCC_clus(x, config_file); k <- length(unique(rcc$clusters)); aris <- sapply(runs, function(r) aricode::ARI(rcc$clusters, r))
```

## Evaluation signals

- Reproduced cluster counts exactly equal reported values: 66 (Ivy GAP), 144 (Human tissue), 124 (TCGA pan-cancer), 6 (Biase), 13 (Pollen), 43 (Darmanis), 80 (Neftel).
- Each dataset's ARI against a randomly selected run falls within the reported range 0.5–1.
- rcc_benchmark_results.csv contains cluster counts and ARI values for all seven datasets.
- Summary bar plot is rendered in the style of Figure 2A of the paper.
- ConsensusClusterPlus with k-means is confirmed as the base algorithm in the config_file for all runs.

## Limitations

- One of the limitations of the RCC algorithm is computational time, which is dependent on the underlying base algorithm (consensus clustering with k-means).
- Analysis of large integrated datasets with millions of cells is the next step, not yet demonstrated.
- Stability (ARI 0.5–1) is measured against a randomly selected run, not against gold-standard external annotations, so it captures run-to-run consistency rather than biological correctness.

## Evidence

- [abstract] RCC is an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
- [intro] RCC uses ConsensusClusterPlus as base algorithm: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [methods] RCC_clus invocation: "RCC_clus(x, config_file)"
- [discussion] Computational time limitation: "One of the limitations of our algorithm is computational time."
- [readme] README installation instructions for RCC: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
