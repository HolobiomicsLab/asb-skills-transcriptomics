---
name: adjusted-rand-index-computation
description: Use when benchmarking a single-cell clustering method (e.g. scDHA) against
  simulated ground-truth cell-type labels from sc_mosim/MOSim and you need a chance-corrected
  agreement metric over per-cell label vectors.
when_to_use_negative:
- The data are real, unannotated single-cell data with no ground-truth labels — ARI
  requires a reference labeling and MOSim/sc_mosim synthetic data exist precisely
  to supply this ground truth
- You are validating continuous expression profiles rather than discrete cluster/label
  assignments — the MOSim paper instead uses Spearman's distance, k-medoids clustering,
  and Pearson's/Kendall's Tb correlations for that setting
- You need to know which specific cell types were confused or merged — ARI gives a
  single agreement score, so first build an explicit cluster-to-label mapping to interpret
  the result
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: Supplies the simulated scRNA-seq data and ground-truth cell-type labels (via
    sc_mosim) against which the ARI is computed
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment in which the ARI between predicted clusters and
    ground-truth labels is computed and results are written to CSV
- name: scDHA
  role: VAE-based clustering method whose per-cell cluster assignments are compared
    to the simulated ground truth via ARI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:297c211ad053e80c1204b43fd84d38de7226fc22be019042bf0e7f6e215e9511
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - scDHA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# adjusted-rand-index-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the Adjusted Rand Index (ARI), a chance-corrected measure of agreement between predicted cluster assignments and ground-truth cell-type labels, to quantify how well a clustering method recovers simulated cell populations. Used here to validate scDHA clustering of scMOSim-simulated scRNA-seq data (ARI 0.949, 5 of 6 cell types).

## When to use

Use when benchmarking a single-cell clustering method (e.g. scDHA) against simulated ground-truth cell-type labels from sc_mosim/MOSim and you need a chance-corrected agreement metric over per-cell label vectors.

## When NOT to use

- The data are real, unannotated single-cell data with no ground-truth labels — ARI requires a reference labeling and MOSim/sc_mosim synthetic data exist precisely to supply this ground truth
- You are validating continuous expression profiles rather than discrete cluster/label assignments — the MOSim paper instead uses Spearman's distance, k-medoids clustering, and Pearson's/Kendall's Tb correlations for that setting
- You need to know which specific cell types were confused or merged — ARI gives a single agreement score, so first build an explicit cluster-to-label mapping to interpret the result

## Inputs

- Per-cell cluster assignment vector from the clustering method (e.g. scDHA clusters on log(x+1)-transformed, nonlinear-kernel-normalized scMOSim counts)
- Simulated ground-truth cell-type labels for the same cells (from sc_mosim simulation, as produced in task_003)

## Outputs

- Scalar Adjusted Rand Index value quantifying clustering vs. ground-truth agreement
- Per-cell cluster assignments plus ARI written to scdha_clustering_results.csv
- Cluster-to-cell-type mapping table identifying merged/missing populations (e.g. cDC + Treg in one cluster)

## How to apply

Load the per-cell cluster assignment vector produced by the clustering method (e.g. scDHA run on log(x+1)-transformed, kernel-normalized scMOSim-simulated counts) and the simulated ground-truth cell-type labels produced by sc_mosim. Compute the ARI in R between the two per-cell label vectors; ARI is permutation-invariant, so no cluster-to-label matching is required for the score itself, and it ranges from ~0 (chance agreement) to 1 (perfect agreement). Map predicted clusters onto ground-truth cell types separately to interpret the score, since ARI alone does not reveal which populations were merged or misassigned (e.g. in the scDHA benchmark, cDC and Treg merged into one cluster of five, yielding ARI ≈ 0.949 against six simulated types). Write the per-cell cluster assignments together with the ARI to scdha_clustering_results.csv so the score and its underlying assignments remain reproducible.

## Related tools

- **MOSim** (Supplies the simulated scRNA-seq data and ground-truth cell-type labels (via sc_mosim) against which the ARI is computed) — https://github.com/ConesaLab/MOSim
- **R** (Implementation environment in which the ARI between predicted clusters and ground-truth labels is computed and results are written to CSV)
- **scDHA** (VAE-based clustering method whose per-cell cluster assignments are compared to the simulated ground truth via ARI)

## Evaluation signals

- Reproduced ARI is approximately 0.949 for the scDHA-on-scMOSim benchmark with six simulated cell types
- Five clusters represent the six simulated cell types, with cDC and Treg merged into a single cluster
- ARI equals 1.0 on a sanity check where predicted labels equal ground-truth labels, and near 0.949 on the real run (values ≤ 0 indicate chance-level agreement)
- Both label vectors cover the same cells with no missing or NA assignments before computing ARI
- scdha_clustering_results.csv contains per-cell cluster assignments and the ARI metric as required

## Limitations

- ARI does not identify which cell types were merged or misassigned; the scDHA run required a separate mapping step to discover the cDC/Treg merge
- Cluster-granularity mismatch lowers ARI: setting the cluster count to six while two populations merge changes both cluster count and score
- Ground truth must come from simulation — MOSim's synthetic data are the source of true labels, and its single-cell simulation is restricted to scRNA-seq and scATAC-seq

## Evidence

- [abstract] generate realistic and customizable bulk and single-cell multi-omics datasets to benchmark and validate analytical methods: "generate realistic and customizable bulk and single-cell multi-omics datasets to benchmark and validate analytical methods"
- [intro] Synthetic data as ground truth for benchmarking: "Synthetic data, serving as ground truth, provide an indispensable resource for defining true positive and negative feature sets, enabling rigorous benchmarking"
- [readme] MOSim flexibility for clustering method testing: "MOSim has great flexibility in defining experimental designs, DEGs, and active regulators making it a versatile tool for a variety of different applications"
