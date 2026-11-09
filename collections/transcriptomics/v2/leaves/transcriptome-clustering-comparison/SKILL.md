---
name: transcriptome-clustering-comparison
description: Use when you need to quantify whether parallelizing RCC's consensus clustering
  across multiple CCP processes changes clustering accuracy (ARI) or processing time
  relative to a single-process run with more repeats.
when_to_use_negative:
- When the goal is subtype discovery rather than benchmarking runtime/accuracy — use
  standard RCC_clus with a single default configuration instead
- When no reference partition exists to compute an adjusted Rand index against, since
  the accuracy side of the comparison cannot be quantified
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: R package providing RCC_clus(x, config_file) to run recursive consensus clustering
    under parallel or single-process configurations
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: ConsensusClusterPlus
  role: Base consensus clustering algorithm (CCP) whose processes are parallelized
    and whose pItem/pFeature subsampling parameters are varied
- name: mclust
  role: Source of the adjusted Rand index used to quantify agreement of each clustering
    with the reference partition
- name: pforeach
  role: R parallelization backend installed per the README to run multiple CCP processes
    concurrently
  repo: https://github.com/hoxo-m/pforeach
- name: R
  role: Execution environment in which RCC_clus is invoked and ARI/runtime are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:0fbfe3db71eafc38bf39d580ce4e77a958878d9ffc19071519a57846f47f4529
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - mclust
  - pforeach
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# transcriptome-clustering-comparison

## Summary

Benchmark parallel RCC execution (eight concurrent ConsensusClusterPlus CCP processes with variant pItem/pFeature subsampling, 100 repeats each) against a single-process run with default pItem/pFeature and 800 repetitions, comparing adjusted Rand index (ARI) and wall-clock processing time on the TCGA pan-cancer expression matrix.

## When to use

Use when you need to quantify whether parallelizing RCC's consensus clustering across multiple CCP processes changes clustering accuracy (ARI) or processing time relative to a single-process run with more repeats.

## When NOT to use

- When the goal is subtype discovery rather than benchmarking runtime/accuracy — use standard RCC_clus with a single default configuration instead
- When no reference partition exists to compute an adjusted Rand index against, since the accuracy side of the comparison cannot be quantified

## Inputs

- TCGA pan-cancer transcriptome expression matrix (samples x genes)
- RCC configuration file specifying number of parallel processes, pItem/pFeature values, and repeat count
- Reference partition of samples (e.g., cancer-type labels) for ARI computation

## Outputs

- Cluster assignments from each RCC_clus run
- Wall-clock processing time for each configuration
- ARI values comparing each clustering to the reference partition
- Comparison table (ARI and processing time for parallel vs single-process) saved as a CSV file (Table S2 style)

## How to apply

Load the TCGA pan-cancer expression matrix and the RCC R package, then invoke RCC_clus(x, config_file) twice with different configurations. Configuration A uses 8 parallel processes (via pforeach), variant pItem/pFeature subsampling fractions, and 100 repeats per process; Configuration B uses a single process, default pItem/pFeature, and 800 repetitions. Record each run's cluster assignments and wall-clock processing time. Compute the adjusted Rand index (ARI) of each run's clustering against a reference partition (e.g., known cancer-type labels) using the mclust/clue ARI implementation, then assemble ARI and time for both settings into a comparison table saved as CSV (Table S2 style). The rationale is that splitting repeats across parallel CCP processes can preserve or improve agreement with the reference while reducing total runtime, since RCC's computational time is tied to its consensus-clustering-with-k-means base algorithm.

## Related tools

- **RCC** (R package providing RCC_clus(x, config_file) to run recursive consensus clustering under parallel or single-process configurations) — github.com/MSCTR/RecursiveConsensusClustering
- **ConsensusClusterPlus** (Base consensus clustering algorithm (CCP) whose processes are parallelized and whose pItem/pFeature subsampling parameters are varied)
- **mclust** (Source of the adjusted Rand index used to quantify agreement of each clustering with the reference partition)
- **pforeach** (R parallelization backend installed per the README to run multiple CCP processes concurrently) — https://github.com/hoxo-m/pforeach
- **R** (Execution environment in which RCC_clus is invoked and ARI/runtime are computed)

## Examples

```
RCC_clus(x, config_file)
```

## Evaluation signals

- ARI of the parallel configuration is higher than or comparable to the single-process 800-repeat run, as reported in Table S2
- Wall-clock processing time of the 8-process/100-repeat configuration is lower than the single-process 800-repeat run
- Both runs produce cluster assignments for the same TCGA pan-cancer samples, enabling a like-for-like ARI comparison against the same reference partition
- Comparison table contains exactly two rows/columns of settings (parallel vs single-process) each with paired ARI and time values, exported as CSV

## Limitations

- RCC's computational time is a stated limitation: 'One of the limitations of our algorithm is computational time', and this time depends on the underlying base algorithm (consensus clustering with k-means)
- Results are specific to the TCGA pan-cancer transcriptome dataset and the chosen reference partition; ARI may differ on other datasets or references

## Evidence

- [intro] RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data.: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [methods] RCC_clus(x, config_file): "RCC_clus(x, config_file)"
- [discussion] One of the limitations of our algorithm is computational time.: "One of the limitations of our algorithm is computational time."
- [discussion] This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means.: "This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means."
- [intro] popular clustering methods like k-means, hierarchical clustering, pam, mclust3, etc.: "popular clustering methods like k-means, hierarchical clustering, pam, mclust3, etc."
- [readme] devtools::install_github("hoxo-m/pforeach"): "devtools::install_github("hoxo-m/pforeach")"
