---
name: anndata-single-cell-data-handling
description: Use when single-cell RNA-seq or scATAC data must be represented as an
  AnnData object with sample, cluster, and condition metadata columns before running
  proportion analysis or downstream Scanpy-based tools.
when_to_use_negative:
- When the data are already aggregated per-cluster cell counts rather than single-cell
  resolution data requiring clustering into cell types first
- When a dedicated tool expects a different container format (e.g. R SingleCellExperiment)
  and no Python/Scanpy integration is required
- When cluster annotations are not trustworthy — clusters might reflect technical
  parameters (e.g. Leiden resolution, batch effects) rather than real cell types
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: Scanpro
  role: Consumes the AnnData object via the scanpro() function using samples_col,
    clusters_col, and conds_col to run proportion analysis
  repo: https://github.com/loosolab/scanpro
- name: AnnData
  role: Data container format that stores expression/accessibility matrices plus sample,
    cluster, and condition metadata for Scanpro input
- name: Scanpy
  role: Python framework in which AnnData handling and prior clustering/cell-type
    annotation are performed
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:4f4c1ef10500e19817daee5b25c6e0facdc66cce267d0b8263f627bb755550a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# anndata-single-cell-data-handling

## Summary

Load, annotate, and pass single-cell data as Python AnnData (h5ad) objects so that proportion-analysis tools like Scanpro can consume sample/cluster/condition metadata natively within the Scanpy ecosystem.

## When to use

Use when single-cell RNA-seq or scATAC data must be represented as an AnnData object with sample, cluster, and condition metadata columns before running proportion analysis or downstream Scanpy-based tools.

## When NOT to use

- When the data are already aggregated per-cluster cell counts rather than single-cell resolution data requiring clustering into cell types first
- When a dedicated tool expects a different container format (e.g. R SingleCellExperiment) and no Python/Scanpy integration is required
- When cluster annotations are not trustworthy — clusters might reflect technical parameters (e.g. Leiden resolution, batch effects) rather than real cell types

## Inputs

- Single-cell gene-expression or chromatin-accessibility matrix (e.g. snRNA-seq, scATAC) as an .h5ad AnnData file
- Cell-level metadata in AnnData .obs: sample column (samples_col), cluster/cell-type column (clusters_col), condition column (conds_col), optional covariate columns (sex, age)

## Outputs

- AnnData object with validated sample, cluster, and condition annotations ready for proportion analysis
- Derived per-cell-type results tables (e.g. out.results) and CSV exports such as scanpro_kuppe_results.csv or scanpro_covariate_results.csv

## How to apply

Represent the single-cell experiment as an AnnData object, typically loaded from an h5ad file (e.g. scanpro_tutorial.h5ad via ad.read_h5ad), ensuring the .obs metadata carries the columns needed for proportion analysis: a sample identifier (samples_col, e.g. 'sample'), a cell-type/cluster label (clusters_col, e.g. 'cluster'), and a condition label (conds_col, e.g. 'conds'). This matters because AnnData is the native input Scanpro expects, and if samples_col is omitted or set to None the dataset is treated as unreplicated and Scanpro switches to its bootstrapping pseudo-replicate mode — a decision point controlled purely by how the AnnData object is annotated. Correct cell clustering and cell-type identification in the AnnData object is paramount for proportion analysis, and the article recommends larger, less specific clusters with higher cell numbers over many small clusters, since clusters supported by less than 2% of all cells produce unstable results. Covariate columns (e.g. sex, age) can also be added to the AnnData metadata for covariate-adjusted linear-model-based testing.

## Related tools

- **Scanpro** (Consumes the AnnData object via the scanpro() function using samples_col, clusters_col, and conds_col to run proportion analysis) — https://github.com/loosolab/scanpro
- **AnnData** (Data container format that stores expression/accessibility matrices plus sample, cluster, and condition metadata for Scanpro input)
- **Scanpy** (Python framework in which AnnData handling and prior clustering/cell-type annotation are performed)

## Examples

```
import scanpy as ad; adata = ad.read_h5ad('scanpro_tutorial.h5ad'); from scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- The AnnData object exposes the required metadata columns (samples_col, clusters_col, conds_col) in .obs before calling scanpro(); a missing samples_col silently triggers the bootstrapping mode
- Clusters carry biologically meaningful cell-type labels; clusters representing less than 2% of all cells are flagged as unstable by the article's benchmarking
- Loading succeeds via ad.read_h5ad on the .h5ad file and downstream out.results returns a non-empty per-cell-type statistics table
- Covariate columns (sex, age) present in .obs produce the expected covariate-adjusted significance calls (e.g. monocytes significant for age, NK cells becoming significant when age is included)

## Limitations

- Clusters supported by less than 2% of all cells produce unstable proportion-analysis results; larger, less specific clusters with higher cell numbers are recommended
- Leiden clustering parameters such as resolution critically affect the number and size of clusters, so AnnData cluster annotations may not reflect real cell types but technical parameters
- Batch effects from experimental design (different sequencing machines, different timepoints) may distort clustering encoded in the AnnData object
- Lack of replicates in the AnnData design limits conclusions about variability and is per se not advisable; bootstrapped results require additional experimental validation

## Evidence

- [intro] which saves data matrices into a format known as AnnData: [evidence span withheld — anchor status: not_found]
- [intro] a modular tool seamlessly integrating into the widely accepted Python frameworks such as Scanpy and able to natively handle anndata objects: [evidence span withheld — anchor status: not_found]
- [results] adata = ad.read_h5ad('scanpro_tutorial.h5ad'): [evidence span withheld — anchor status: not_found]
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): [evidence span withheld — anchor status: not_found]
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: not_found]
- [discussion] correct cell clustering and thus cell type identification is paramount for PA: [evidence span withheld — anchor status: not_found]
- [discussion] we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: [evidence span withheld — anchor status: not_found]
