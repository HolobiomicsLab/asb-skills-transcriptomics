---
name: hierarchical-cluster-structure-generation
description: Use when benchmarking transfer-learning clustering on scRNA-Seq data
  requires synthetic count matrices with known ground-truth labels and controlled
  complete/incomplete/no-overlap between source and target cell types.
when_to_use_negative:
- Input is already a real scRNA-Seq dataset with experimental labels — this skill
  only manufactures synthetic benchmarks, not reference annotations for real data
- The clustering method requires no ground truth and no controlled source–target overlap,
  so simulation adds no evaluation value
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scRNA (cmd_generate_data.py)
  role: Command-line script from the scRNA Python framework (simulation.py) that generates
    the artificial hierarchical count data, source/target splits, and ground-truth
    labels
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Downstream clustering algorithm applied to X_trg_new (TransferCluster), the
    target alone (TargetCluster), and concatenated source+target (ConcatenateCluster)
    on the simulated datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:9411722e15d5426b7bf2e8366597cd38e8f56691e0a459c85ff4fa03af9b4076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hierarchical-cluster-structure-generation

## Summary

Generates simulated count-level scRNA-Seq datasets with a known hierarchical cluster structure (8 clusters derived from 5 top nodes) and controlled source–target cell-type overlap, providing ground-truth labels for benchmarking transfer-learning-augmented clustering against TargetCluster and ConcatenateCluster baselines.

## When to use

Use when benchmarking transfer-learning clustering on scRNA-Seq data requires synthetic count matrices with known ground-truth labels and controlled complete/incomplete/no-overlap between source and target cell types.

## When NOT to use

- Input is already a real scRNA-Seq dataset with experimental labels — this skill only manufactures synthetic benchmarks, not reference annotations for real data
- The clustering method requires no ground truth and no controlled source–target overlap, so simulation adds no evaluation value

## Inputs

- Simulation configuration (number of clusters derived from top nodes V–Z, 10000 genes, 1800 cells, Dirichlet concentration 10, gamma shape 2 / rate 0.1, DE-gene fraction 10–40%, log2FC ~ N(1, 0.5), NB dispersion 0.1)
- Overlap scenario specification (complete-overlap, incomplete-overlap, no-overlap)
- Target dataset sizes for downsampling (10, 50, 100, 200, 400, 600, 800 cells)

## Outputs

- Source and target count matrices with gene ids and ground-truth cluster labels (as produced by cmd_generate_data.py / scRNA-generate-data.sh)
- 100 simulated datasets per overlap setting for statistical aggregation
- Aggregated ARI results table (simulation_ari_results.csv) with mean ARI and 95% confidence intervals per setting × target size × method

## How to apply

Use the scRNA package's data-generation script (scRNA/simulation.py, cmd_generate_data.py, exposed as scRNA-generate-data.sh after pip install) to generate 100 datasets per overlap setting: 8 clusters derived from 5 top nodes, 10,000 genes, 1800 cells, cluster sizes drawn from a Dirichlet distribution with concentration 10, gamma background expression (shape 2, rate 0.1), 10–40% differentially expressed genes per cluster with log2 fold changes ~ N(1, 0.5), and negative binomial count sampling with dispersion 0.1. Split each dataset into a 1000-cell source dataset and a target dataset downsampled to 10–800 cells, assigning top-node clusters to source/target according to the complete-overlap, incomplete-overlap, or no-overlap scenario. Preprocess source and target separately (cell filter, gene filter for ubiquitous and rare genes, log-transformation after adding pseudo-count 1), then run TransferCluster, TargetCluster (SC3 on target alone) and ConcatenateCluster (SC3 on concatenated data). Compute Adjusted Rand Index against the known ground-truth labels on target cells only, aggregate over 100 repetitions per setting × target size × method, and report 95% confidence intervals.

## Related tools

- **scRNA (cmd_generate_data.py)** (Command-line script from the scRNA Python framework (simulation.py) that generates the artificial hierarchical count data, source/target splits, and ground-truth labels) — https://github.com/nicococo/scRNA
- **SC3** (Downstream clustering algorithm applied to X_trg_new (TransferCluster), the target alone (TargetCluster), and concatenated source+target (ConcatenateCluster) on the simulated datasets)

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git; mkdir /foo && cd /foo && scRNA-generate-data.sh
```

## Evaluation signals

- Output files must include gene ids, source/target data, and source/target ground truth labels, matching the README's description of scRNA-generate-data.sh results
- Each simulated dataset must contain 8 clusters derived from 5 top nodes, 10,000 genes, and 1800 cells, replicated 100 times per overlap setting
- simulation_ari_results.csv must show TransferCluster achieving higher mean ARI than both TargetCluster and ConcatenateCluster for complete-overlap and incomplete-overlap settings across target sizes
- In the no-overlap setting, TransferCluster must not significantly degrade performance relative to TargetCluster (overlapping 95% confidence intervals), consistent with Figure 2B–D

## Limitations

- Simulated data derive from a specific generative model (gamma background, NB counts with dispersion 0.1, N(1, 0.5) log2 fold changes); conclusions may not transfer to real data with different noise structure
- If source and target only share a small set of transcripts, 'a loss of (probably) vital information is inevitable since only the set of genes present in both datasets can be used'
- No software version numbers are stated for SC3 or other tools, complicating exact reproduction

## Evidence

- [methods] Simulated data scale and replication: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [supplementary] Data generation script availability: "The command line script for simulating scRNA-Seq datasets is available at https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py"
- [readme] README description of package scope: "This package contains methods for generating artificial data, clustering, and transfering knowledge from a source to a target dataset."
- [methods] Simulation code location in repository: "the full code is provided in https://github.com/nicococo/scRNA/blob/master/scRNA/simulation.py"
