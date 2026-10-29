---
name: computation-runtime-benchmarking
description: Use when a pipeline's computational time (a stated limitation of RCC)
  can be traded against clustering accuracy by comparing parallel low-repeat runs
  against a single high-repeat run with ARI and timing measurements.
when_to_use_negative:
- When the analysis goal is subtype discovery or cluster annotation (use clusterAnnotation/clusterAttr
  workflows) rather than runtime/accuracy benchmarking
- When no reference partition exists, since ARI cannot be computed without a comparison
  partition
- When datasets with millions of cells are the target — the article notes analysis
  of large integrated single-cell datasets was still the untested next step
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RCC
  role: R package whose RCC_clus(x, config_file) call executes both the parallel and
    serial clustering configurations being benchmarked
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: Environment in which the benchmark runs and the ARI/time comparison table
    is assembled and saved as CSV
- name: ConsensusClusterPlus
  role: Base algorithm (CCP) whose process count and repeat settings are the benchmarked
    configuration parameters
- name: mclust
  role: Clustering method referenced as an alternative base method; RCC's runtime
    depends on the underlying base algorithm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:b4a511595496f9262e3394a062640dc9ee5b50bd5a5d52a5a2204bfc0d17baf3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - mclust
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# computation-runtime-benchmarking

## Summary

Benchmark the computational cost and clustering quality of alternative RCC configurations — here eight parallel ConsensusClusterPlus (CCP) processes with variant pItem/pFeature and 100 repeats versus a single process with default pItem/pFeature and 800 repeats — quantified by ARI and wall-clock processing time.

## When to use

Use when a pipeline's computational time (a stated limitation of RCC) can be traded against clustering accuracy by comparing parallel low-repeat runs against a single high-repeat run with ARI and timing measurements.

## When NOT to use

- When the analysis goal is subtype discovery or cluster annotation (use clusterAnnotation/clusterAttr workflows) rather than runtime/accuracy benchmarking
- When no reference partition exists, since ARI cannot be computed without a comparison partition
- When datasets with millions of cells are the target — the article notes analysis of large integrated single-cell datasets was still the untested next step

## Inputs

- TCGA pan-cancer expression matrix loaded in R
- RCC configuration files (parallel config: 8 processes, variant pItem/pFeature, 100 repeats; serial config: 1 process, default pItem/pFeature, 800 repeats)
- Reference partition of samples for ARI computation

## Outputs

- Cluster assignments from each RCC_clus run
- Wall-clock processing times for both configurations
- ARI values quantifying agreement with the reference partition
- Comparison table (ARI vs processing time) saved as a CSV file, analogous to Table S2

## How to apply

Load the TCGA pan-cancer expression matrix and the RCC R package, then invoke RCC_clus(x, config_file) with a first configuration using 8 parallel processes, variant pItem/pFeature subsampling fractions, and 100 repeats, recording cluster assignments and wall-clock processing time. Invoke RCC_clus a second time with a configuration using a single process, default pItem/pFeature, and 800 repeats, again recording cluster assignments and processing time. Compute the adjusted Rand index (ARI) of each run's clustering against the reference partition (e.g. tissue/cancer-type labels) to quantify agreement. Assemble ARI and processing-time values for both settings into a single comparison table and save it as a CSV (as in Table S2). The rationale is that RCC's computational time is a known limitation tied to its base algorithm, so parallelization with fewer repeats per process can dominate a serial run on both speed and ARI.

## Related tools

- **RCC** (R package whose RCC_clus(x, config_file) call executes both the parallel and serial clustering configurations being benchmarked) — github.com/MSCTR/RecursiveConsensusClustering
- **R** (Environment in which the benchmark runs and the ARI/time comparison table is assembled and saved as CSV)
- **ConsensusClusterPlus** (Base algorithm (CCP) whose process count and repeat settings are the benchmarked configuration parameters)
- **mclust** (Clustering method referenced as an alternative base method; RCC's runtime depends on the underlying base algorithm)

## Examples

```
RCC_clus(x, config_file)
```

## Evaluation signals

- The parallel configuration (8 processes, 100 repeats each) should produce higher ARI and lower wall-clock time than the single-process 800-repeat configuration, as reported in Table S2
- Wall-clock processing time must be recorded for both runs under identical hardware so the timing comparison is meaningful
- ARI values must be computed against the same reference partition for both configurations
- The comparison table must contain paired ARI and processing-time values for exactly the two tested configurations and be saved as CSV
- Both runs must be reproducible from explicit config files passed to RCC_clus(x, config_file)

## Limitations

- RCC's computational time is an acknowledged limitation of the algorithm and is dependent on the underlying base algorithm, currently consensus clustering with k-means
- Analysis of large integrated datasets with millions of cells was described as the next step, not yet demonstrated, so benchmark results may not extrapolate to such scales
- The published comparison is limited to two configurations (parallel variant-parameter vs serial default-parameter) on TCGA pan-cancer data; other parallelization factors or datasets were not benchmarked

## Evidence

- [supplementary] The parallel RCC configuration (eight processes, variant pItem/pFeature, 100 repeats each) showed higher ARI and was also faster than the single-process configuration with default parameters and 800 repetitions, as reported in Table S2.: [evidence span withheld — anchor status: not_found]
- [methods] RCC_clus(x, config_file): "RCC_clus(x, config_file)"
- [discussion] One of the limitations of our algorithm is computational time.: "One of the limitations of our algorithm is computational time."
- [discussion] This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means.: "This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means"
- [readme] devtools::install_github("MSCTR/RecursiveConsensusClustering"): "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
