---
name: pathway-geneset-formatting
description: Use when preparing pathway gene sets (MSigDB collections, csv, or gmt
  files) as input to SCPA or another pathway-analysis method that expects a structured
  pathway table mapping pathway names to gene sets.
when_to_use_negative:
- The input is already a formatted pathway table compatible with compare_pathways
  (no reformatting needed)
- The analysis goal is gene-level differential expression rather than pathway-level
  distribution comparison, since SCPA gene sets are irrelevant there
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: SCPA
  role: Provides format_pathways and get_paths functions to convert msigdbr output
    or pathway files into the gene-set structure used by compare_pathways
  repo: https://github.com/jackbibby1/SCPA
- name: msigdbr
  role: R package used to pull MSigDB pathway collections (e.g. Hallmark) before formatting
    with format_pathways
- name: R
  role: Execution environment for the gene-set retrieval and formatting workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:33dfbaa71e99ade87a44ca8128e668b344a32770c01cfe4f4ede4c57248a2c2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - R
  - SCPA
  - dplyr
  - msigdbr
  - tidyverse
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# pathway-geneset-formatting

## Summary

Format curated pathway gene sets (MSigDB Hallmark/KEGG/Reactome/GO/regulatory sets via msigdbr or csv/gmt files) into the gene-set table structure required by SCPA's compare_pathways function for single-cell pathway perturbation analysis.

## When to use

Use when preparing pathway gene sets (MSigDB collections, csv, or gmt files) as input to SCPA or another pathway-analysis method that expects a structured pathway table mapping pathway names to gene sets.

## When NOT to use

- The input is already a formatted pathway table compatible with compare_pathways (no reformatting needed)
- The analysis goal is gene-level differential expression rather than pathway-level distribution comparison, since SCPA gene sets are irrelevant there

## Inputs

- msigdbr output (MSigDB pathway collections, e.g. Hallmark, KEGG, Reactome, GO, regulatory pathways)
- csv pathway files (e.g. combined_metabolic_pathways.csv, h_k_r_go_pid_reg_wik.csv)
- gmt gene set files (multiple gmt files supported from SCPA v1.5.2)
- pathway gene sets from github.com/jackbibby1/SCPA/tree/main/gene_sets

## Outputs

- Formatted pathway gene-set table (pathway name to gene list mapping) compatible with compare_pathways

## How to apply

Retrieve canonical, gene-ontology, and regulatory pathways from MSigDB (e.g. Hallmark, KEGG, Reactome via msigdbr) or load a curated file such as combined_metabolic_pathways.csv or h_k_r_go_pid_reg_wik.csv. Apply SCPA's format_pathways function to msigdbr output to convert it into the two-column pathway/gene structure consumed by compare_pathways, or use get_paths to read pathway files from disk. Recent SCPA versions (v1.5.2+) also accept multiple gmt files as the pathway input. The resulting gene-set table is then passed to compare_pathways(samples, pathways, ...) alongside the extracted expression matrices, and the curated Hallmark/KEGG/Reactome sets used in the paper are hosted in the SCPA repository's gene_sets directory.

## Related tools

- **SCPA** (Provides format_pathways and get_paths functions to convert msigdbr output or pathway files into the gene-set structure used by compare_pathways) — https://github.com/jackbibby1/SCPA
- **msigdbr** (R package used to pull MSigDB pathway collections (e.g. Hallmark) before formatting with format_pathways)
- **R** (Execution environment for the gene-set retrieval and formatting workflow)

## Examples

```
library(SCPA); library(msigdbr); pathways <- format_pathways(msigdbr(species = "Homo sapiens", category = "H"))
```

## Evaluation signals

- The formatted object is accepted by compare_pathways(samples = list(...), pathways = pathways) without error
- Output contains per-pathway entries covering the expected collections (Hallmark/KEGG/Reactome/GO/regulatory from MSigDB)
- Pathway names in the formatted table match those reported in SCPA results (joined by Pathway when comparing outputs)
- When starting from msigdbr, format_pathways output columns conform to the structure shown in the SCPA quick-start workflow

## Limitations

- SCPA installation may require manually installing crossmatch (v1.3.1) and multicross (v2.1.0) dependencies before gene-set workflows can run
- Older SCPA versions did not support multiple gmt file inputs (added in v1.5.2)
- qval is the primary statistic for judging biological relevance; fold change/enrichment is only a secondary informative value, so gene-set choice should anticipate distribution-level changes

## Evidence

- [readme] multiple gmt file support: [evidence span withheld — anchor status: not_found]
- [readme] SCPA pathway input requirement: [evidence span withheld — anchor status: external_doc]
