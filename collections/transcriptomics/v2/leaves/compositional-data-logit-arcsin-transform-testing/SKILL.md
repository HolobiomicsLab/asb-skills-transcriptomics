---
name: compositional-data-logit-arcsin-transform-testing
description: Use when cell counts per cluster/condition are available in an AnnData
  object and you must test whether cell-type proportions shift between conditions
  — including unreplicated data where propeller-style tools fail.
when_to_use_negative:
- Clusters supported by less than 2% of all cells in the experiment — results are
  unstable for such rare cell types, and bootstrapping has limited benefits there
- Raw cell counts without prior cell clustering/annotation into cell types — proportion
  analysis requires clustering of cells into cell types first
- Drawing definitive conclusions from unreplicated data — lack of replicates limits
  conclusions about variability and is per se not advisable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Scanpro
  role: Implements the linear regression (logit) proportion-analysis approach and
    extends it with arcsin-transformed iteration-based bootstrapping (sim_scanpro)
    for pseudo-replicates on unreplicated data
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python framework used to load and preprocess the benchmark AnnData objects
    before proportion analysis
- name: propeller
  role: State-of-the-art R tool whose linear regression approach Scanpro implements;
    used for benchmark comparison on replicated and pseudo-replicated matrices
- name: scCODA
  role: Bayesian compositional-analysis benchmark comparator producing per-cluster
    significance calls on the same datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:7058433e0845f82a7b6f745ec78d472f3f80540d1a33b722ea9291fb2fe06403
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - Python
  - Scanpro
  - Scanpy
  - propeller
  - scCODA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# compositional-data-logit-arcsin-transform-testing

## Summary

Statistical testing of cell-type proportion shifts in single-cell data using Scanpro's linear-regression (logit) approach on replicated AnnData objects, extended with arcsin-transformed bootstrapping (sim_scanpro) to simulate pseudo-replicates when no biological replicates exist.

## When to use

Use when cell counts per cluster/condition are available in an AnnData object and you must test whether cell-type proportions shift between conditions — including unreplicated data where propeller-style tools fail.

## When NOT to use

- Clusters supported by less than 2% of all cells in the experiment — results are unstable for such rare cell types, and bootstrapping has limited benefits there
- Raw cell counts without prior cell clustering/annotation into cell types — proportion analysis requires clustering of cells into cell types first
- Drawing definitive conclusions from unreplicated data — lack of replicates limits conclusions about variability and is per se not advisable

## Inputs

- AnnData object (.h5ad, e.g. scanpro_tutorial.h5ad) with clustering done and metadata columns for sample, cluster, and condition
- Cell-type count/proportion table derived from single-cell data (replicated or unreplicated)

## Outputs

- Per-cluster results table with p-values and significance calls (out.results)
- Result plots of proportion shifts per cell type (out.plot())

## How to apply

After clustering cells into cell types (cell clustering must precede proportion analysis), run Scanpro on the AnnData object with sample, cluster, and condition metadata columns via scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'), which applies the linear regression (logit) approach to cell proportions and ANOVA/t-test statistics. For unreplicated datasets, omit samples_col (or set it to None) so Scanpro automatically triggers the iteration-based bootstrapping method (sim_scanpro) that internally simulates pseudo-replicates using the arcsin transform. Retrieve per-cluster p-values and significance calls via out.results and visualize with out.plot(). Avoid direct comparison of raw cell counts between clusters, since sequencing depth, cell-extraction bias, and other technical/biological effects confound them; prefer larger clusters over many tiny ones, because clusters supported by less than 2% of all cells produce unstable results.

## Related tools

- **Scanpro** (Implements the linear regression (logit) proportion-analysis approach and extends it with arcsin-transformed iteration-based bootstrapping (sim_scanpro) for pseudo-replicates on unreplicated data) — https://github.com/loosolab/scanpro
- **Scanpy** (Python framework used to load and preprocess the benchmark AnnData objects before proportion analysis)
- **propeller** (State-of-the-art R tool whose linear regression approach Scanpro implements; used for benchmark comparison on replicated and pseudo-replicated matrices)
- **scCODA** (Bayesian compositional-analysis benchmark comparator producing per-cluster significance calls on the same datasets)

## Examples

```
from scanpro.scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'); out.results
```

## Evaluation signals

- Reproduces Fig. 1d,e calls: NK cells significant in PBMC with logit, NK retained with arcsin on unreplicated data
- Per-cluster p-values match reported values, e.g. mDC p=0.049 for Scanpro vs p=0.054 for propeller on the COVID-19 BALF dataset
- scCODA calls neurons significant in the unreplicated heart development dataset, consistent with the pseudo-replicate run
- Compiled results table (out.results) covers every method/dataset/condition combination with comparable per-cluster significance calls
- Clusters with <2% of all cells flagged as unstable are excluded or interpreted with caution

## Limitations

- propeller requires replicates for all conditions investigated; replicated single-cell datasets are rare due to experimental cost
- Bootstrapping has limited benefits when handling rare cell types, and unreplicated results should be interpreted with caution and additional experimental validation
- Clustering parameters such as Leiden resolution critically affect cluster number and size, and clusters may reflect technical parameters (batch effects) rather than real cell types
- Lack of replicates limits the ability to draw definitive conclusions about variability and is per se not advisable

## Evidence

- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [discussion] Iteration-based bootstrapping to simulate pseudo-replicates for non-replicated single-cell datasets: "we address this issue by our iteration-based bootstrapping method"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [discussion] From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [intro] Due to technical (e.g. sequencing depth, bias in the cell extracting method and different cell decay rate) and biological (e.g. differences in age) effects, raw cell counts cannot be compared: "Due to technical (e.g. sequencing depth, bias in the cell extracting method and different cell decay rate) and biological (e.g. differences in age) effects, raw cell counts cannot be compared"
