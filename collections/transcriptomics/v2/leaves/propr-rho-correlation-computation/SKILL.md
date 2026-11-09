---
name: propr-rho-correlation-computation
description: Use when a CellexalVR user has defined a cell group in-session and requests
  a transcription factor network, requiring TF-restricted rho correlations and an
  adjacency-list file for the VR network generator.
when_to_use_negative:
- The expression matrix has not been restricted to AnimalTFDB-annotated transcription
  factors — correlating all genes with propr rho is not what this skill does
- No cell group has been defined in a CellexalVR session and no backend cellexalvrR
  data files are available for the selected cells
- A full-genome co-expression or regulatory-network inference (e.g. ligand-receptor
  or GRN methods) is wanted rather than the TF-only network CellexalVR's generator
  expects
tools:
- name: propr
  role: Computes the pairwise rho correlations between the restricted transcription
    factors on the selected cells
- name: cellexalvrR
  role: R package backend that loads expression values for the selected cell group
    and performs the in-session TF network calculation
  repo: https://www.github.com/sonejilab/cellexalvrr
- name: R
  role: Runtime environment (R 4.0.0 or later) invoked by CellexalVR through Rscript.exe
    to execute the backend correlation computation
- name: CellexalVR
  role: Consumes tf_network.txt and generates the transcription factor network visualization
    in the VR main scene
  repo: https://github.com/sonejilab/cellexalvr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:8852573dd3881a281130a5121b63e9b09082154145e9403fe342196922b236c0
schema_version: 0.2.0
metadata:
  license_tier: open
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
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# propr-rho-correlation-computation

## Summary

Computes pairwise co-expression correlations between transcription factors from a user-defined CellexalVR cell group using the propr package's rho metric, and exports the top 130 interactions as a tf_network.txt adjacency list that CellexalVR's transcription factor network generator can render in VR.

## When to use

Use when a CellexalVR user has defined a cell group in-session and requests a transcription factor network, requiring TF-restricted rho correlations and an adjacency-list file for the VR network generator.

## When NOT to use

- The expression matrix has not been restricted to AnimalTFDB-annotated transcription factors — correlating all genes with propr rho is not what this skill does
- No cell group has been defined in a CellexalVR session and no backend cellexalvrR data files are available for the selected cells
- A full-genome co-expression or regulatory-network inference (e.g. ligand-receptor or GRN methods) is wanted rather than the TF-only network CellexalVR's generator expects

## Inputs

- Gene expression values for a user-defined cell group from CellexalVR data files/sqlite3 expression database
- Gene-to-transcription-factor annotations from AnimalTFDB
- A cellexalvrR object / R session with the selected cells' expression matrix

## Outputs

- tf_network.txt — adjacency list text file with exactly 130 TF-TF interaction entries, consumable by the CellexalVR transcription factor network generator

## How to apply

Load the gene expression values for the selected cell group from the CellexalVR data files/sqlite3 expression database via the cellexalvrR R package backend. Restrict the expression matrix to genes annotated as transcription factors in AnimalTFDB. Compute all pairwise TF-TF correlations using the propr package's rho metric (the default setting) on the selected cells. Rank the pairwise correlations by magnitude and retain the top 130 interactions. Write these retained interactions as an adjacency list text file (tf_network.txt) that the CellexalVR transcription factor network generator consumes; the InputReader script in CellexalVR reads the R package output and calls the generators that create transcription factor networks.

## Related tools

- **propr** (Computes the pairwise rho correlations between the restricted transcription factors on the selected cells)
- **cellexalvrR** (R package backend that loads expression values for the selected cell group and performs the in-session TF network calculation) — https://www.github.com/sonejilab/cellexalvrr
- **R** (Runtime environment (R 4.0.0 or later) invoked by CellexalVR through Rscript.exe to execute the backend correlation computation)
- **CellexalVR** (Consumes tf_network.txt and generates the transcription factor network visualization in the VR main scene) — https://github.com/sonejilab/cellexalvr

## Examples

```
library(propr); pr <- propr(tf_expr[, selected_cells], metric = "rho")  # rho is the default metric
```

## Evaluation signals

- tf_network.txt exists in the expected CellexalVR data/output location after the R script run completes
- tf_network.txt contains exactly 130 transcription factor interaction entries
- Entries are formatted as an adjacency list of TF-TF pairs (gene identifiers annotated in AnimalTFDB) and the file is parseable by the CellexalVR transcription factor network generator
- Correlations were computed only on the cells of the user-defined group, not the full dataset
- The transcription factor network graph is successfully rendered in the CellexalVR main scene after the generator reads the file

## Limitations

- Backend calculations such as this one take longer to complete depending on how many cells/groups are selected — large cell groups slow the VR session
- CellexalVR must be configured with the path to Rscript.exe (R 4.0.0+) in Config/config.xml for the backend R package to run
- The output is specific to CellexalVR's TF network generator (adjacency list of the top 130 interactions); it is not a general-purpose weighted co-expression network with correlation values

## Evidence

- [methods] workflow step 3-4: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 5 + validation: [evidence span withheld — anchor status: not_found]
- [readme] InputReader script role: "reads the data from the CellexalVR R package and calls the appropriate functions to create graphs and transcription factor networks and so on."
- [readme] cellexalvrR purpose: [evidence span withheld — anchor status: external_doc]
