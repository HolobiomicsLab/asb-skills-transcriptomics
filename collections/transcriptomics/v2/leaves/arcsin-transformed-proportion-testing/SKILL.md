---
name: arcsin-transformed-proportion-testing
description: Use when a single-cell dataset annotated with cell type clusters (e.g.
  an AnnData object with cluster/condition/sample columns) must be tested for differential
  cell-type proportions, especially when replicates are missing or unbalanced across
  conditions.
when_to_use_negative:
- When the input is raw cell counts compared directly between clusters — raw counts
  cannot be compared due to sequencing depth and other technical/biological effects,
  so transformation and modelling are required instead
- When the biological question is within-cell-type gene expression change rather than
  cell-type proportion shift
- When rare cell types make up less than 2% of all cells and would drive unstable
  results — merge or interpret such clusters with caution
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2229
tools:
- name: Scanpro
  role: Implements arcsin-transformed linear regression (propeller-style), t-test/ANOVA
    testing, and bootstrapping pseudo-replicate simulation on AnnData input
  repo: https://github.com/loosolab/scanpro
- name: AnnData
  role: 'Native input container: data matrices plus sample/cluster/condition annotations
    consumed by the scanpro function'
- name: scCODA
  role: Bayesian compositional alternative used for cross-validation of significant
    detections on pairwise-replicate versions of the same atlas
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:8a5b41ae3a94a72aff8f82fd82edb3d1609ec4b5159e917caf050e3e12fecf60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - scCODA
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# arcsin-transformed-proportion-testing

## Summary

Statistical testing of cell-type proportion shifts by applying Scanpro's arcsin (sqrt-linear) transformation to cell type fractions followed by linear modelling (t-test for two conditions, ANOVA for multiple), optionally extended with iteration-based bootstrapping to simulate pseudo-replicates for unreplicated single-cell datasets.

## When to use

Use when a single-cell dataset annotated with cell type clusters (e.g. an AnnData object with cluster/condition/sample columns) must be tested for differential cell-type proportions, especially when replicates are missing or unbalanced across conditions.

## When NOT to use

- When the input is raw cell counts compared directly between clusters — raw counts cannot be compared due to sequencing depth and other technical/biological effects, so transformation and modelling are required instead
- When the biological question is within-cell-type gene expression change rather than cell-type proportion shift
- When rare cell types make up less than 2% of all cells and would drive unstable results — merge or interpret such clusters with caution

## Inputs

- AnnData (.h5ad) single-cell or single-nucleus object with cell-level sample, cluster, and condition annotations
- Column names identifying samples (samples_col), clusters (clusters_col), and conditions (conds_col)

## Outputs

- out.results: per-cell-type table of transformed-proportion statistics with p-values and FDR (exportable as CSV, e.g. scanpro_kuppe_results.csv)
- Plots of proportion changes via out.plot() (line plots or boxplots via the kind parameter)

## How to apply

Load the annotated dataset (e.g. Kuppe et al. human myocardial infarction snRNA-seq atlas) as an AnnData object carrying sample, cluster, and condition annotations, and call the scanpro function with samples_col='sample', clusters_col='cluster', conds_col='conds'. Scanpro implements propeller's linear regression approach on arcsin-transformed proportions: for two conditions a t-test is applied, for more conditions an ANOVA. If samples_col is omitted or None, the dataset is treated as unreplicated and Scanpro runs iteration-based bootstrapping (e.g. 8 pseudo-replicates for the ischemic-vs-control MI runs) to simulate replicates before testing. Access out.results to obtain per-cell-type statistics with FDR/p-values, and call out.plot() (optionally with the kind parameter set to boxplots) to visualize proportion shifts. Clusters supported by less than 2% of all cells produce unstable results, so prefer larger, less specific clusters over many small highly specific clusters.

## Related tools

- **Scanpro** (Implements arcsin-transformed linear regression (propeller-style), t-test/ANOVA testing, and bootstrapping pseudo-replicate simulation on AnnData input) — https://github.com/loosolab/scanpro
- **AnnData** (Native input container: data matrices plus sample/cluster/condition annotations consumed by the scanpro function)
- **scCODA** (Bayesian compositional alternative used for cross-validation of significant detections on pairwise-replicate versions of the same atlas)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'); out.results; out.plot(kind='boxplot')
```

## Evaluation signals

- On the unreplicated Kuppe et al. MI atlas, bootstrapping should call cardiomyocytes, cycling cells, and myeloid cells as significantly changed, matching the original publication's three reported cell types
- Significant detections under bootstrapping should be concordant with scCODA results run on the pairwise-replicate versions of the same atlas
- out.results must contain per-cluster statistics including p-value/FDR columns for every cell type
- Clusters representing less than 2% of all cells should be flagged or excluded as unstable in the benchmarking sense
- Aggregated result table (e.g. scanpro_kuppe_results.csv) must retain FDR/p-values for all tested clusters

## Limitations

- Lack of replicates limits drawing definitive conclusions about variability and is per se not advisable; bootstrapping results are an indication and should be interpreted with caution with additional experimental validation
- Bootstrapping has limited benefits for rare cell types
- Clusters under 2% of all cells produce unstable results
- Clustering parameters such as Leiden resolution critically affect cluster number and size, and clusters may reflect technical parameters rather than real cell types
- The bootstrapped approach often but not always reproduces the conclusions of replicated datasets

## Evidence

- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [discussion] From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [intro] Due to technical (e.g. sequencing depth, bias in the cell extracting method and different cell decay rate) and biological (e.g. differences in age) effects, raw cell counts cannot be compared: "Due to technical (e.g. sequencing depth, bias in the cell extracting method and different cell decay rate) and biological (e.g. differences in age) effects, raw cell counts cannot be compared"
