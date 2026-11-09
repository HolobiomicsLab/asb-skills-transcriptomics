---
name: linear-regression-proportion-testing
description: Use when you need to test whether cell-type proportions shift between
  conditions in single-cell resolution data (.h5ad/AnnData), including unreplicated
  datasets where standard regression approaches fail.
when_to_use_negative:
- The dataset contains rare cell types / clusters under ~2% of all cells, where bootstrapping
  has limited benefits and results are unstable
- You intend to compare raw cell counts directly between clusters, which is invalid
  due to sequencing depth and other technical and biological effects
- Clusters do not reflect real cell types but are defined by technical parameters
  (e.g. Leiden resolution) — clustering parameters must be validated first
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanpro
  role: Implements propeller's linear regression approach on cell proportions and
    extends it with bootstrapping to simulate pseudo-replicates; provides exported
    anova/t_test helpers
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python framework Scanpro integrates into for loading and preprocessing single-cell
    data prior to proportion analysis
- name: AnnData
  role: Native input format (data matrix plus metadata) handled by the scanpro() function
- name: propeller
  role: Source of the linear regression approach on cell proportions that Scanpro
    implements
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:4f38d40bb7a31d0c72aad7da6c4bd41d37f4348813799b816fa30957ae84d468
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# linear-regression-proportion-testing

## Summary

Fits propeller-style linear regressions of per-cluster cell proportions on condition for differential compositional analysis of single-cell data, extending the regression approach with iteration-based bootstrapping to simulate pseudo-replicates when samples lack biological replicates.

## When to use

Use when you need to test whether cell-type proportions shift between conditions in single-cell resolution data (.h5ad/AnnData), including unreplicated datasets where standard regression approaches fail.

## When NOT to use

- The dataset contains rare cell types / clusters under ~2% of all cells, where bootstrapping has limited benefits and results are unstable
- You intend to compare raw cell counts directly between clusters, which is invalid due to sequencing depth and other technical and biological effects
- Clusters do not reflect real cell types but are defined by technical parameters (e.g. Leiden resolution) — clustering parameters must be validated first

## Inputs

- AnnData object (.h5ad) with columns for sample, cluster, and condition metadata (e.g. 'sample', 'cluster', 'conds')
- Prior single-cell clustering/cell-type annotation on a neighborhood graph
- Optional: n_min threshold for minimum cells per cluster

## Outputs

- Per-cluster regression results (beta coefficients and adjusted p-values, pooled across bootstrap iterations via Rubin's rule and median adjusted p-values) accessible via out.results
- Pooled statistics table (e.g. pooled_statistics.csv) with one row per retained cluster
- Pseudo-replicate cell assignment table (e.g. pseudo_replicate_cells.csv) accounting for all input cells per sample
- Plots of per-cluster proportions (out.plot(), optionally boxplots)

## How to apply

Load the dataset with scanpy/anndata (e.g. scanpro_tutorial.h5ad) and ensure metadata columns for sample, cluster, and condition exist, then call scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'). Apply n_min trimming first: clusters supported by fewer than ~2% of all cells produce unstable results and should be dropped before analysis. For unreplicated data, omit samples_col (or set to None) so Scanpro runs iteration-based bootstrapping, splitting samples into pseudo-replicates without replacement each iteration and reusing the exported anova/t_test helpers from scanpro.scanpro to fit per-cluster regressions of proportions on condition. Pool per-cluster beta coefficients across iterations with Rubin's multiple-imputation rule and take the median of adjusted p-values per cluster; retrieve results via out.results and visualize with out.plot() (e.g. out.plot(kind='boxplot')).

## Related tools

- **Scanpro** (Implements propeller's linear regression approach on cell proportions and extends it with bootstrapping to simulate pseudo-replicates; provides exported anova/t_test helpers) — https://github.com/loosolab/scanpro
- **Scanpy** (Python framework Scanpro integrates into for loading and preprocessing single-cell data prior to proportion analysis)
- **AnnData** (Native input format (data matrix plus metadata) handled by the scanpro() function)
- **propeller** (Source of the linear regression approach on cell proportions that Scanpro implements)

## Examples

```
from scanpro.scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- out.results contains one row per retained cluster with a finite pooled beta coefficient and a median adjusted p-value column
- Clusters supported by fewer than ~2% of all cells were removed by n_min trimming before testing
- The pseudo-replicate cell table accounts for all input cells per sample after the without-replacement splits
- Results are retrievable via out.results and plottable via out.plot() without errors
- Dropping samples_col triggers the bootstrapping method, confirming the unreplicated code path was used as intended

## Limitations

- Bootstrapping-based results should be seen as an indication of cell types with a proportional shift and interpreted with caution, with additional experimental validation
- The lack of replicates limits drawing definitive conclusions about variability and is per se not advisable
- Bootstrapping has limited benefits when handling rare cell types
- Clustering parameters such as Leiden resolution critically affect cluster number and size, challenging biologically meaningful proportion analysis

## Evidence

- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [discussion] From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [discussion] the results should be seen as an indication of cell types that have a proportional shift and should be interpreted with caution and additional experimental validation: "the results should be seen as an indication of cell types that have a proportional shift and should be interpreted with caution and additional experimental validation"
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
