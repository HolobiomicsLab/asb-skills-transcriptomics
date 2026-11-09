---
name: transcription-factor-network-inference
description: 'Use when a CellexalVR in-session cell group needs a TF-only co-expression
  network: the input is expression values for the selected cells and the goal is a
  tf_network.txt consumed by the VR TF network generator.'
when_to_use_negative:
- When the goal is a general gene co-expression network rather than one restricted
  to AnimalTFDB-annotated transcription factors
- When the user wants other group-based outputs (differential expression heatmaps,
  pseudotime trajectories, manual annotations) instead of a TF network
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellexalVR
  role: VR platform whose transcription factor network generator consumes tf_network.txt
    and renders the network from the user's in-session cell group
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package backend that loads expression values from CellexalVR data files/sqlite3
    and performs in-session calculations via RScriptRunner
  repo: https://github.com/sonejilab/cellexalvrR
- name: propr
  role: R package supplying the rho correlation metric (default setting) used to compute
    pairwise TF-TF correlations
- name: R
  role: Runtime language (R 4.0.0+) in which the backend correlation workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:2520052dfd067bfc4f913e588597a3227351dd81c8fa22e83d278a55c75c3ef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - RScriptRunner
  - cellexalvrR
  - propr
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# transcription-factor-network-inference

## Summary

Builds a transcription factor (TF) interaction network from a user-defined CellexalVR cell group by computing pairwise propr rho correlations between TFs and writing a top-130 adjacency list that the CellexalVR TF network generator renders in VR.

## When to use

Use when a CellexalVR in-session cell group needs a TF-only co-expression network: the input is expression values for the selected cells and the goal is a tf_network.txt consumed by the VR TF network generator.

## When NOT to use

- When the goal is a general gene co-expression network rather than one restricted to AnimalTFDB-annotated transcription factors
- When the user wants other group-based outputs (differential expression heatmaps, pseudotime trajectories, manual annotations) instead of a TF network

## Inputs

- Gene expression values for a user-defined cell group (from CellexalVR data files or the sqlite3 expression database)
- Transcription factor annotation list from AnimalTFDB
- CellexalVR in-session cell group selection

## Outputs

- tf_network.txt — adjacency list text file containing exactly 130 TF-TF interaction entries
- Transcription factor network object rendered in the CellexalVR VR scene

## How to apply

Load the gene expression values for the selected cell group from the CellexalVR data files/sqlite3 expression database using the cellexalvrR R package backend, which CellexalVR invokes through RScriptRunner. Restrict the expression matrix to genes annotated as transcription factors in AnimalTFDB so the network contains only TF-TF edges. Compute all pairwise correlations between the restricted TFs using the propr package's rho metric (default setting) on the selected cells only, then rank the pairwise correlations by magnitude and retain the top 130 interactions. Write the retained interactions as an adjacency list text file named tf_network.txt, which the CellexalVR transcription factor network generator consumes and renders alongside other group-based outputs such as differential expression heatmaps.

## Related tools

- **CellexalVR** (VR platform whose transcription factor network generator consumes tf_network.txt and renders the network from the user's in-session cell group) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package backend that loads expression values from CellexalVR data files/sqlite3 and performs in-session calculations via RScriptRunner) — https://github.com/sonejilab/cellexalvrR
- **propr** (R package supplying the rho correlation metric (default setting) used to compute pairwise TF-TF correlations)
- **R** (Runtime language (R 4.0.0+) in which the backend correlation workflow is executed)

## Examples

```
Rscript -e 'library(cellexalvrR); rho <- propr(t(expr[, cells])); top <- findTopRho(rho, N=130); writeAdjacency(top, file="tf_network.txt")'
```

## Evaluation signals

- tf_network.txt exists after the backend run completes
- tf_network.txt contains exactly 130 transcription factor interaction entries
- The file is formatted as an adjacency list consumable by the CellexalVR TF network generator
- Every edge connects two genes annotated as TFs in AnimalTFDB (no non-TF genes appear)
- Correlations were computed only on cells belonging to the user-selected group, not the full dataset

## Limitations

- Backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected — large in-session selections slow the correlation step
- TF coverage depends on AnimalTFDB annotation; genes not annotated as TFs are excluded from the network
- No external validation or benchmark of CellexalVR against other VR tools is reported in this text

## Evidence

- [other] TF restriction and rho correlation step: [evidence span withheld — anchor status: not_found]
- [other] Top-130 ranking and retention: [evidence span withheld — anchor status: not_found]
- [readme] Generator renders TF networks in the Unity main scene: "The generators are responsible for generating things such as graphs, heatmaps and transcription factor networks."
- [readme] cellexalvrR is the R backend for in-session calculations: [evidence span withheld — anchor status: external_doc]
