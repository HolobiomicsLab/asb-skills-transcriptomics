---
name: gene-expression-subsetting-to-selected-cells
description: Use when a user has defined a cell group during a CellexalVR VR session
  and downstream backend calculations (e.g. TF-TF correlation networks with propr
  rho) need the expression matrix restricted to exactly those selected cells.
when_to_use_negative:
- No user-defined cell group exists yet (e.g. the task operates on the full dataset
  or a pre-computed feature table rather than an in-session selection)
- The dataset has not been converted to CellexalVR input files via cellexalvrR/export2cellexalvr,
  so there is no CellexalVR data directory or sqlite3 expression database to read
  from
tools:
- name: cellexalvrR
  role: R package backend that stores/reads the expression data and performs in-session
    calculations on the selected cell group
  repo: https://github.com/sonejilab/cellexalvrR
- name: CellexalVR
  role: VR frontend where the user defines the cell group; its InputReader reads the
    data returned from the CellexalVR R package and its generators produce graphs,
    heatmaps and transcription factor networks
  repo: https://github.com/sonejilab/cellexalvr
- name: propr
  role: 'Downstream consumer of the subsetted expression matrix: computes pairwise
    rho correlations among transcription factors for the top-130 TF network'
- name: RScriptRunner
  role: CellexalVR component that invokes Rscript.exe (path set in Config/config.xml)
    to execute the cellexalvrR backend routines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:e7f69a3a52cdc3b2a612508fbe4dfbe4c1ec706aa2155ccc6e9a6c85030c94d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - R
  - RScriptRunner
  - cellexalvrR
  - propr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-expression-subsetting-to-selected-cells

## Summary

Extracts the gene expression values for a user-defined cell group from CellexalVR's data files/sqlite3 expression database using the cellexalvrR R package backend, producing a cell-group-restricted expression matrix that feeds in-session analyses such as the propr-based transcription factor network generator.

## When to use

Use when a user has defined a cell group during a CellexalVR VR session and downstream backend calculations (e.g. TF-TF correlation networks with propr rho) need the expression matrix restricted to exactly those selected cells.

## When NOT to use

- No user-defined cell group exists yet (e.g. the task operates on the full dataset or a pre-computed feature table rather than an in-session selection)
- The dataset has not been converted to CellexalVR input files via cellexalvrR/export2cellexalvr, so there is no CellexalVR data directory or sqlite3 expression database to read from

## Inputs

- CellexalVR data files / sqlite3 expression database for the loaded dataset
- User-defined cell group (selection made in-session in CellexalVR)
- cellexalvrR object / input files produced by export2cellexalvr (or as_cellexalvrR on Seurat/AnnData objects)

## Outputs

- Expression matrix restricted to the selected cell group
- Group-specific backend results (e.g. tf_network.txt adjacency list of 130 TF-TF interactions, heatmap DE genes) consumed by CellexalVR generators

## How to apply

Read the gene expression values for the selected cell group from the CellexalVR data files or the sqlite3 expression database via the cellexalvrR R package backend, invoked by CellexalVR's RScriptRunner using the Rscript.exe path configured in Config/config.xml. Restrict (subset) the expression matrix columns to the cells the user selected in-session; subsetting is what makes subsequent backend calculations (Wilcox-based differential expression for heatmaps, hierarchical clustering of top N marker genes with default 250, or propr rho correlations among AnimalTFDB-annotated transcription factors) operate only on the group of interest. The InputReader script on the CellexalVR side reads the data returned from the CellexalVR R package and routes it to the appropriate generator (graph, heatmap, or transcription factor network). Rationale: keeping the computation backend-side in cellexalvrR lets CellexalVR generate group-specific outputs (differential expression heatmaps, tf_network.txt adjacency lists) alongside other group-based outputs without exporting raw data out of the VR workflow.

## Related tools

- **cellexalvrR** (R package backend that stores/reads the expression data and performs in-session calculations on the selected cell group) — https://github.com/sonejilab/cellexalvrR
- **CellexalVR** (VR frontend where the user defines the cell group; its InputReader reads the data returned from the CellexalVR R package and its generators produce graphs, heatmaps and transcription factor networks) — https://github.com/sonejilab/cellexalvr
- **propr** (Downstream consumer of the subsetted expression matrix: computes pairwise rho correlations among transcription factors for the top-130 TF network)
- **RScriptRunner** (CellexalVR component that invokes Rscript.exe (path set in Config/config.xml) to execute the cellexalvrR backend routines)

## Examples

```
Rscript -e 'library(cellexalvrR); devtools::install_github("sonejilab/cellexalvrR")'  # cellexalvrR backend invoked via Rscript.exe configured in Config/config.xml
```

## Evaluation signals

- The subsetted matrix contains exactly the cells of the user-defined group and no others
- Downstream tf_network.txt exists and contains exactly 130 transcription factor interaction entries as an adjacency list
- Group-based outputs (differential expression heatmap, TF network) appear in CellexalVR alongside the defined group
- Backend calculation runtime scales with the number of cells/groups selected — unexpectedly long runs suggest wrong group size or misconfigured RscriptFilePath
- An HTML report of the session actions is generated after the session finishes, confirming the backend call completed

## Limitations

- Backend calculations (e.g. identifying differentially expressed genes) take longer to complete depending on how many cells/groups are selected
- Each user must have a local copy of the data being analysed; data files are not transmitted over RPCs in multi-user mode
- The R installation path must be configured (RscriptFilePath in Config/config.xml); R >= 4.0.0 and the cellexalvrR package must be installed, and pandoc is needed for HTML reports

## Evidence

- [readme] InputReader routes R results to generators: "which reads the data from the CellexalVR R package and calls the appropriate functions to create graphs and transcription factor networks and so on"
- [readme] Generators produce TF networks and heatmaps: "The generators are responsible for generating things such as graphs, heatmaps and transcription factor networks."
- [readme] cellexalvrR provides backend in-session calculations: [evidence span withheld — anchor status: external_doc]
- [results] Input files come from cellexalvrR: "converted to a set of CellexalVR input files using our accompanying R package called cellexalvrR"
- [discussion] Runtime scales with selected cells/groups: "backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected"
