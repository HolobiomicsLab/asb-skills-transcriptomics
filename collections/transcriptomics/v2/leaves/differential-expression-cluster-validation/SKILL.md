---
name: differential-expression-cluster-validation
description: Use when RCC (or a consensus clustering run over candidate k values)
  has produced clusters and you must decide whether each cluster is a genuine subtype
  before subdividing further, using differential gene expression as the quality gate.
when_to_use_negative:
- Clusters already have established biological annotation, so validation by novel
  DE genes is unnecessary
- The dataset is not transcriptomic (e.g., no gene-level expression matrix), since
  RCC is designed for bulk and single-cell transcriptome data
- A fully exploratory clustering with no interest in subtype validity is desired and
  stability/overlap/DE gates would over-filter the result
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RCC
  role: Recursive consensus clustering framework whose recursive subdivision is gated
    by the differential-expression cluster-validation criterion
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: geneMarkers
  role: 'RCC interface that computes differentially-expressed genes per cluster: geneMarkers(inputDir,
    exprsFile, sampleOrd, outputDir)'
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base algorithm used inside RCC to generate consensus clustering results over
    candidate k values that are then validated
- name: R
  role: Language in which the RCC package and validation module (rcc_bestk_module.R)
    are implemented
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:32d67c387183fb3568ae8b9708db3fe0cf4703190a0c60d60a186e14f6499150
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - geneMarkers
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# differential-expression-cluster-validation

## Summary

Validates clusters/subtypes produced by Recursive Consensus Clustering (RCC) by checking that each cluster is supported by differentially-expressed genes, computed with the RCC package's geneMarkers interface. This gates whether a subdivision of the data (or a subcluster at a candidate k) is biologically meaningful or whether recursion should stop.

## When to use

Use when RCC (or a consensus clustering run over candidate k values) has produced clusters and you must decide whether each cluster is a genuine subtype before subdividing further, using differential gene expression as the quality gate.

## When NOT to use

- Clusters already have established biological annotation, so validation by novel DE genes is unnecessary
- The dataset is not transcriptomic (e.g., no gene-level expression matrix), since RCC is designed for bulk and single-cell transcriptome data
- A fully exploratory clustering with no interest in subtype validity is desired and stability/overlap/DE gates would over-filter the result

## Inputs

- Expression matrix (x) as consumed by RCC_clus(x, config_file)
- RCC configuration file (config_file)
- Sample ordering / cluster assignment files (exprsFile, sampleOrd) for geneMarkers

## Outputs

- Per-cluster differentially-expressed gene lists used as cluster-quality evidence
- Accept/reject decision for each candidate k and each cluster sub-division
- Final set of validated subtypes after recursive subdivision terminates

## How to apply

After running consensus clustering with ConsensusClusterPlus (k-means base) inside RCC and selecting a best k from the CDF delta-area plateau (line length >= 30, slope < 10 degrees) plus stability gates (intra-cluster stability > 0.8, inter-cluster overlap < 0.2), compute differentially-expressed genes per cluster via the geneMarkers interface geneMarkers(inputDir, exprsFile, sampleOrd, outputDir). A cluster is retained as a valid subtype only if it passes the DE-gene criterion in addition to the stability and overlap thresholds; if no candidate k passes all gates (CDF, stability, overlap, DE genes), the recursive subdivision terminates. The rationale is that a cluster that fails to show differential expression is unlikely to represent a biologically meaningful subtype, echoing RCC's goal of finding novel, biologically meaningful subtypes from bulk or single-cell transcriptome data. Verify the module end-to-end by running it on a sample expression matrix through RCC_clus(x, config_file).

## Related tools

- **RCC** (Recursive consensus clustering framework whose recursive subdivision is gated by the differential-expression cluster-validation criterion) — https://github.com/MSCTR/RecursiveConsensusClustering
- **geneMarkers** (RCC interface that computes differentially-expressed genes per cluster: geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)) — https://github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base algorithm used inside RCC to generate consensus clustering results over candidate k values that are then validated)
- **R** (Language in which the RCC package and validation module (rcc_bestk_module.R) are implemented) — https://github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
RCC_clus(x, config_file); geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)
```

## Evaluation signals

- The selected k satisfies all threshold gates: CDF delta-area plateau line length >= 30 and slope < 10 degrees, intra-cluster stability > 0.8, inter-cluster overlap < 0.2, and the DE-gene criterion
- Every retained cluster has a non-empty differentially-expressed gene set from geneMarkers
- The recursion terminates and never subdivides a cluster that fails no criterion
- The module (rcc_bestk_module.R) runs without error on a sample expression matrix via RCC_clus(x, config_file)

## Limitations

- Computational time of RCC is a stated limitation, and this validation step adds cost on top of the consensus clustering with k-means base algorithm
- Typical pipelines rely on known annotations and manual cluster identification through visualization; DE-based validation cannot substitute for external clinical annotation checks

## Evidence

- [methods] geneMarkers: "geneMarkers(inputDir, exprsFile, sampleOrd, outputDir)"
- [intro] RCC uses ConsensusClusterPlus as base algorithm: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [abstract] RCC available as R package: "RCC is available as an R package and facilitates the generation of new"
- [readme] sample input and output files via MSCTR: "For sample input and ouput files please download data from: https://www.msctr.org/2019/05/30/recursive-consensus-clustering/"
