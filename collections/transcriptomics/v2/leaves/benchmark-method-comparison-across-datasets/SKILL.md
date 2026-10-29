---
name: benchmark-method-comparison-across-datasets
description: Use when a new analysis tool claims parity with existing tools and the
  claim must be validated by running both on identical inputs (e.g. .h5ad AnnData
  with sample/cluster/condition columns) and comparing per-feature significance calls.
when_to_use_negative:
- Clusters supported by less than 2% of all cells in an experiment — the article's
  benchmarking found these produce unstable results, so their calls are not a fair
  basis for method comparison
- Datasets lacking the metadata needed to define replicates and conditions (sample/cluster/condition
  columns), since neither the replicated nor the pseudo-replicate comparison design
  can be set up
- When the goal is the analysis itself rather than validating tool agreement — for
  a single dataset with replicates, running one tool (e.g. Scanpro via scanpro())
  suffices without benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: Scanpro
  role: Method under test; run on replicated datasets via the linear-regression (logit)
    approach and on unreplicated data via iteration-based bootstrapping (sim_scanpro)
    with the arcsin transform; installed with 'pip install scanpro'
  repo: https://github.com/loosolab/scanpro
- name: propeller
  role: State-of-the-art comparator whose linear regression PA approach Scanpro implements;
    run on the same replicated and pseudo-replicated matrices for per-cluster significance
    comparison
- name: scCODA
  role: State-of-the-art comparator run on the same replicated and pseudo-replicated
    matrices; its calls (e.g. neurons in unreplicated heart data) anchor the benchmark
    validation
- name: Scanpy
  role: Used to load the benchmark AnnData objects (e.g. PBMC, heart development,
    COVID-19 BALF) with sample/cluster/condition metadata before proportion analysis
- name: AnnData
  role: Native input format Scanpro handles for each benchmark dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:434fbdf7dced6e9158605a7d6c8600b906656aea86477a28db8602ff8a114535
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
  - scCODA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# benchmark-method-comparison-across-datasets

## Summary

Benchmark a new computational method against state-of-the-art alternatives by running all tools on the same replicated and pseudo-replicated single-cell datasets (e.g. Scanpro vs propeller and scCODA on PBMC, heart-development, and COVID-19 BALF AnnData objects) and compiling per-cluster p-values and significance calls into one comparable table.

## When to use

Use when a new analysis tool claims parity with existing tools and the claim must be validated by running both on identical inputs (e.g. .h5ad AnnData with sample/cluster/condition columns) and comparing per-feature significance calls.

## When NOT to use

- Clusters supported by less than 2% of all cells in an experiment — the article's benchmarking found these produce unstable results, so their calls are not a fair basis for method comparison
- Datasets lacking the metadata needed to define replicates and conditions (sample/cluster/condition columns), since neither the replicated nor the pseudo-replicate comparison design can be set up
- When the goal is the analysis itself rather than validating tool agreement — for a single dataset with replicates, running one tool (e.g. Scanpro via scanpro()) suffices without benchmarking

## Inputs

- AnnData (.h5ad) single-cell datasets with per-cell metadata columns for sample, cluster, and condition (e.g. scanpro_tutorial.h5ad loaded with ad.read_h5ad)
- Pseudo-replicated count matrices built by merging biological replicates
- Reference significance calls from the published benchmark (Fig. 1d,e)

## Outputs

- Compiled table of per-cluster p-values and significance calls for every method/dataset/condition combination (Scanpro, propeller, scCODA; replicated and unreplicated)
- Per-method result objects accessible via out.results, plottable with out.plot()

## How to apply

Run the new tool (Scanpro) and the comparators (propeller, scCODA) on the same replicated benchmark datasets — e.g. PBMC male-vs-female, heart development, and COVID-19 BALF loaded via Scanpy as AnnData with 'sample', 'cluster', and 'conds' metadata columns — using the linear-regression (logit) approach on cell proportions. Then merge biological replicates into pseudo-replicates and rerun the unreplicated matrices with Scanpro's iteration-based bootstrapping (sim_scanpro) using the arcsin transform, and run propeller and scCODA on the same pseudo-replicated matrices. Retrieve per-cluster results for every method via out.results and compile a single table of p-values and significance calls. Judge correctness by reproducing the published calls (Fig. 1d,e): NK cells significant in PBMC with logit, NK retained with arcsin on unreplicated data, mDC p=0.049 for Scanpro vs p=0.054 for propeller, and scCODA calling neurons in unreplicated heart data. Exclude or flag clusters supported by less than 2% of all cells, since the article's benchmarking found such clusters produce unstable results.

## Related tools

- **Scanpro** (Method under test; run on replicated datasets via the linear-regression (logit) approach and on unreplicated data via iteration-based bootstrapping (sim_scanpro) with the arcsin transform; installed with 'pip install scanpro') — https://github.com/loosolab/scanpro
- **propeller** (State-of-the-art comparator whose linear regression PA approach Scanpro implements; run on the same replicated and pseudo-replicated matrices for per-cluster significance comparison)
- **scCODA** (State-of-the-art comparator run on the same replicated and pseudo-replicated matrices; its calls (e.g. neurons in unreplicated heart data) anchor the benchmark validation)
- **Scanpy** (Used to load the benchmark AnnData objects (e.g. PBMC, heart development, COVID-19 BALF) with sample/cluster/condition metadata before proportion analysis)
- **AnnData** (Native input format Scanpro handles for each benchmark dataset)

## Examples

```
from scanpro import scanpro
out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- Compiled table reproduces the reported Fig. 1d,e calls: NK cells significant in PBMC with logit and NK retained with arcsin on unreplicated data
- mDC significance values match the published ordering: p=0.049 for Scanpro vs p=0.054 for propeller
- scCODA call on unreplicated heart data (neurons) is recovered in the comparison table
- Setting samples_col to None triggers the bootstrapping path rather than the replicated path (per README behaviour), confirming the pseudo-replicate run is actually unreplicated
- Conclusions from bootstrapped pseudo-replicates agree with replicated runs for non-rare cell types, as reported in the article's benchmarks

## Limitations

- Bootstrapping has limited benefits when handling rare cell types; clusters supported by less than 2% of all cells produce unstable results
- Bootstrapping-based results should be seen as an indication of cell types with a proportional shift and interpreted with caution plus additional experimental validation; the article notes that lack of replicates limits drawing definitive conclusions about variability and is per se not advisable
- Clustering parameters (e.g. Leiden resolution) critically affect cluster number and size; the article recommends larger, less specific clusters with higher cell numbers, and warns clusters may be defined by technical parameters or batch effects rather than real cell types

## Evidence

- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [discussion] we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [abstract] Scanpro is fast, accurate, supports datasets without replicates: "Scanpro is fast, accurate, supports datasets without replicates"
