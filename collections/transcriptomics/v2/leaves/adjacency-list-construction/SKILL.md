---
name: adjacency-list-construction
description: Use when a custom PPIN adjacency list is needed for a biological process
  or user-supplied gene set for ORIGINS2 activity() scoring, and only a gene ID list
  is available from which to subset the full_human_ppin edge list.
when_to_use_negative:
- The biological process already ships as a pre-built edges dataset in ORIGINS2 (e.g.,
  differentiation_edges, acute_inflammatory_response_edges, dna_repair_edges) — use
  the bundled dataset instead of rebuilding.
- You already hold a normalized expression matrix and want process activity scores
  — call activity() directly; build_ppin is only the network-construction step.
- The analysis requires interactions with proteins outside the gene list — build_ppin
  keeps only edges whose both endpoints are inside the list.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ORIGINS2
  role: Provides the build_ppin() function and the bundled full_human_ppin adjacency
    list that is subset to the gene list
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Implementation language; ORIGINS2 is an open source R package used to load
    and manipulate the PPIN edge data
- name: dplyr
  role: Used inside build_ppin to filter the full human PPIN edge list so both endpoint
    genes belong to the supplied gene list
- name: magrittr
  role: Supplies the %>% pipe used to chain the dplyr filtering operations when inducing
    the subnetwork
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:c72333ea34a816479414ffa26cce1cdc2e5f1cdde9252cf06f1d6e5690a0e69b
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# adjacency-list-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a biological-process-specific protein-protein interaction network (PPIN) adjacency list by overlaying a gene ID list onto the ORIGINS2 full Human PPIN (full_human_ppin) and retaining only edges whose two endpoint genes are both in the list (build_ppin). The resulting edge data frame is the adjacency_edges input for ORIGINS2 activity() scoring of scRNA-seq data.

## When to use

Use when a custom PPIN adjacency list is needed for a biological process or user-supplied gene set for ORIGINS2 activity() scoring, and only a gene ID list is available from which to subset the full_human_ppin edge list.

## When NOT to use

- The biological process already ships as a pre-built edges dataset in ORIGINS2 (e.g., differentiation_edges, acute_inflammatory_response_edges, dna_repair_edges) — use the bundled dataset instead of rebuilding.
- You already hold a normalized expression matrix and want process activity scores — call activity() directly; build_ppin is only the network-construction step.
- The analysis requires interactions with proteins outside the gene list — build_ppin keeps only edges whose both endpoints are inside the list.

## Inputs

- genes_bp: gene ID list associated with the biological process (or user-supplied gene set) of interest
- full_human_ppin: complete full Human PPIN adjacency list bundled within the ORIGINS2 package (edge data frame)
- R environment with ORIGINS2 installed (imports dplyr and magrittr)

## Outputs

- Adjacency list data frame of PPIN edges restricted to the supplied gene list (both endpoints in genes_bp)
- CSV export of the reconstructed edge set (e.g., reconstructed_bp_ppin_edges.csv)
- Ready-to-use adjacency_edges argument for ORIGINS2 activity() scoring

## How to apply

Load ORIGINS2 (library(ORIGINS2)), which bundles the complete full Human PPIN adjacency list as the full_human_ppin data frame. Supply genes_bp, the gene ID list associated with the biological process or user gene set of interest, and call build_ppin(genes_bp, ppi_edges_complete = full_human_ppin); internally the function chains dplyr filters with the magrittr %>% pipe so that both endpoint genes of each retained edge belong to genes_bp, inducing the subnetwork. The output is an edge data frame with the same schema as the package's pre-built BP datasets (e.g., differentiation_edges, cell_cycle_edges), so it drops directly into activity(expression_matrix, adjacency_edges, normalization) as the adjacency_edges argument. Validate by re-implementing the subsetting logic in dplyr/magrittr against the gene list underlying a pre-built dataset, comparing the reconstructed edge set row-by-row to the shipped reference (output_matches_reference), and writing it as reconstructed_bp_ppin_edges.csv with the expected edge-count rows.

## Related tools

- **ORIGINS2** (Provides the build_ppin() function and the bundled full_human_ppin adjacency list that is subset to the gene list) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Implementation language; ORIGINS2 is an open source R package used to load and manipulate the PPIN edge data)
- **dplyr** (Used inside build_ppin to filter the full human PPIN edge list so both endpoint genes belong to the supplied gene list)
- **magrittr** (Supplies the %>% pipe used to chain the dplyr filtering operations when inducing the subnetwork)

## Examples

```
library(ORIGINS2); custom_edges <- build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
```

## Evaluation signals

- Row-by-row comparison of the reconstructed edge set against the corresponding pre-built BP edges dataset shipped with ORIGINS2 matches exactly (output_matches_reference).
- Every output edge has both endpoint genes present in genes_bp; no edge references a gene outside the supplied list.
- Edge count equals the number of full_human_ppin rows where both nodes are members of the gene list, and the CSV (e.g., reconstructed_bp_ppin_edges.csv) has that many rows.
- The output data frame has the same schema as the package's *_edges datasets and feeds activity(expression_matrix, adjacency_edges, normalization) without identifier errors.
- Re-running the same gene list reproduces the identical edge set (deterministic subsetting of full_human_ppin).

## Limitations

- Only the full Human PPIN adjacency list is included within the package; non-human gene sets are not covered.
- Edges are dropped unless both endpoint genes are in the gene list, so interactions to proteins outside the list (e.g., hub links) are lost.
- Gene identifiers in genes_bp must match the node identifiers used in full_human_ppin, otherwise the induced subnetwork silently shrinks.

## Evidence

- [methods] build_ppin call for custom PPIN construction: "Build any PPIN associated with any gene ID list using the function build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)"
- [intro] Custom PPIN function is an ORIGINS2 feature: "ORIGINS2 also provides a function to create custom PPINs associated with any biological process of interest."
- [readme] Function arguments: gene list plus complete PPIN adjacency list: "Where genes_bp is the gene ID list associated with the biological process of interest and ppi_edges_complete is the complete PPIN adjacency list"
- [methods] Full Human PPIN bundled with the package: "the full Human PPIN adjacency list is included within the package (full_human_ppin)"
- [readme] Output edge data frame schema used by activity(): "adjacency_edges is a data frame containing the edges of the Protein-protein Interaction Network (PPIN) asociated with the biological process of interest."
