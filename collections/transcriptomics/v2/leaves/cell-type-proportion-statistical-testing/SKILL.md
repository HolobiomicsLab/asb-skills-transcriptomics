---
name: cell-type-proportion-statistical-testing
description: Use when single-cell resolution data (AnnData with sample, cluster, condition
  metadata columns) needs differential compositional analysis of cell-type proportions,
  with or without biological replicates.
when_to_use_negative:
- Clusters supported by less than 2% of all cells in the experiment — benchmarking
  showed such rare clusters produce unstable results and bootstrapping has limited
  benefits for rare cell types
- Raw cell counts must be compared without statistical modeling of technical (sequencing
  depth, cell extraction bias) and biological (age) effects — proportions require
  compositional analysis, not direct count comparison
- 'Clusters not yet defined: clustering parameters (e.g., Leiden resolution) critically
  affect cluster number and size, so PA should not be run before correct cell clustering
  and cell-type identification'
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: Scanpro
  role: 'Main tool: runs proportion analysis via linear regression/ANOVA and bootstrapping
    on AnnData objects; provides scanpro, run_stats, anova, t_test, sim_scanpro'
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python single-cell framework Scanpro integrates into; used for prior clustering/annotation
    of cells into cell types
- name: AnnData
  role: Native input format Scanpro handles for single-cell data with sample/cluster/condition
    metadata
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:de3e4573c95425f9f58d062dc0aa831af99a09caef30bbe2a2087d8a2a801541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  - run_stats
  - sim_scanpro
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cell-type-proportion-statistical-testing

## Summary

Statistical testing of cell-type proportion shifts across conditions using Scanpro, which runs linear regression/ANOVA (propeller-style) on cell proportions from an AnnData object and extends it with bootstrapping to simulate pseudo-replicates for unreplicated data.

## When to use

Use when single-cell resolution data (AnnData with sample, cluster, condition metadata columns) needs differential compositional analysis of cell-type proportions, with or without biological replicates.

## When NOT to use

- Clusters supported by less than 2% of all cells in the experiment — benchmarking showed such rare clusters produce unstable results and bootstrapping has limited benefits for rare cell types
- Raw cell counts must be compared without statistical modeling of technical (sequencing depth, cell extraction bias) and biological (age) effects — proportions require compositional analysis, not direct count comparison
- Clusters not yet defined: clustering parameters (e.g., Leiden resolution) critically affect cluster number and size, so PA should not be run before correct cell clustering and cell-type identification

## Inputs

- AnnData object (.h5ad) with clustering done and metadata columns: sample (or None for unreplicated), cluster, conds
- Optional design metadata for covariates (e.g., sex, age columns)
- Optional simulated proportion-change datasets generated with sim_scanpro

## Outputs

- out.results table with per-cell-type statistics (p-values, significance calls)
- Per-cluster p-value boxplots via out.plot(kinde='boxplot')
- Consolidated CSV tables (e.g., scanpro_covariate_results.csv, simulation_accuracy.csv)

## How to apply

Load the single-cell dataset as an AnnData object (e.g., scanpro_tutorial.h5ad) after prior clustering and cell-type annotation, then call scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds') to perform linear regression on cell proportions with ANOVA (run_stats with anova and t_test backends). If samples_col is not given or set to None, the dataset is treated as unreplicated and the iteration-based bootstrapping method simulates pseudo-replicates instead. For unreplicated data, tune the pseudo-replicate count: benchmarks on simulated datasets (20,000 cells per condition, three of seven clusters changed) show accuracy peaks at 5-8 pseudo-replicates and decreases with more or fewer; arcsin slightly outperforms the logit transformation at 5-14 pseudo-replicates. Covariates such as sex or age can be added to the linear-model-based ANOVA testing to check whether they drive proportion effects (e.g., cardiomyocyte and myeloid changes in the MI dataset were partially sex-driven; NK cells became significant for age in PBMC). Retrieve per-cell-type statistics from out.results and visualize per-cluster p-value distributions with out.plot(kinde='boxplot').

## Related tools

- **Scanpro** (Main tool: runs proportion analysis via linear regression/ANOVA and bootstrapping on AnnData objects; provides scanpro, run_stats, anova, t_test, sim_scanpro) — https://github.com/loosolab/scanpro
- **Scanpy** (Python single-cell framework Scanpro integrates into; used for prior clustering/annotation of cells into cell types)
- **AnnData** (Native input format Scanpro handles for single-cell data with sample/cluster/condition metadata)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'); out.results
```

## Evaluation signals

- out.results is populated with per-cluster statistics for every cell type in clusters_col
- Replicated-setting accuracy on simulated proportion-change data (three of seven clusters changed, 100 simulated datasets per replicate count 2-14) falls within the reported 0.6-0.97 range
- Bootstrapping accuracy on unreplicated data shows the 5-8 pseudo-replicate sweet spot, with arcsin slightly outperforming logit at 5-14 pseudo-replicates
- Bootstrapped (pseudo-replicate) significance calls agree with the replicated-analysis conclusions (e.g., cardiomyocytes, intestinal epithelial cells, and skeletal muscle cells significant; smooth muscle cells not)
- Clusters with <2% of total cells show unstable p-value distributions in the per-cluster boxplots

## Limitations

- Clusters supported by less than 2% of all cells produce unstable results; bootstrapping has limited benefits for rare cell types (e.g., the Changed3 cluster <2% of cells)
- Bootstrapping-based results should be seen only as an indication of proportional shifts, interpreted with caution, and backed by additional experimental validation
- The lack of replicates limits drawing definitive conclusions about variability and is per se not advisable
- Clustering parameters such as Leiden resolution critically affect cluster number and size; clusters might reflect technical parameters rather than real cell types, challenging PA interpretation

## Evidence

- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [abstract] Scanpro is fast, accurate, supports datasets without replicates, and is intended to be used by bioinformatics experts and beginners alike.: "Scanpro is fast, accurate, supports datasets without replicates, and is intended to be used by bioinformatics experts and beginners alike."
- [discussion] we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality"
- [methods] out.plot(kinde='boxplot'): "out.plot(kinde='boxplot')"
