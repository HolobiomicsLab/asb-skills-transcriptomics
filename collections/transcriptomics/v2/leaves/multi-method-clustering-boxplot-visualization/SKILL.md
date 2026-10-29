---
name: multi-method-clustering-boxplot-visualization
description: Use when a task requires comparing clustering quality of scSpace vs baselines
  (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) on labeled
  simulated datasets via ARI boxplots.
when_to_use_negative:
- Input is real biological scRNA-seq/ST data without ground-truth cluster labels —
  ARI validation is only defined here against Splatter-simulated subcluster labels
- The goal is spatial domain identification of ST data itself (e.g., SpaGCN/BayesSpace/DR-SC's
  native use case) rather than spatial analysis of scRNA-seq cells, which scSpace
  targets
- Tissues with low spatial coherence (e.g., SCC Patients 5 and 9) or when the spatial
  reference differs architecturally from the target tissue (e.g., agranular dACC lacking
  cortical layer 4) — the paper says to apply scSpace cautiously and assess reference
  appropriateness first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: scSpace
  role: 'Primary method under benchmark: TCA-based pseudo-space reconstruction and
    space-informed clustering evaluated by ARI'
  repo: https://github.com/ZJUFanLab/scSpace
- name: scanpy
  role: Runs Louvain/Leiden baseline community clustering via igraph and leidenalg
    on the simulated datasets
  repo: https://github.com/scverse/scanpy
- name: scikit-learn
  role: Provides K-means and agglomerative clustering baselines and the adjusted_rand_score
    function for ARI computation
- name: pandas
  role: Aggregates the 140-per-method ARI scores into a comparison table
- name: matplotlib
  role: Renders boxplots of ARI distributions per method across the simulation series
    (Fig. 1d-e)
- name: PyTorch
  role: Implements scSpace's MLP projection of single cells into the pseudo space
- name: SpaGCN
  role: Baseline spatial-domain identification method run on the same 140 simulations
- name: BayesSpace
  role: Baseline spatial-domain identification method run on the same 140 simulations
- name: DR-SC
  role: Baseline spatial-domain identification method run on the same 140 simulations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:997e54eef70c8b20790be78b98a2e781d75a415ba44ec48b34429771aa3a47b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - BayesSpace
  - DR-SC
  - PyTorch
  - SpaGCN
  - igraph
  - leidenalg
  - matplotlib
  - numpy
  - pandas
  - scSpace
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# Multi-method clustering boxplot visualization

## Summary

Benchmark scSpace against clustering and spatial-domain baselines by computing adjusted Rand index (ARI) against ground-truth labels across 140 Splatter-simulated paired scRNA-seq/ST datasets, aggregating scores with pandas, and rendering per-method ARI boxplots (Fig. 1d-e) with matplotlib.

## When to use

Use when a task requires comparing clustering quality of scSpace vs baselines (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS) on labeled simulated datasets via ARI boxplots.

## When NOT to use

