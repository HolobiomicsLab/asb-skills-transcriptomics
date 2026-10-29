---
name: unreplicated-dataset-bootstrapping
description: Use when a single-cell dataset (e.g., an AnnData/h5ad object with clustered,
  annotated cell types) has no replicate samples for all conditions, and you need
  differential cell-type proportion analysis with p-values and significance calls.
when_to_use_negative:
- When the dataset has biological replicates for all conditions — use the replicated
  mode (pass samples_col) instead, since bootstrapping is meant for unreplicated data
- When the cell type of interest is rare (<2% of all cells) — benchmarking found such
  small clusters produce unstable results and bootstrapping has limited benefits for
  rare cell types
- When you need definitive conclusions about biological variability — the lack of
  replicates per se is not advisable, and bootstrapped results require additional
  experimental validation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanpro
  role: Implements propeller's linear regression on cell proportions with ANOVA/t-test
    and iteration-based bootstrapping to simulate pseudo-replicates for unreplicated
    datasets
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python framework into which Scanpro integrates; used for prior clustering/cell-type
    annotation of the single-cell data
- name: AnnData
  role: Native input format for the scanpro() function (data matrices saved as AnnData/.h5ad
    objects)
- name: propeller
  role: Precursor method whose linear regression approach Scanpro implements; propeller
    requires replicates, which bootstrapping removes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:58114a450964936517169b18159a0c4453f7b5fe1b21c1b3e8d3b3a9b5875e42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  - propeller
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# unreplicated-dataset-bootstrapping

## Summary

Scanpro's bootstrapping mode extends the propeller-style linear regression on cell-type proportions with iteration-based simulation of pseudo-replicates, enabling proportion analysis on single-cell datasets that lack biological replicates. It yields significance calls that benchmarks show often match replicated analyses.

## When to use

Use when a single-cell dataset (e.g., an AnnData/h5ad object with clustered, annotated cell types) has no replicate samples for all conditions, and you need differential cell-type proportion analysis with p-values and significance calls.

## When NOT to use

- When the dataset has biological replicates for all conditions — use the replicated mode (pass samples_col) instead, since bootstrapping is meant for unreplicated data
- When the cell type of interest is rare (<2% of all cells) — benchmarking found such small clusters produce unstable results and bootstrapping has limited benefits for rare cell types
- When you need definitive conclusions about biological variability — the lack of replicates per se is not advisable, and bootstrapped results require additional experimental validation

## Inputs

- AnnData object (.h5ad) with clustered/annotated single-cell data and metadata columns for cluster and condition (e.g., clusters_col='cluster', conds_col='conds'); samples_col omitted or None
- Prior cell clustering/cell-type annotation into cell types

## Outputs

- out.results: per-cell-type results table with p-values and significance calls for proportional shifts
- Optional plots via out.plot() (e.g., boxplots)
- Exported CSV of combined per-tissue results with p-values and significance calls (e.g., scanpro_fetal_tissue_results.csv)

## How to apply

Run the scanpro function on the AnnData object WITHOUT providing samples_col (or set samples_col=None), which the README states causes Scanpro to assume the dataset is not replicated and to run its bootstrapping method; pass clusters_col (e.g., 'cluster') and conds_col (e.g., 'conds') to point at cell-type and condition metadata. Scanpro implements propeller's linear regression approach with ANOVA/t-test and internally simulates pseudo-replicates via iteration-based bootstrapping. Retrieve the results table via out.results, which contains p-values per cell type, and optionally visualize with out.plot(). Compare the bootstrapped significance calls against a replicated run (scanpro(..., samples_col='sample')) where possible, since benchmarks show the two often reach the same conclusion; interpret bootstrapped-only results as indicative and prefer larger, less specific clusters (the article's example uses tissue-specialized types like cardiomyocytes, intestinal epithelial cells, and skeletal muscle cells).

## Related tools

- **Scanpro** (Implements propeller's linear regression on cell proportions with ANOVA/t-test and iteration-based bootstrapping to simulate pseudo-replicates for unreplicated datasets) — https://github.com/loosolab/scanpro
- **Scanpy** (Python framework into which Scanpro integrates; used for prior clustering/cell-type annotation of the single-cell data)
- **AnnData** (Native input format for the scanpro() function (data matrices saved as AnnData/.h5ad objects))
- **propeller** (Precursor method whose linear regression approach Scanpro implements; propeller requires replicates, which bootstrapping removes)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, clusters_col='clusters', conds_col='condition'); out.results
```

## Evaluation signals

- out.results is populated with a p-value and significance call for every cluster/cell type in the AnnData object
- Bootstrapped significance calls agree with the replicated run (scanpro with samples_col set) when both are performed on the same data — e.g., in the fetal scATAC heart/intestine/skeletal-muscle subset, cardiomyocytes, intestinal epithelial cells, and skeletal muscle cells are significant while smooth muscle cells are not, in both modes
- Every cluster included in the analysis has sufficient support (>=2% of all cells) to avoid unstable results
- Expected pattern of tissue-specialized cell types detected as significant while shared/non-specialized types (e.g., smooth muscle cells) are non-significant
- Results are exported with both p-values and significance calls (e.g., written to scanpro_fetal_tissue_results.csv) for cross-mode comparison

## Limitations

- Bootstrapping has limited benefits when handling rare cell types; clusters supported by less than 2% of all cells produce unstable results
- The lack of replicates limits the ability to draw definitive conclusions about variability, and unreplicated analysis is per se not advisable
- Bootstrapping-based results should be interpreted with caution and require additional experimental validation, although they often match replicated conclusions
- Clustering parameters (e.g., Leiden resolution) critically affect the number and size of clusters, and clusters may reflect technical parameters rather than real cell types
- Technical variability (different sequencing machines, sampling timepoints) may introduce batch effects that impact clustering and proportion analysis

## Evidence

- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [discussion] From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [discussion] Although this approach allows for proportion analysis on non replicated datasets and often results in the same conclusion as replicated datasets (see benchmarks): "Although this approach allows for proportion analysis on non replicated datasets and often results in the same conclusion as replicated datasets (see benchmarks)"
- [methods] from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
