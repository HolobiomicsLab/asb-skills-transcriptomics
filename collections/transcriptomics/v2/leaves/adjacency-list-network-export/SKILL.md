---
name: adjacency-list-network-export
description: Use when you need to derive a transcription factor co-expression network
  from a cell group selected during a CellexalVR VR session and export it as an adjacency
  list the in-session TF network generator can consume.
when_to_use_negative:
- The goal is a general-purpose network exported for external graph tools rather than
  consumption by CellexalVR's TF network generator
- No user-defined cell group exists and correlations would be computed on all cells
  rather than the in-session selection
- Genes of interest are not transcription factors and are not annotated in AnimalTFDB
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: VR platform whose InputReader reads the R backend output and whose generator
    builds the transcription factor network from tf_network.txt
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package backend providing in-session functions that load expression data
    and run the network calculation via RScriptRunner
  repo: https://github.com/sonejilab/cellexalvrR
- name: propr
  role: R package supplying the rho correlation metric (default) used to score pairwise
    TF-TF co-expression
- name: R
  role: Language in which the backend TF-network workflow (cellexalvrR + propr) is
    implemented and run via Rscript.exe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:6a5b2fbdd886f35d3e032d145ed132953b96d9da1ff47bd3bf12f1805785eede
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# adjacency-list-network-export

## Summary

Generates a CellexalVR-consumable transcription factor network by computing pairwise propr rho correlations between AnimalTFDB-annotated TFs in a user-defined cell group and writing the top-ranked interactions as an adjacency-list text file (tf_network.txt).

## When to use

Use when you need to derive a transcription factor co-expression network from a cell group selected during a CellexalVR VR session and export it as an adjacency list the in-session TF network generator can consume.

## When NOT to use

- The goal is a general-purpose network exported for external graph tools rather than consumption by CellexalVR's TF network generator
- No user-defined cell group exists and correlations would be computed on all cells rather than the in-session selection
- Genes of interest are not transcription factors and are not annotated in AnimalTFDB

## Inputs

- Gene expression values for a user-defined cell group from CellexalVR data files or the sqlite3 expression database
- Transcription factor annotations from AnimalTFDB
- cellexalvrR object / CellexalVR input files for the loaded dataset

## Outputs

- tf_network.txt: adjacency-list text file containing exactly 130 ranked TF-TF interactions
- In-session transcription factor network rendered in CellexalVR

## How to apply

Load gene expression values for the user-defined cell group from the CellexalVR data files or the sqlite3 expression database using the cellexalvrR R package backend. Restrict the expression matrix to genes annotated as transcription factors in AnimalTFDB. Compute pairwise TF-TF correlations on the selected cells with the propr package's rho metric (the default setting), rank all pairs by magnitude, and retain the top 130 interactions. Write the retained interactions as an adjacency list text file named tf_network.txt, the format expected by CellexalVR's InputReader and TF network generator, which creates the network alongside other group-based outputs such as differential expression heatmaps.

## Related tools

- **CellexalVR** (VR platform whose InputReader reads the R backend output and whose generator builds the transcription factor network from tf_network.txt) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package backend providing in-session functions that load expression data and run the network calculation via RScriptRunner) — https://github.com/sonejilab/cellexalvrR
- **propr** (R package supplying the rho correlation metric (default) used to score pairwise TF-TF co-expression)
- **R** (Language in which the backend TF-network workflow (cellexalvrR + propr) is implemented and run via Rscript.exe)

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- tf_network.txt exists in the expected output location after the session backend run
- tf_network.txt contains exactly 130 transcription factor interaction entries
- File is formatted as an adjacency list that CellexalVR's transcription factor network generator can consume without parsing errors
- All genes in the network are restricted to AnimalTFDB-annotated transcription factors from the selected cell group
- Correlations were computed with propr's rho metric and retained pairs are the top 130 by magnitude

## Limitations

- Backend calculations run through cellexalvrR take longer as more cells/groups are selected, so large selections slow network generation
- The 130-interaction cutoff is a fixed default and is not adaptively tuned to network density or dataset size
- Requires a VR-ready Windows 10 workstation with an R installation (R 4.0.0+) and the cellexalvrR package; only HTC Vive and Valve Index were used during development

## Evidence

- [readme] InputReader reads R output and builds TF networks: "reads the data from the CellexalVR R package and calls the appropriate functions to create graphs and transcription factor networks and so on"
- [readme] Generators create TF networks among other outputs: "The generators are responsible for generating things such as graphs, heatmaps and transcription factor networks."
- [results] TF networks are generated from user-defined cell groups in-session: [evidence span withheld — anchor status: not_found]