- Input is real biological scRNA-seq/ST data without ground-truth cluster labels — ARI validation is only defined here against Splatter-simulated subcluster labels
- The goal is spatial domain identification of ST data itself (e.g., SpaGCN/BayesSpace/DR-SC's native use case) rather than spatial analysis of scRNA-seq cells, which scSpace targets
- Tissues with low spatial coherence (e.g., SCC Patients 5 and 9) or when the spatial reference differs architecturally from the target tissue (e.g., agranular dACC lacking cortical layer 4) — the paper says to apply scSpace cautiously and assess reference appropriateness first

## Inputs

- 140 Splatter-simulated paired scRNA-seq/spatial transcriptomics datasets (5000 genes each) with ground-truth spatially heterogeneous subcluster labels (2-10 subclusters)
- Five formatted .csv input files per scSpace run (read in by pandas), per the repository quick-start
- Predicted cluster labels from each baseline method (Louvain/Leiden via scanpy, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS)

## Outputs

- Per-simulation, per-method ARI scores for all 140 simulations and 9 methods
- Pandas comparison table aggregating the 140-per-method ARI values
- Matplotlib boxplots of ARI distributions per method across the simulation series (reproducing Fig. 1d-e)

## How to apply

Generate 140 paired scRNA-seq/ST simulations with Splatter (5000 genes each, spatially heterogeneous subcluster counts from 2 to 10, retaining ground-truth subcluster labels per cell). Run scSpace on each simulation — extracting characteristic matrices of ST and scRNA-seq data via transfer component analysis (TCA), projecting cells into pseudo space with an MLP model (PyTorch), and space-informed clustering on the joint gene-expression/spatial-weight embedding. Run the same simulations through baselines: Louvain/Leiden via scanpy with igraph and leidenalg, K-means and agglomerative hierarchical clustering via scikit-learn, and the spatial-domain methods SpaGCN, STAGATE, BayesSpace, and DR-SC. Compute ARI between each method's predicted clusters and the ground-truth subclusters per simulation with scikit-learn, aggregate the 140-per-method scores into a pandas comparison table, and render boxplots of ARI distributions per method across the simulation series with matplotlib. Validate that all ARIs fall in the valid 0-1 range and that scSpace attains equal or higher ARI than baselines, matching the paper's finding.

## Related tools

- **scSpace** (Primary method under benchmark: TCA-based pseudo-space reconstruction and space-informed clustering evaluated by ARI) — https://github.com/ZJUFanLab/scSpace
- **scanpy** (Runs Louvain/Leiden baseline community clustering via igraph and leidenalg on the simulated datasets) — https://github.com/scverse/scanpy
- **scikit-learn** (Provides K-means and agglomerative clustering baselines and the adjusted_rand_score function for ARI computation)
- **pandas** (Aggregates the 140-per-method ARI scores into a comparison table)
- **matplotlib** (Renders boxplots of ARI distributions per method across the simulation series (Fig. 1d-e))
- **PyTorch** (Implements scSpace's MLP projection of single cells into the pseudo space)
- **SpaGCN** (Baseline spatial-domain identification method run on the same 140 simulations)
- **BayesSpace** (Baseline spatial-domain identification method run on the same 140 simulations)
- **DR-SC** (Baseline spatial-domain identification method run on the same 140 simulations)

## Examples

```
cd scSpace-master; pip install -r requirements.txt; python setup.py build; python setup.py install
```

## Evaluation signals

- All ARI values fall within the valid 0-1 range for all 140 simulations per method
- scSpace attains equal or higher ARI than every baseline (Louvain, K-means, Hclust, SpaGCN, STAGATE, BayesSpace, DR-SC, scCoGAPS), consistent with the paper's finding
- scSpace maintains the highest ARI consistently as the simulated subcluster count increases from 2 to 10
- Each method yields exactly 140 ARI scores (one per simulation) before aggregation into the pandas table
- Boxplot x-axis covers all benchmarked methods and y-axis spans the ARI distributions, reproducing Fig. 1d-e layout

## Limitations

- Benchmark is limited to Splatter-simulated datasets; performance on real biological data without ground-truth labels cannot be validated by ARI in this workflow
- scSpace should be applied cautiously to tissues with low spatial coherence, and the intrinsic difference in spatial homogeneity between tissues may influence accuracy
- The choice of spatial reference directly influences the spatial reconstruction, so reference appropriateness should be assessed before comparing methods

## Evidence

- [methods] ARI computation over 140 simulations: [evidence span withheld — anchor status: not_found]
- [results] scSpace highest ARI finding: [evidence span withheld — anchor status: not_found]
- [readme] Paper figure code location: "Code for generating the figures in this study can be found [here](AnalysisPaper)."
- [readme] scSpace TCA characteristic matrix extraction: "scSpace could extract the characteristic matrixes of spatial transcriptomics and scRNA-seq, and project single cells into a pseudo space via a [multiple layer perceptron (MLP)]"
