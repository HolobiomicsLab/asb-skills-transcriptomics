---
name: single-cell-rna-seq-data-simulation
description: 'Use when benchmarking spatial subpopulation or spatial-domain methods
  against a known truth: simulated paired scRNA-seq/ST data with ground-truth subcluster
  labels are needed for ARI-style evaluation.'
when_to_use_negative:
- The analysis goal is biological discovery on real tissue data rather than controlled
  benchmarking with known ground truth
- The method under test cannot consume paired single-cell and spatial transcriptomics
  inputs with fixed ground-truth subcluster assignments
- Evaluation requires metrics other than agreement with known subclusters (e.g. trajectory
  inference quality), for which ARI against simulated subclusters is not informative
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: Generates the paired simulated scRNA-seq/spatial transcriptomics datasets
    with controlled subcluster counts and retained ground-truth labels
- name: scSpace
  role: 'Candidate method under benchmark: TCA feature extraction, MLP pseudo-space
    projection, and space-informed clustering on simulated data'
  repo: github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Runs Louvain/Leiden community clustering baselines on the simulated datasets
  repo: github.com/scverse/scanpy
- name: scikit-learn
  role: K-means and hierarchical clustering baselines; computes adjusted Rand index
    against ground-truth subclusters
- name: pandas
  role: Aggregates the per-simulation ARI scores into a comparison table
- name: matplotlib
  role: Renders boxplots of ARI distributions per method across the simulation series
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:cb4be383c5031f3c2febc06fba23949badad49a8245c92bec0eff73949c75791
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BayesSpace
  - DR-SC
  - SpaGCN
  - Splatter
  - igraph
  - leidenalg
  - matplotlib
  - numpy
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# single-cell-rna-seq-data-simulation

## Summary

Generating synthetic paired scRNA-seq/spatial transcriptomics datasets with Splatter, with known ground-truth spatially heterogeneous subcluster labels, to benchmark clustering and spatial-domain identification methods such as scSpace.

## When to use

Use when benchmarking spatial subpopulation or spatial-domain methods against a known truth: simulated paired scRNA-seq/ST data with ground-truth subcluster labels are needed for ARI-style evaluation.

## When NOT to use

- The analysis goal is biological discovery on real tissue data rather than controlled benchmarking with known ground truth
- The method under test cannot consume paired single-cell and spatial transcriptomics inputs with fixed ground-truth subcluster assignments
- Evaluation requires metrics other than agreement with known subclusters (e.g. trajectory inference quality), for which ARI against simulated subclusters is not informative

## Inputs

- Simulation design parameters (number of paired datasets, e.g. 140; gene count, e.g. 5000; subcluster count range, e.g. 2-10)
- Splatter (R/Bioconductor) simulation model for paired scRNA-seq/spatial data
- Ground-truth subcluster labels per simulated cell

## Outputs

- 140 paired simulated scRNA-seq/ST datasets with varying subcluster counts (2-10)
- Per-cell ground-truth subcluster label files
- Per-simulation ARI values for each method (scSpace and baselines)
- Aggregated ARI comparison table and boxplots of ARI distributions per method

## How to apply

Generate simulated paired single-cell and spatial transcriptomics data with Splatter, controlling the number of spatially heterogeneous subclusters (e.g. varying from 2 to 10) and gene count (e.g. 5000 genes per simulation), while retaining ground-truth subcluster labels for every simulated cell. In the scSpace paper this was done at scale, producing 140 paired simulation datasets, so that a method's performance could be assessed as subcluster complexity increases. Run the candidate method (e.g. scSpace via TCA characteristic matrices, MLP pseudo-space projection, and space-informed clustering) and each baseline (Louvain/Leiden via scanpy with igraph and leidenalg, K-means, hierarchical clustering, SpaGCN, STAGATE, BayesSpace, DR-SC) on the identical simulations. Compute the adjusted Rand index (ARI) between predicted clusters and ground-truth subclusters per simulation, aggregate per-method scores, and compare distributions (e.g. boxplots) to determine whether the method consistently attains equal or higher ARI across the simulation series.

## Related tools

- **Splatter** (Generates the paired simulated scRNA-seq/spatial transcriptomics datasets with controlled subcluster counts and retained ground-truth labels)
- **scSpace** (Candidate method under benchmark: TCA feature extraction, MLP pseudo-space projection, and space-informed clustering on simulated data) — github.com/ZJUFanLab/scSpace
- **scanpy** (Runs Louvain/Leiden community clustering baselines on the simulated datasets) — github.com/scverse/scanpy
- **scikit-learn** (K-means and hierarchical clustering baselines; computes adjusted Rand index against ground-truth subclusters)
- **pandas** (Aggregates the per-simulation ARI scores into a comparison table)
- **matplotlib** (Renders boxplots of ARI distributions per method across the simulation series)

## Examples

```
conda create -n scspace python=3.8; conda activate scspace; python setup.py build; python setup.py install
```

## Evaluation signals

- ARI values for all 140 simulations per method fall within the valid 0-1 range
- scSpace attains equal or higher ARI than every baseline (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) as subcluster count increases from 2 to 10
- Each simulated dataset retains ground-truth subcluster labels for every simulated cell, enabling direct label-matching validation
- Per-method ARI distributions can be aggregated into a comparison table without missing simulations (140 per method)

## Limitations

- Simulated data may not capture real spatial-coherence variation: the paper notes that intrinsic differences in spatial homogeneity between tissues may influence accuracy, so simulation-based benchmarks do not fully represent low-coherence tissues
- Performance on simulations does not guarantee real-data performance; the paper additionally benchmarks on biological datasets (embryonic heart, MTG, melanoma, COVID-19 lung)
- Results depend on the choice of baselines and simulation parameters (gene count, subcluster range); SpaGCN, BayesSpace and DR-SC target spatial-domain identification of ST data rather than scRNA-seq spatial analysis, a scope difference noted by the authors

## Evidence

- [methods] Splatter simulation scale and design: [evidence span withheld — anchor status: not_found]
- [results] Benchmark finding: [evidence span withheld — anchor status: not_found]
- [abstract] Paper's stated benchmark conclusion: "We benchmark scSpace with both simulated and biological datasets, and demonstrate that scSpace can accurately and robustly identify spatially variated cell subpopulations."
- [readme] scSpace pipeline executed on simulations: [evidence span withheld — anchor status: not_found]
- [discussion] Scope distinction vs baselines: "scSpace focuses on the spatial analysis of scRNA-seq data whereas other methods target the spatial domain identification of ST data"
