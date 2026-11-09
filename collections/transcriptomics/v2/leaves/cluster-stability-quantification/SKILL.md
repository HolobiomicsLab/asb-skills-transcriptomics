---
name: cluster-stability-quantification
description: Use when clustering transcriptomic (bulk or single-cell) data with an
  unknown number of clusters and you must algorithmically determine the optimal k
  and decide when to stop recursive subdivision.
when_to_use_negative:
- The number of clusters is known beforehand and the goal is only to partition data,
  not to discover novel subtypes or select k
- The analysis goal is supervised classification against known annotations rather
  than unsupervised subtype discovery; RCC targets data with an unknown number of
  clusters
- Datasets with millions of cells — the authors note computational time is a limitation
  tied to the consensus clustering with k-means base algorithm, and large integrated
  datasets are still a next step
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: Recursive Consensus Clustering framework that performs the parallel consensus
    clustering runs, best-k selection, and recursive subdivision; invoked via RCC_clus(x,
    config_file)
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used inside RCC for dividing the data over candidate k values,
    generating the consensus matrices whose CDFs indicate cluster stability
- name: geneMarkers
  role: Interface used to evaluate the differentially-expressed-gene criterion at
    the selected k
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: Language/environment in which the RCC package implementing this stability
    quantification runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:4be749ebed68dc58d05c1406b2529e2bd43a62c9764f54a93ae8db597554e7d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - geneMarkers
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# cluster-stability-quantification

## Summary

Quantifies cluster stability to select the optimal number of clusters (k) during each parallel consensus clustering run of RCC, using empirical CDF plots from ConsensusClusterPlus plus stability, overlap, and differential-expression quality gates, enabling recursive subdivision of transcriptome data into novel subtypes.

## When to use

Use when clustering transcriptomic (bulk or single-cell) data with an unknown number of clusters and you must algorithmically determine the optimal k and decide when to stop recursive subdivision.

## When NOT to use

- The number of clusters is known beforehand and the goal is only to partition data, not to discover novel subtypes or select k
- The analysis goal is supervised classification against known annotations rather than unsupervised subtype discovery; RCC targets data with an unknown number of clusters
- Datasets with millions of cells — the authors note computational time is a limitation tied to the consensus clustering with k-means base algorithm, and large integrated datasets are still a next step

## Inputs

- Gene expression matrix (x), bulk or single-cell transcriptome data
- RCC configuration file (config_file) specifying candidate k values and algorithm parameters
- Ordered gene expression file (exprsFile) and sample order file (sampleOrd) for geneMarkers differential-expression checks

## Outputs

- Selected optimal k per consensus clustering run (candidate best k from CDF plots)
- Cluster assignments passing stability (> 0.8) and overlap (< 0.2) gates
- CDF / delta-area plots indicating cluster stability
- Differentially-expressed gene markers per cluster from geneMarkers

## How to apply

Load the expression matrix (x) and configuration file into RCC_clus(x, config_file), which runs ConsensusClusterPlus with k-means as the base algorithm over candidate k values. Compute the empirical CDF of the consensus distributions and select the candidate best k requiring the delta-area plateau to satisfy line length >= 30 and slope < 10 degrees. At the selected k, apply quality gates: intra-cluster stability > 0.8 and inter-cluster overlap < 0.2, together with a differentially-expressed-gene criterion (e.g. via geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)). Recursively subdivide clusters only while some candidate k passes all CDF, stability, overlap, and DE-gene criteria; if none passes, terminate subdivision. This rationale is grounded in RCC's design as an unsupervised R package that addresses optimal-k selection and reveals finer hierarchical structures for novel subtype discovery.

## Related tools

- **RCC** (Recursive Consensus Clustering framework that performs the parallel consensus clustering runs, best-k selection, and recursive subdivision; invoked via RCC_clus(x, config_file)) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used inside RCC for dividing the data over candidate k values, generating the consensus matrices whose CDFs indicate cluster stability)
- **geneMarkers** (Interface used to evaluate the differentially-expressed-gene criterion at the selected k) — https://github.com/MSCTR/RecursiveConsensusClustering
- **R** (Language/environment in which the RCC package implementing this stability quantification runs)

## Examples

```
devtools::install_github("MSCTR/RecursiveConsensusClustering"); RCC_clus(x, config_file)
```

## Evaluation signals

- The implemented best-k selector returns a k whose delta-area plateau satisfies line length >= 30 and slope < 10 degrees
- At the selected k, intra-cluster stability exceeds 0.8 and inter-cluster overlap is below 0.2
- The selected k also passes the differentially-expressed-gene criterion (e.g. non-trivial marker sets via geneMarkers)
- Recursion terminates without subdividing any cluster that failed no criterion (no subdivision when all quality gates fail)
- The module runs end-to-end on a sample input via RCC_clus(x, config_file) reproducing output structure

## Limitations

- Computational time is a limitation of RCC; performance depends on the underlying base algorithm, currently consensus clustering with k-means
- Analysis of very large integrated datasets with millions of cells is not yet demonstrated (next step)
- If no candidate k passes the CDF, stability, overlap, and DE-gene criteria, subdivision stops and no finer subtype is reported

## Evidence

- [full_text] CDF plateau thresholds: [evidence span withheld — anchor status: not_found]
- [full_text] quality gates: [evidence span withheld — anchor status: not_found]
- [intro] base algorithm: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [abstract] optimal k problem: "The major problem while clustering the data is the selection of the optimal number of clusters (k) for each dataset"
- [methods] entry point: "RCC_clus(x, config_file)"
- [readme] install instructions: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
