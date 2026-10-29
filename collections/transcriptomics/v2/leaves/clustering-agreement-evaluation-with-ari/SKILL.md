---
name: clustering-agreement-evaluation-with-ari
description: Use when two or more clustering partitions (from RCC, ConsensusClusterPlus,
  or other methods like k-means, hierarchical clustering, pam, mclust) must be quantitatively
  compared against a reference to decide which parameterization (e.g., repeats, parallel
  processes, pItem/pFeature settings) agrees .
when_to_use_negative:
- You need to discover novel subtypes or determine the optimal number of clusters
  (k) in an unlabeled dataset — use RCC itself rather than ARI-vs-reference scoring
- There is no reference partition available; ARI requires a second partition and cannot
  assess absolute clustering quality on unlabeled data
- You are evaluating single-cell integration or batch correction (e.g., scanorama-based
  workflows) rather than comparing cluster label assignments
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: Generates the candidate cluster assignments being scored; RCC_clus(x, config_file)
    is run under different configurations (parallel vs single-process, repeat counts,
    pItem/pFeature settings)
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: mclust
  role: Source of the adjusted Rand index implementation used to score cluster agreement
    against a reference partition
- name: ConsensusClusterPlus
  role: Base algorithm underlying RCC's clustering divisions (CCP), whose configurations
    (process count, repeats) produce the partitions compared by ARI
- name: R
  role: Language in which RCC_clus is invoked, ARI is computed, and the comparison
    CSV table is assembled
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:56ad6cf3bc5cc0d18cee6c8fd0ec1870c3c8b92f247160730f3c851024f3f29b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - mclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Clustering agreement evaluation with ARI

## Summary

Quantify agreement between a clustering result (e.g., RCC/ConsensusClusterPlus-derived partitions from TCGA pan-cancer transcriptome data) and a reference partition using the adjusted Rand index (ARI), enabling like-for-like comparison of alternative clustering configurations such as parallel vs. single-process RCC runs.

## When to use

Use when two or more clustering partitions (from RCC, ConsensusClusterPlus, or other methods like k-means, hierarchical clustering, pam, mclust) must be quantitatively compared against a reference to decide which parameterization (e.g., repeats, parallel processes, pItem/pFeature settings) agrees best.

## When NOT to use

- You need to discover novel subtypes or determine the optimal number of clusters (k) in an unlabeled dataset — use RCC itself rather than ARI-vs-reference scoring
- There is no reference partition available; ARI requires a second partition and cannot assess absolute clustering quality on unlabeled data
- You are evaluating single-cell integration or batch correction (e.g., scanorama-based workflows) rather than comparing cluster label assignments

## Inputs

- Cluster assignments (partition labels per sample) from a candidate clustering run, e.g. RCC_clus output on the TCGA pan-cancer expression matrix
- Reference partition (known cluster/subtype labels) for the same samples
- Optional: recorded wall-clock processing times per run for joint ARI-vs-runtime comparison

## Outputs

- ARI value per clustering run quantifying agreement with the reference partition
- Comparison table (CSV) of ARI and processing-time values across configurations (e.g., parallel 8-process/100-repeat vs single-process/800-repeat RCC runs)

## How to apply

Run each clustering configuration to completion, e.g. by invoking RCC_clus(x, config_file) in R once with a parallel configuration (8 CCP processes, variant pItem/pFeature, 100 repeats) and once with a single-process, default-parameter, 800-repeat configuration, recording cluster assignments and wall-clock time for each. Then compute the adjusted Rand index (ARI, available via the mclust R package) of each run's cluster assignments against a reference partition to quantify label agreement independently of cluster naming. Assemble ARI and processing-time values for all configurations into a single comparison table (e.g., Table S2-style CSV) so that accuracy (ARI) and runtime trade-offs can be judged together. Use higher ARI as the correctness signal for clustering agreement and compare wall-clock time only after ARI confirms both runs recover the reference structure. This matters because RCC's authors note that computational time is a limitation tied to the underlying base algorithm (consensus clustering with k-means), so runtime-vs-agreement comparisons are central to configuring RCC.

## Related tools

- **RCC** (Generates the candidate cluster assignments being scored; RCC_clus(x, config_file) is run under different configurations (parallel vs single-process, repeat counts, pItem/pFeature settings)) — https://github.com/MSCTR/RecursiveConsensusClustering
- **mclust** (Source of the adjusted Rand index implementation used to score cluster agreement against a reference partition)
- **ConsensusClusterPlus** (Base algorithm underlying RCC's clustering divisions (CCP), whose configurations (process count, repeats) produce the partitions compared by ARI)
- **R** (Language in which RCC_clus is invoked, ARI is computed, and the comparison CSV table is assembled)

## Examples

```
ari <- mclust::adjustedRandIndex(rcc_parallel_clusters, reference_partition); ari2 <- mclust::adjustedRandIndex(rcc_single_clusters, reference_partition)
```

## Evaluation signals

- ARI is computed for every clustering configuration and is bounded such that identical partitions yield ARI = 1; ARI for each run against the reference should be reported, not just eyeballed cluster plots
- The comparison table contains exactly the paired metrics (ARI and processing time) for every configuration compared, e.g. 8-process/100-repeat vs single-process/800-repeat, and is saved as a CSV
- The winning configuration is justified by both higher ARI against the reference and measured runtime, matching the reported result that the parallel RCC configuration had higher ARI and was faster than the single-process default run
- Cluster assignments from both runs cover the same sample set as the reference partition (row/label alignment verified before ARI computation)
- Wall-clock times are recorded per run so runtime claims are reproducible rather than asserted

## Limitations

- Computational time is a stated limitation of RCC: 'One of the limitations of our algorithm is computational time', and it depends on the underlying base algorithm (consensus clustering with k-means), so runtime comparisons may not generalize to other base methods
- ARI quantifies agreement with a reference but cannot by itself validate biological relevance of novel subtypes not present in the reference
- Reducing repeats (e.g., 100 vs 800) may change stability of consensus partitions; ARI and runtime must be evaluated jointly per configuration rather than assuming parallelism is always safe

## Evidence

- [methods] RCC_clus(x, config_file): "RCC_clus(x, config_file)"
- [intro] RCC uses ConsensusClusterPlus as the base algorithm for dividing the data: "RCC uses ConsensusClusterPlus7 (CCP) as the base algorithm for dividing the data."
- [discussion] Computational time limitation depends on the base algorithm: "One of the limitations of our algorithm is computational time."
- [discussion] Time dependent on underlying base algorithm (consensus clustering with k-means): "This time is dependent on the underlying base algorithm which currently is consensus clustering with k-means."
- [intro] mclust listed among popular clustering methods: "popular clustering methods like k-means, hierarchical clustering, pam, mclust3, etc."
- [readme] README installation commands for the RCC package environment: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
