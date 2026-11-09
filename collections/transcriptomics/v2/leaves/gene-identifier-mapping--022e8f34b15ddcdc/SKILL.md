---
name: gene-identifier-mapping
description: Use when you have a gene ID list associated with a biological process
  of interest and need to derive its PPIN adjacency edge list for downstream PPIN
  activity scoring in ORIGINS2.
when_to_use_negative:
- The biological process of interest already has a pre-built *_edges dataset bundled
  with ORIGINS2 (18 BP are included, e.g. differentiation_edges, cell_cycle_edges)
  — use those directly instead of rebuilding.
- You need PPIN activity scores rather than a network — call activity(expression_matrix,
  adjacency_edges, normalization) instead of build_ppin.
- The gene list is not in the same identifier space as the full_human_ppin adjacency
  list — mapping/identifier harmonization must be done first, otherwise no or incorrect
  edges will be retained.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0128
tools:
- name: ORIGINS2
  role: Provides the build_ppin function and the bundled full_human_ppin adjacency
    list used to subset the network
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Imported dependency used to filter the full human PPIN edge list to edges
    whose endpoints belong to the gene list
- name: magrittr
  role: Imported dependency providing the %>% pipe used in the edge-filtering workflow
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github()
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:d235292397ce39f576c4da0962a28b6a254d1ffe8590abc56892e8ac21f812ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielasenraoka/ORIGINS2
  repo_url: https://github.com/danielasenraoka/ORIGINS2
  tools:
  - ORIGINS2
  - R
  - dplyr
  - magrittr
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gene-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a custom protein-protein interaction network (PPIN) by overlaying a user-supplied gene ID list onto the full human PPIN and subsetting the edges whose endpoints both belong to the list, as implemented in the ORIGINS2 build_ppin function.

## When to use

Use when you have a gene ID list associated with a biological process of interest and need to derive its PPIN adjacency edge list for downstream PPIN activity scoring in ORIGINS2.

## When NOT to use

- The biological process of interest already has a pre-built *_edges dataset bundled with ORIGINS2 (18 BP are included, e.g. differentiation_edges, cell_cycle_edges) — use those directly instead of rebuilding.
- You need PPIN activity scores rather than a network — call activity(expression_matrix, adjacency_edges, normalization) instead of build_ppin.
- The gene list is not in the same identifier space as the full_human_ppin adjacency list — mapping/identifier harmonization must be done first, otherwise no or incorrect edges will be retained.

## Inputs

- genes_bp: a gene ID list associated with the biological process of interest
- ppi_edges_complete: the complete Human PPIN adjacency list (full_human_ppin, bundled with ORIGINS2)

## Outputs

- A subset adjacency edge list (data frame) of protein-protein interactions restricted to the supplied gene list
- Optionally a CSV file (e.g. reconstructed_bp_ppin_edges.csv) with one row per retained edge

## How to apply

Load the ORIGINS2 package (library(ORIGINS2)) which bundles the full_human_ppin adjacency list — the complete Human PPIN edge list. Call build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), where genes_bp is the gene ID list associated with the biological process of interest. The function filters the full human PPIN edge list (via dplyr/magrittr %>% operations) so that both endpoint genes of each edge are present in the supplied gene list, returning the subset adjacency list of protein-protein interactions specific to that process. Validate the reconstruction by applying it to a gene list underlying one of the pre-built *_edges datasets (e.g. differentiation_edges) and comparing the resulting edge set row-by-row against the shipped reference dataset. Write the reconstructed adjacency list to CSV (e.g. reconstructed_bp_ppin_edges.csv) for reuse with the activity() function.

## Related tools

- **ORIGINS2** (Provides the build_ppin function and the bundled full_human_ppin adjacency list used to subset the network) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Imported dependency used to filter the full human PPIN edge list to edges whose endpoints belong to the gene list)
- **magrittr** (Imported dependency providing the %>% pipe used in the edge-filtering workflow)
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github())

## Examples

```
library(ORIGINS2); custom_edges <- build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
```

## Evaluation signals

- Apply the implemented build_ppin to the gene list of a pre-built BP (e.g. the genes underlying differentiation_edges) and confirm the reconstructed edge set matches the reference *_edges dataset row-by-row (output_matches_reference).
- Check that every edge in the output has both endpoint genes contained in the supplied genes_bp list.
- Verify the output is a valid adjacency edge list (data frame of gene–gene pairs) and that the CSV file exists with the expected edge-count rows.
- Confirm the resulting edge list works as the adjacency_edges argument to activity(expression_matrix, adjacency_edges, normalization) without errors.

## Limitations

- Only edges from the bundled full_human_ppin can be retained; interactions absent from the complete Human PPIN cannot be recovered by this subsetting approach.
- Genes in the input list not present in full_human_ppin will contribute no edges; identifier mismatches silently reduce network size.
- The result is a static subnetwork; the normalization parameter of the downstream activity() call must be set explicitly (FALSE gives the sum of all PPIN edges; TRUE normalizes by the number of edges of the PPIN).

## Evidence
