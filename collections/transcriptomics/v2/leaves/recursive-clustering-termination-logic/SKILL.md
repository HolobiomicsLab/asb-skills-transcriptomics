---
name: recursive-clustering-termination-logic
description: Use when implementing or reproducing RCC's decision logic for choosing
  the k per consensus clustering run and deciding when recursive cluster subdivision
  should stop on transcriptome data.
when_to_use_negative:
- Input is not an expression matrix (e.g. already-clustered labels, metadata tables,
  or feature counts without samples-by-genes structure)
- The number of clusters is known beforehand and no novel subtype discovery is needed
  — a fixed-k clustering method suffices
- Datasets so large that consensus clustering runtime is prohibitive; the article
  notes computational time is a limitation dependent on the k-means base algorithm
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: Hosts the recursive consensus clustering loop, best-k selection, and termination
    criteria via RCC_clus(x, config_file)
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used to divide the data and generate consensus CDF plots over
    candidate k values
- name: R
  role: Implementation language of the RCC package and the best-k/termination module
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:cdd7425e5b8dbc2e97735e54b5b6bed18d7c49fda13df0532ab4855b8d614724
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# recursive-clustering-termination-logic

## Summary

Encodes the optimal-k selection and recursive subdivision termination rules of Recursive Consensus Clustering (RCC), which stops subdividing a cluster when no candidate k passes CDF-based stability, intra-/inter-cluster quality, and differentially-expressed-gene gates.

## When to use

Use when implementing or reproducing RCC's decision logic for choosing the best k per consensus clustering run and deciding when recursive cluster subdivision should stop on transcriptome data.

## When NOT to use

- Input is not an expression matrix (e.g. already-clustered labels, metadata tables, or feature counts without samples-by-genes structure)
- The number of clusters is known beforehand and no novel subtype discovery is needed — a fixed-k clustering method suffices
- Datasets so large that consensus clustering runtime is prohibitive; the article notes computational time is a limitation dependent on the k-means base algorithm

## Inputs

- Gene expression matrix x (bulk or single-cell transcriptome data)
- RCC configuration file (config_file)
- Candidate k values for consensus clustering runs

## Outputs

- Selected optimal k per clustering run
- Empirical CDF plots / delta-area plots of consensus distributions
- Cluster membership assignments with stability and overlap metrics
- Termination decision per cluster (subdivide vs. stop) and final recursive subtype hierarchy
- rcc_bestk_module.R module with unit checks

## How to apply

Run ConsensusClusterPlus (with k-means as the base algorithm) over candidate k values on the expression matrix x as consumed by RCC_clus(x, config_file). For each parallel run, compute the empirical CDF of the consensus distributions and select the best k from the CDF/delta-area plateau, requiring line length >= 30 and slope < 10 degrees. At the selected k, apply cluster quality gates: intra-cluster stability > 0.8 and inter-cluster overlap < 0.2, plus a differentially-expressed-gene criterion (e.g. via geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)). Encode the recursive termination rule so subdivision of a cluster stops when no candidate k satisfies all gates simultaneously (CDF, stability, overlap, DE genes); a cluster that fails no criterion is subdivided further. The rationale is that the CDF plateau indicates cluster stability while the stability/overlap/DE-gene gates ensure the subdivision is biologically meaningful before recursing.

## Related tools

- **RCC** (Hosts the recursive consensus clustering loop, best-k selection, and termination criteria via RCC_clus(x, config_file)) — github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used to divide the data and generate consensus CDF plots over candidate k values)
- **R** (Implementation language of the RCC package and the best-k/termination module)

## Examples

```
devtools::install_github("MSCTR/RecursiveConsensusClustering"); res <- RCC_clus(x, config_file)
```

## Evaluation signals

- On a test expression matrix, the best-k selector returns a k that passes all threshold gates: delta-area line length >= 30, slope < 10 degrees, intra-cluster stability > 0.8, inter-cluster overlap < 0.2
- The recursion terminates without subdividing a cluster that fails no criterion
- The implemented module, when run via RCC_clus(x, config_file), executes without error on a sample input from msctr.org
- Cluster quality metrics at the selected k fall within the specified ranges (stability > 0.8, overlap < 0.2)
- Differentially-expressed-gene check via geneMarkers(inputDir, exprsFile, sampleOrd, outputDir) yields markers supporting the selected partition

## Limitations

- Computational time of RCC is a limitation; runtime depends on the underlying base algorithm, currently consensus clustering with k-means
- Analysis of large integrated datasets with millions of cells is not yet demonstrated (a stated next step)
- The best-k gates are threshold-based heuristics (line length >= 30, slope < 10 degrees, stability > 0.8, overlap < 0.2); data failing these cutoffs may terminate prematurely

## Evidence

- [intro] RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data.: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [full_text] For each parallel run, RCC determines candidate best k values from cumulative distribution function (CDF) plots (indicative of cluster stability) generated by ConsensusClusterPlus, satisfying the best-k selection criteria.: [evidence span withheld — anchor status: not_found]
- [intro] The recursive clustering of the dataset reveals finer structures in the data leading to the identification of novel subtypes.: "The recursive clustering of the dataset reveals finer structures in the data leading to the identification of novel subtypes."
- [discussion] One of the limitations of our algorithm is computational time. This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means.: "This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means."
- [methods] RCC_clus(x, config_file) is the entry point consuming the expression matrix and configuration file.: "RCC_clus(x, config_file)"
- [readme] README confirms installation dependencies and the package repository for RecursiveConsensusClustering.: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
