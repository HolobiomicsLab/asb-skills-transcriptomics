---
name: anndata-metadata-handling-sample-cluster-condition
description: Use when an .h5ad/AnnData single-cell dataset must be passed to scanpro()
  and the per-cell metadata columns identifying replicate samples, clusters, and condition
  labels need to be specified or verified first.
when_to_use_negative:
- The data is not cell-level single-cell data but already aggregated per-sample per-cluster
  counts — Scanpro expects clustered single-cell resolution AnnData input
- Cells have not yet been clustered/annotated into cell types; proportion analysis
  presupposes correct cell clustering, and clusters supported by less than 2% of all
  cells produce unstable results
- The goal is differential gene expression within populations, not differential compositional
  (proportion) analysis between cell types
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Scanpro
  role: Target tool that reads samples_col, clusters_col, and conds_col metadata columns
    from the AnnData object to run proportion analysis
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Loads the single-cell dataset into the AnnData format whose metadata columns
    Scanpro consumes
- name: AnnData
  role: Native input format storing cell matrices together with the .obs metadata
    columns (sample, cluster, condition) required by Scanpro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:f538728906e91bc8c50691600bbdf31930c33470cda3e98245936efcccc9176a
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
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# anndata-metadata-handling-sample-cluster-condition

## Summary

Prepare AnnData objects so that per-cell metadata columns carrying sample (replicate), cluster (cell type), and condition annotations are correctly wired into Scanpro's proportion analysis function, since Scanpro reads these columns directly from the AnnData object rather than from separate count tables.

## When to use

Use when an .h5ad/AnnData single-cell dataset must be passed to scanpro() and the per-cell metadata columns identifying replicate samples, clusters, and condition labels need to be specified or verified first.

## When NOT to use

- The data is not cell-level single-cell data but already aggregated per-sample per-cluster counts — Scanpro expects clustered single-cell resolution AnnData input
- Cells have not yet been clustered/annotated into cell types; proportion analysis presupposes correct cell clustering, and clusters supported by less than 2% of all cells produce unstable results
- The goal is differential gene expression within populations, not differential compositional (proportion) analysis between cell types

## Inputs

- AnnData (.h5ad) object with .obs columns: sample/replicate identifier, cluster/cell-type label, and condition label
- Column names for samples_col, clusters_col, and conds_col as they appear in adata.obs

## Outputs

- AnnData with metadata columns verified/renamed so scanpro() can consume them
- A scanpro results object whose out.results table carries per-cluster p-values keyed to those metadata columns

## How to apply

Load the single-cell dataset as an AnnData object with Scanpy (e.g. ad.read_h5ad('scanpro_tutorial.h5ad')) and confirm that .obs contains a column per cell for the replicate/sample identifier, the cluster/cell-type label, and the condition of interest. These columns are required to have been produced by prior single-cell analysis — clustering into cell types is a prerequisite for proportion analysis. When calling scanpro(), pass samples_col='sample', clusters_col='cluster', and conds_col='conds' to bind Scanpro to those metadata columns; the column names must match what is actually in .obs for the specific dataset. If samples_col is omitted or set to None, Scanpro treats the dataset as unreplicated and automatically switches to the iteration-based bootstrapping method (sim_scanpro) that simulates pseudo-replicates.

## Related tools

- **Scanpro** (Target tool that reads samples_col, clusters_col, and conds_col metadata columns from the AnnData object to run proportion analysis) — https://github.com/loosolab/scanpro
- **Scanpy** (Loads the single-cell dataset into the AnnData format whose metadata columns Scanpro consumes)
- **AnnData** (Native input format storing cell matrices together with the .obs metadata columns (sample, cluster, condition) required by Scanpro)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, clusters_col='clusters', conds_col='condition', samples_col='sample')
```

## Evaluation signals

- out.results returns a populated per-cluster results table without KeyError on samples_col/clusters_col/conds_col, confirming the metadata columns were correctly bound
- Every cell in adata.obs carries a non-null value in all three columns; missing sample labels silently trigger the unreplicated bootstrapping path instead of the replicated linear-regression path
- Cluster labels correspond to a completed clustering/cell-type-identification step, and no cluster holds less than 2% of all cells (a threshold below which the article reports unstable results)
- Changing samples_col to None in the call flips the tool into bootstrapping mode, verifying the replicated/unreplicated decision point is wired correctly

## Limitations

- Column names passed to scanpro() (e.g. 'sample', 'cluster', 'conds') are dataset-specific and must match the actual .obs columns; the tutorial dataset uses conds_col='conds'
- Clusters supported by less than 2% of all cells in an experiment produce unstable results; larger, less specific clusters are recommended over many highly specific clusters with small cell numbers
- Metadata may carry batch effects (e.g. different sequencing machines, different collection timepoints), which can affect clustering and hence the condition/cluster labels stored in .obs
- If samples_col is not given or set to None, the dataset is silently assumed to be not replicated and bootstrapping runs — an easy mistake when sample metadata exists but was not linked

## Evidence

- [methods] run scanpro function on anndata object: "from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [readme] bootstrapping trigger: [evidence span withheld — anchor status: external_doc]
- [intro] clustering prerequisite: "Based on the clustering of single-cell resolution data into cell types, differential compositional analysis, which we define as proportion analysis (PA), can be performed."
- [discussion] unstable rare clusters: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [discussion] AnnData native input: "we introduced Scanpro as a versatile tool to perform PA in a highly automated fashion, which is able to handle input from Python AnnData objects"
