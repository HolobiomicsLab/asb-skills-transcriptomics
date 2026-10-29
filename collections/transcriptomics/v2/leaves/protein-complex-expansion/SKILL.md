---
name: protein-complex-expansion
description: Use when building or rebuilding a ligand-receptor reference database
  for cell-cell communication inference and raw source records contain heteromeric
  complexes that must be decomposed into pairwise gene-gene records.
when_to_use_negative:
- The input is already an expanded, gene-symbol-level LR pair table — re-expansion
  and ortholog mapping would duplicate records or break identifiers
- The goal is per-dataset inference of significant LR associations from scRNA-seq
  expression (use CommPath's findLRpairs on a CommPath object instead of rebuilding
  the reference database)
- The analysis requires pathway-level filtering of LR interactions (findLRpath/filterLR),
  which operates downstream of the reference database rather than on the raw CellPhoneDB/FANTOM5
  sources
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CommPath
  role: R package whose bundled LR and pathway databases are the target artifact of
    the expansion and curation procedure
  repo: https://github.com/yingyonghui/CommPath
- name: CellPhoneDB
  role: External source resource whose curated ligand-receptor interactions (including
    heteromeric complexes) are parsed and expanded
- name: R
  role: Language in which both sources are parsed, complexes expanded to gene pairs,
    symbols/orthologs mapped, and the merged table deduplicated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:e389cf20b9befe125325219a3fb2e146632b6b5e86bfb00210b92d02a50c6a70
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
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# protein-complex-expansion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs CommPath's curated ligand-receptor reference database by parsing CellPhoneDB and FANTOM5 resources and expanding heteromeric protein complexes into individual (ligand, receptor) gene-pair interactions, mapped to official gene symbols per supported species.

## When to use

Use when building or rebuilding a ligand-receptor reference database for cell-cell communication inference and raw source records contain heteromeric complexes that must be decomposed into pairwise gene-gene records.

## When NOT to use

- The input is already an expanded, gene-symbol-level LR pair table — re-expansion and ortholog mapping would duplicate records or break identifiers
- The goal is per-dataset inference of significant LR associations from scRNA-seq expression (use CommPath's findLRpairs on a CommPath object instead of rebuilding the reference database)
- The analysis requires pathway-level filtering of LR interactions (findLRpath/filterLR), which operates downstream of the reference database rather than on the raw CellPhoneDB/FANTOM5 sources

## Inputs

- CellPhoneDB curated ligand-receptor interaction table (downloaded)
- FANTOM5 ligand-receptor interaction resource (downloaded)
- Target species identifier (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegans)

## Outputs

- Merged, deduplicated LR pair reference table keyed on (ligand, receptor) official gene symbols with a source-provenance column (e.g. LR_pairs_reference.csv)
- Species-resolved gene symbol mappings for ligand and receptor entries

## How to apply

Download the CellPhoneDB curated ligand-receptor interaction table and the FANTOM5 ligand-receptor resource, then parse both into standardized ligand/receptor records in R, expanding heteromeric protein complexes into individual gene-pair interactions so that a complex involving multiple subunits contributes one row per (subunit, partner) combination. Map all identifiers to official gene symbols for human (hsapiens) and via ortholog mapping for the other CommPath-supported species (mmusculus, rnorvegicus, drerio, dmelanogaster, celegans). Merge the two sources into a single table keyed on the (ligand, receptor) gene pair, deduplicate records, and add a source-provenance column. The merged database ships inside CommPath and is consumed downstream by createCommPath/findLRpairs as the reference for statistical identification of potential LR associations (logFC.thre=0, p.thre=0.01 in the HCC vignette).

## Related tools

- **CommPath** (R package whose bundled LR and pathway databases are the target artifact of the expansion and curation procedure) — https://github.com/yingyonghui/CommPath
- **CellPhoneDB** (External source resource whose curated ligand-receptor interactions (including heteromeric complexes) are parsed and expanded)
- **R** (Language in which both sources are parsed, complexes expanded to gene pairs, symbols/orthologs mapped, and the merged table deduplicated)

## Examples

```
devtools::install_github("yingyonghui/CommPath"); tumor.obj <- createCommPath(expr.mat = tumor.expr, cell.info = tumor.label, species = "hsapiens")
```

## Evaluation signals

- The merged table is non-empty and each row is a concrete (ligand, receptor) pair of valid official gene symbols — no residual complex identifiers remain unexpanded
- Heteromeric entries from CellPhoneDB yield multiple rows (cartesian expansion of subunits against their partners), and deduplication removes exact duplicate pairs contributed by both CellPhoneDB and FANTOM5
- Ligands/receptors used in the CommPath vignette (e.g., ligand CXCL12, receptor ACKR1) resolve to valid gene symbols in the saved table
- Pairs contributed by both source databases are present, and the source-provenance column correctly attributes each record to CellPhoneDB, FANTOM5, or both
- Loading the database in CommPath with species='hsapiens' (or ortholog-mapped species such as mmusculus) succeeds when creating a CommPath object

## Limitations

- The article/README do not state version numbers for the CellPhoneDB or FANTOM5 resources, so the exact provenance snapshot of the curated database is undocumented
- Ortholog-based symbol mapping for non-human species (mmusculus, rnorvegicus, drerio, dmelanogaster, celegans) can lose pairs whose ortholog relationships are missing or ambiguous
- Content of supplementary artifacts referenced in the article (e.g., Supplementary Table S1) is not included in the provided text, limiting verification of the database contents

## Evidence
