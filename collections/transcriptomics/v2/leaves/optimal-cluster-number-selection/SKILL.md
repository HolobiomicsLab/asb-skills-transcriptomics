---
name: optimal-cluster-number-selection
description: Use when clustering transcriptome data (bulk or single-cell expression
  matrix) with an unknown number of clusters and you need to determine the k for each
  recursive subdivision step.
when_to_use_negative:
- The number of clusters is already known and only point-assignment (e.g. fixed-k
  k-means or hierarchical cut) is needed
- Input is not transcriptomic (e.g. already a pre-computed distance matrix or a feature/annotation
  table rather than an expression matrix)
- A quick single-pass clustering is needed and the computational time of repeated
  consensus clustering runs is prohibitive
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: RCC
  role: Wrapper algorithm that recursively runs consensus clustering and applies the
    best-k selection and termination criteria
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used for dividing the data over candidate k values, generating
    the consensus matrices and CDF plots
- name: geneMarkers
  role: Interface used to evaluate the differentially-expressed-gene criterion at
    the selected k
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:1dad6ed3936792e985d850c1edda77b4a867e007ffdfbfcb0a1ddee7002fdbec
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# optimal-cluster-number-selection

## Summary

Algorithmically selects the optimal number of clusters (k) in transcriptomic data by running ConsensusClusterPlus with k-means over candidate k values, reading cluster stability from empirical CDF plots of the consensus distributions, and gating candidates on stability, overlap, and differentially-expressed-gene criteria.

## When to use

Use when clustering transcriptome data (bulk or single-cell expression matrix) with an unknown number of clusters and you need to determine the best k for each recursive subdivision step.

## When NOT to use

- The number of clusters is already known and only point-assignment (e.g. fixed-k k-means or hierarchical cut) is needed
- Input is not transcriptomic (e.g. already a pre-computed distance matrix or a feature/annotation table rather than an expression matrix)
- A quick single-pass clustering is needed and the computational time of repeated consensus clustering runs is prohibitive

## Inputs

- Expression matrix (x) of transcriptome data (bulk or single-cell)
- RCC configuration file (config_file) with clustering parameters
- Candidate k values for consensus clustering

## Outputs

- Selected optimal k per clustering run
- CDF/delta-area consensus clustering plots indicating cluster stability
- Cluster assignments passing stability, overlap, and DE-gene gates
- Recursive subdivision tree of subtypes with termination points

## How to apply

Load the expression matrix (x) and configuration file as consumed by RCC_clus(x, config_file), then run ConsensusClusterPlus (the base algorithm, with k-means) over candidate k values. Compute the empirical CDF of the consensus distributions and select candidate best-k values by requiring the delta-area plateau to satisfy line length >= 30 and slope < 10 degrees. At the selected k, apply quality gates: intra-cluster stability > 0.8, inter-cluster overlap < 0.2, and a differentially-expressed-gene criterion (e.g. via the geneMarkers(inputDir, exprsFile, sampleOrd, outputDir) interface). Encode recursive subdivision so that splitting stops when no candidate k passes all CDF, stability, overlap, and DE-gene criteria, thereby revealing finer hierarchical subtypes.

## Related tools

- **RCC** (Wrapper algorithm that recursively runs consensus clustering and applies the best-k selection and termination criteria) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used for dividing the data over candidate k values, generating the consensus matrices and CDF plots)
- **geneMarkers** (Interface used to evaluate the differentially-expressed-gene criterion at the selected k) — https://github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
devtools::install_github("MSCTR/RecursiveConsensusClustering"); RCC_clus(x, config_file)
```

## Evaluation signals

- The implemented best-k selector returns a k whose delta-area plateau satisfies line length >= 30 and slope < 10 degrees
- At the chosen k, intra-cluster stability > 0.8 and inter-cluster overlap < 0.2
- The differentially-expressed-gene criterion (e.g. via geneMarkers) is satisfied at the selected k
- The recursion terminates without subdividing a cluster that fails no criterion
- The module runs end-to-end on a test expression matrix via RCC_clus(x, config_file) without errors

## Limitations

- Computational time of RCC is a limitation; it depends on the underlying base algorithm (consensus clustering with k-means)
- Performance is tied to ConsensusClusterPlus with k-means as the current base algorithm
- Analysis of large integrated datasets with millions of cells is still the next step, not yet demonstrated

## Evidence

- [intro] RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data.: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [methods] RCC_clus invocation in methods: "RCC_clus(x, config_file)"
- [methods] geneMarkers interface in methods: "geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)"
- [full_text] CDF-based best-k finding from source card: [evidence span withheld — anchor status: not_found]
- [discussion] computational time limitation: "One of the limitations of our algorithm is computational time."
- [readme] README install instructions: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
