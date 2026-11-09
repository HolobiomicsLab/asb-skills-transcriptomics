---
name: multi-tissue-significance-pattern-comparison
description: Use when you have Scanpro proportion-analysis results for several tissue
  subsets (e.g. heart, intestine, skeletal muscle) and need to verify that significant
  cell-type proportion shifts agree between the replicated design and the non-replicated
  bootstrapping mode.
when_to_use_negative:
- The input AnnData has no prior cell clustering/cell-type annotation — proportion
  analysis requires clustering into cell types before it can run
- The question is differential gene expression within individual subpopulations rather
  than shifts in the proportions of cell types
- Clusters of interest contain less than 2% of all cells — benchmarking showed such
  rare clusters produce unstable results and bootstrapping has limited benefit there
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpro
  role: Runs the proportion analysis (linear regression with ANOVA on cell proportions)
    in both replicated and bootstrapping modes whose significance calls are compared
  repo: https://github.com/loosolab/scanpro
- name: AnnData
  role: Data format holding the single-cell matrix plus 'sample'/'cluster'/'conds'
    metadata that Scanpro consumes natively
- name: Scanpy
  role: Python single-cell framework Scanpro integrates into; used for loading and
    handling the AnnData object
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:99a1dd23a15bbfc0e046402a36faa9ab499bef3a9d1fd312e9b68df19c5c63e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multi-tissue-significance-pattern-comparison

## Summary

A validation-style skill for cell-type proportion analysis in which significance calls from Scanpro are extracted per tissue and cross-compared between replicated runs and bootstrapped (pseudo-replicate) runs, checking that tissue-specialized cell types are consistently flagged and shared/ubiquitous cell types consistently not flagged.

## When to use

Use when you have Scanpro proportion-analysis results for several tissue subsets (e.g. heart, intestine, skeletal muscle) and need to verify that significant cell-type proportion shifts agree between the replicated design and the non-replicated bootstrapping mode.

## When NOT to use

- The input AnnData has no prior cell clustering/cell-type annotation — proportion analysis requires clustering into cell types before it can run
- The question is differential gene expression within individual subpopulations rather than shifts in the proportions of cell types
- Clusters of interest contain less than 2% of all cells — benchmarking showed such rare clusters produce unstable results and bootstrapping has limited benefit there

## Inputs

- AnnData (.h5ad) single-cell chromatin accessibility dataset with 'sample', 'cluster', and 'conds' metadata columns (e.g. scanpro_tutorial.h5ad-style object)
- Subset of cells from multiple tissues (heart, intestine, skeletal muscle) with prior clustering/cell-type annotation
- Expected significance pattern to compare against (which cell types should be significant per tissue)

## Outputs

- Combined per-tissue Scanpro results table with p-values and significance calls (e.g. scanpro_fetal_tissue_results.csv)
- Side-by-side comparison of significance calls from replicated vs bootstrapped Scanpro runs
- Optional boxplots of proportions per cell type via out.plot()

## How to apply

Load the fetal scATAC (chromatin accessibility) AnnData object, subset it to heart, intestine, and skeletal muscle cells, and run Scanpro twice per subset: once on the replicated design with scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds') and once without samples_col so the iteration-based bootstrapping simulates pseudo-replicates. Extract out.results for each tissue and each mode, recording p-values and significance calls per cell type. Then build the expected pattern — tissue-specialized cells (cardiomyocytes, intestinal epithelial cells, skeletal muscle cells) significant, while the smooth muscle cells shared across tissues non-significant — and check that both the replicated and bootstrapped runs reproduce it, writing the combined per-tissue table to CSV (e.g. scanpro_fetal_tissue_results.csv). The rationale is that Scanpro's bootstrapping 'often results in the same conclusion as replicated datasets', so agreement between modes is itself evidence the proportion analysis is robust; divergent calls point to unstable clusters or rare cell types.

## Related tools

- **Scanpro** (Runs the proportion analysis (linear regression with ANOVA on cell proportions) in both replicated and bootstrapping modes whose significance calls are compared) — https://github.com/loosolab/scanpro
- **AnnData** (Data format holding the single-cell matrix plus 'sample'/'cluster'/'conds' metadata that Scanpro consumes natively)
- **Scanpy** (Python single-cell framework Scanpro integrates into; used for loading and handling the AnnData object)

## Examples

```
from scanpro.scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- Significance pattern reproduced: cardiomyocytes, intestinal epithelial cells, and skeletal muscle cells are called significant in their respective tissues while smooth muscle cells are non-significant
- Bootstrap-based (non-replicated) run and replicated run reach the same conclusion for every cell type in each tissue
- out.results returns a table containing p-values for each cluster/condition combination for both run modes
- Cell types falling below the <2% of total cells threshold are excluded or flagged as unstable before pattern comparison
- Final CSV contains per-tissue rows with matching significance calls between the two modes

## Limitations

- Bootstrapping-based results on non-replicated datasets should be seen as an indication of proportional shifts and interpreted with caution with additional experimental validation
- Bootstrapping has limited benefits when handling rare cell types
- Clusters supported by less than 2% of all cells produce unstable results
- Clustering parameters (e.g. resolution for Leiden clustering) critically affect cluster number and size, and clusters might not necessarily reflect cell types, so care is needed when selecting clusters and interpreting proportion analysis

## Evidence

- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality: "we here present Scanpro (Single-cell analysis of proportions). Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality"
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [discussion] bootstrapping often reaches the same conclusion as replicated datasets: "Although this approach allows for proportion analysis on non replicated datasets and often results in the same conclusion as replicated datasets (see benchmarks)"
- [discussion] clusters under 2% of cells give unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [readme] no samples_col triggers bootstrapping mode: [evidence span withheld — anchor status: external_doc]
