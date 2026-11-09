---
name: ligand-receptor-pair-database-curation
description: Use when building or rebuilding the reference LR interaction database
  needed for cell-cell communication inference (e.g., CommPath, CellPhoneDB-style
  pipelines) from CellPhoneDB and FANTOM5 resources.
when_to_use_negative:
- The goal is downstream LR inference itself (findLRpairs, filterLR) rather than building
  the reference database — those steps require the curated database plus scRNA-seq
  expression data as input.
- Curating signaling pathway databases (KEGG, WikiPathways, Reactome, GO) for pathway
  activation scoring — that is a separate CommPath curation task.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: CommPath
  role: R package whose curated LR reference database (CellPhoneDB + FANTOM5 derived)
    this skill reconstructs; ships species-specific LR databases for six species
  repo: https://github.com/yingyonghui/CommPath
- name: CellPhoneDB
  role: Source database of curated ligand-receptor interactions merged into the LR
    reference
- name: R
  role: Language used to parse, merge, deduplicate, and validate the source interaction
    tables
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:a3c495b09916e2027e56f0593afbdb2cb328c15ad1c722cb7c2d831a72fd8961
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yingyonghui/CommPath
  repo_url: https://github.com/yingyonghui/CommPath
  tools:
  - CellPhoneDB
  - CommPath
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ligand-receptor-pair-database-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates a ligand-receptor (LR) pair reference database by merging interactions from CellPhoneDB and FANTOM5 into a standardized (ligand, receptor) gene-pair table that serves as CommPath's reference for inferring potential LR associations between cell clusters.

## When to use

Use when building or rebuilding the reference LR interaction database needed for cell-cell communication inference (e.g., CommPath, CellPhoneDB-style pipelines) from CellPhoneDB and FANTOM5 resources.

## When NOT to use

- The goal is downstream LR inference itself (findLRpairs, filterLR) rather than building the reference database — those steps require the curated database plus scRNA-seq expression data as input.
- Curating signaling pathway databases (KEGG, WikiPathways, Reactome, GO) for pathway activation scoring — that is a separate CommPath curation task.

## Inputs

- CellPhoneDB curated ligand-receptor interaction table
- FANTOM5 ligand-receptor interaction resource
- Target species designation (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegans)

## Outputs

- Merged, deduplicated LR_pairs_reference.csv keyed on (ligand, receptor) gene pairs with a source-provenance column

## How to apply

Download the CellPhoneDB curated ligand-receptor interaction table and the FANTOM5 ligand-receptor interaction resource, then parse both into standardized ligand/receptor records in R, expanding heteromeric protein complexes into individual gene-pair interactions. Map all gene/protein identifiers to official gene symbols for human (hsapiens) and via ortholog mapping for the other CommPath-supported species (mmusculus, rnorvegicus, drerio, dmelanogaster, celegans). Merge the two sources into one table keyed on the (ligand, receptor) gene pair, deduplicate records, and add a source-provenance column. The resulting curated database is used together with scRNA-seq expression profiles and cell annotation labels as the input reference for inferring potential LR associations between cell clusters.

## Related tools

- **CommPath** (R package whose curated LR reference database (CellPhoneDB + FANTOM5 derived) this skill reconstructs; ships species-specific LR databases for six species) — https://github.com/yingyonghui/CommPath
- **CellPhoneDB** (Source database of curated ligand-receptor interactions merged into the LR reference)
- **R** (Language used to parse, merge, deduplicate, and validate the source interaction tables) — https://github.com/yingyonghui/CommPath

## Examples

```
tumor.obj <- createCommPath(expr.mat = tumor.expr, cell.info = tumor.label, species = "hsapiens")
```

## Evaluation signals

- LR_pairs_reference.csv is non-empty after merging and deduplication.
- LR pairs contributed by both CellPhoneDB and FANTOM5 are present in the merged table.
- Ligands/receptors used in the CommPath vignette (e.g., ligand CXCL12, receptor ACKR1) resolve to valid official gene symbols.
- Heteromeric complexes from CellPhoneDB are expanded into individual gene-pair records rather than kept as complex-level entries.
- Every record carries a source-provenance value indicating CellPhoneDB, FANTOM5, or both.

## Limitations

- Coverage is bounded by what CellPhoneDB and FANTOM5 curate; LR pairs absent from both sources cannot be inferred.
- Non-human species depend on ortholog mapping quality; CommPath supports only six species (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegans).
- The curated database serves intercellular signaling inference only — it does not by itself prioritize functional LR associations or pathway-mediated communication chains.

## Evidence

- [other] CommPath's LR reference database is assembled by curating ligand-receptor interactions from the CellPhoneDB and FANTOM5 resources: [evidence span withheld — anchor status: not_found]
