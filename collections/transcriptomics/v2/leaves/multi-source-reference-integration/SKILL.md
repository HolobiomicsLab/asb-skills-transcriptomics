---
name: multi-source-reference-integration
description: Use when building a ligand-receptor reference by integrating multiple
  external LR resources (CellPhoneDB, FANTOM5) into one standardized, deduplicated,
  species-mapped table for cell-cell communication inference.
when_to_use_negative:
- You need signaling pathway gene sets (KEGG, WikiPathways, Reactome, GO) for pathway
  scoring rather than the LR pair reference — those are separate CommPath databases
- You already have a valid LR reference and want to infer statistically significant
  LR associations from scRNA-seq expression data — that is CommPath's findLRpairs
  step, not reference curation
- Your species is outside the six CommPath-supported species (hsapiens, mmusculus,
  rnorvegicus, drerio, dmelanogaster, celegans), since ortholog mapping is only defined
  for these
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3170
tools:
- name: CommPath
  role: R package whose curated multi-species LR and pathway databases are built by
    this integration skill and consumed for LR interaction inference
  repo: https://github.com/yingyonghui/CommPath
- name: CellPhoneDB
  role: External source resource contributing curated ligand-receptor interactions
    to the merged reference database
- name: R
  role: Language in which the two sources are parsed, gene-symbol-mapped, merged,
    deduplicated, and validated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:8f5dc30178781b85da61eebf3fa9ffec5e5f0c9413b0beaf0a85cf9c24fbcfb4
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
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multi-source-reference-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs CommPath's curated ligand-receptor reference database by merging and standardizing LR interactions from CellPhoneDB and FANTOM5 into a single deduplicated, provenance-tagged table for six supported species.

## When to use

Use when building a ligand-receptor reference by integrating multiple external LR resources (CellPhoneDB, FANTOM5) into one standardized, deduplicated, species-mapped table for cell-cell communication inference.

## When NOT to use

- You need signaling pathway gene sets (KEGG, WikiPathways, Reactome, GO) for pathway scoring rather than the LR pair reference — those are separate CommPath databases
- You already have a valid LR reference and want to infer statistically significant LR associations from scRNA-seq expression data — that is CommPath's findLRpairs step, not reference curation
- Your species is outside the six CommPath-supported species (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegans), since ortholog mapping is only defined for these

## Inputs

- CellPhoneDB curated ligand-receptor interaction table
- FANTOM5 ligand-receptor interaction resource
- Target species identifier (e.g., hsapiens) from the set: hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegans

## Outputs

- LR_pairs_reference.csv — merged, deduplicated (ligand, receptor) gene-pair table with a source-provenance column
- Species-specific LR database variants with gene symbols resolved via ortholog mapping

## How to apply

Download the CellPhoneDB curated ligand-receptor interaction table and the FANTOM5 LR resource, then parse both into standardized ligand/receptor records in R, expanding heteromeric protein complexes into individual gene-pair interactions. Map all identifiers to official gene symbols for human (hsapiens) and via ortholog mapping for the other CommPath-supported species (mmusculus, rnorvegicus, drerio, dmelanogaster, celegans). Merge the two sources into one table keyed on the (ligand, receptor) gene pair, deduplicate records, and add a source-provenance column so downstream tools can trace each pair's origin. Validate before saving LR_pairs_reference.csv: confirm the table is non-empty, that pairs contributed by both sources are present, and that ligands/receptors used in the CommPath vignette (e.g., ligand CXCL12, receptor ACKR1) resolve to valid gene symbols. This merged reference is the lookup table CommPath consumes together with scRNA-seq expression profiles and cell annotation labels to infer potential LR associations between cell clusters.

## Related tools

- **CommPath** (R package whose curated multi-species LR and pathway databases are built by this integration skill and consumed for LR interaction inference) — https://github.com/yingyonghui/CommPath
- **CellPhoneDB** (External source resource contributing curated ligand-receptor interactions to the merged reference database)
- **R** (Language in which the two sources are parsed, gene-symbol-mapped, merged, deduplicated, and validated)

## Examples

```
tumor.obj <- createCommPath(expr.mat = tumor.expr, cell.info = tumor.label, species = "hsapiens")
```

## Evaluation signals

- The merged LR table is non-empty and each row resolves to a valid (ligand, receptor) gene symbol pair
- Pairs contributed by both CellPhoneDB and FANTOM5 are present in the table and distinguishable via the provenance column
- No duplicate records remain when keyed on the (ligand, receptor) gene pair
- Vignette-referenced ligands/receptors (e.g., CXCL12 as ligand, ACKR1 as receptor) resolve to valid gene symbols before saving LR_pairs_reference.csv
- Species variants exist for all six supported species (hsapiens, mmusculus, rnorvegicus, drerio, dmelanogaster, celegans) via ortholog mapping

## Limitations

- Heteromeric protein complexes are expanded into individual gene-pair interactions, which multiplies records relative to the source resources
- Ortholog mapping is only provided for the six CommPath-supported species; other species are not covered
- The article text and README describe the curated databases but do not document version numbers or a changelog for the LR reference or the compared tools

## Evidence
