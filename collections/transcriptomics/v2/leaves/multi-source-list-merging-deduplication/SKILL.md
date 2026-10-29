---
name: multi-source-list-merging-deduplication
description: Use when a curated list must be assembled by unioning multiple published
  source databases and removing duplicate entries across sources, e.g. building a
  default L-R interaction database for cell-cell communication inference.
when_to_use_negative:
- The user already supplies a curated L-R list — dsCellNet explicitly allows users
  to support their own L-R genes as supplementary when the default list's size is
  limiting
- The task requires distinguishing membrane-ligand vs secreted-ligand mediated interactions
  at the pair level, which the published sources merged here do not resolve
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0089
tools:
- name: dsCellNet
  role: Package whose preparepairs step consumes the merged default L-R database via
    dbloc; the merged list is written to default_lr_pairs.csv
  repo: github.com/songzh523/dsCellNet
- name: CellPhoneDB
  role: One of four incorporated source databases of L-R pairs (1,396 pairs)
- name: NATMI
  role: One of four incorporated source databases of L-R pairs (4,071 pairs)
- name: SingleCellSignalR
  role: One of four incorporated source databases of L-R pairs (3,251 pairs)
- name: homologene
  role: Maps gene identifiers to default human/mouse homologs so pairs from all sources
    share a consistent species namespace
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:3e4ed491a6017c5fd0281ded59b4e6922c210ff2dbf5f5aff76149907d17eb5d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-source-list-merging-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merge several published ligand-receptor pair databases (Ramilowski et al. 2015, CellPhoneDB, NATMI, SingleCellSignalR) into a single non-redundant, multi-species supported membrane-or-secreted L-R database, as done to build dsCellNet's default L-R pair list.

## When to use

Use when a curated list must be assembled by unioning multiple published source databases and removing duplicate entries across sources, e.g. building a default L-R interaction database for cell-cell communication inference.

## When NOT to use

- The user already supplies a curated L-R list — dsCellNet explicitly allows users to support their own L-R genes as supplementary when the default list's size is limiting
- The task requires distinguishing membrane-ligand vs secreted-ligand mediated interactions at the pair level, which the published sources merged here do not resolve

## Inputs

- Published L-R pair lists from Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396 pairs), NATMI (4,071 pairs), and SingleCellSignalR (3,251 pairs)
- Gene identifier namespaces from each source requiring homolog mapping
- GO Cellular Compartment annotations (GO:0005576 extracellular region, GO:0005615 extracellular space, GO:0009986 cell surface, GO:0005886 plasma membrane)

## Outputs

- default_lr_pairs.csv — merged, deduplicated, multi-species supported membrane-or-secreted L-R database (default list of 5,744 interactions)
- Per-source pair counts recorded for validation against the published totals

## How to apply

Load each published L-R pair source separately, recording per-source counts (Ramilowski et al. 2015: 2,557 pairs; CellPhoneDB: 1,396; NATMI: 4,071; SingleCellSignalR: 3,251). Map gene identifiers into a consistent species namespace using the homologene package with default human/mouse homologs, as done by the preparepairs step against dbloc. Annotate ligand and receptor genes with GO Cellular Compartment terms (GO:0005576, GO:0005615, GO:0009986, GO:0005886) and filter: keep ligands that are secreted (GO:0005576/GO:0005615) or plasma-membrane (GO:0005886) and keep only receptors annotated at the plasma membrane (GO:0005886). Merge the filtered pair lists, remove duplicates across sources, and write the result to default_lr_pairs.csv as the multi-species supported membrane-or-secreted L-R database.

## Related tools

- **dsCellNet** (Package whose preparepairs step consumes the merged default L-R database via dbloc; the merged list is written to default_lr_pairs.csv) — github.com/songzh523/dsCellNet
- **CellPhoneDB** (One of four incorporated source databases of L-R pairs (1,396 pairs))
- **NATMI** (One of four incorporated source databases of L-R pairs (4,071 pairs))
- **SingleCellSignalR** (One of four incorporated source databases of L-R pairs (3,251 pairs))
- **homologene** (Maps gene identifiers to default human/mouse homologs so pairs from all sources share a consistent species namespace)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab)
```

## Evaluation signals

- Retained default L-R pair list contains exactly 5,744 interactions, matching the count reported in the paper's results
- Sum of per-source counts before merging equals 2,557 + 1,396 + 4,071 + 3,251 source pairs as stated in the article
- Every kept ligand is annotated secreted (GO:0005576/GO:0005615) or plasma-membrane (GO:0005886); every kept receptor is plasma-membrane (GO:0005886)
- No duplicate L-R pairs remain across sources after the deduplication step
- Note: the paper's tools comparison table lists the LR database size as 5,684, a discrepancy with the 5,744 result count that should be flagged during validation

## Limitations

- The default L-R gene list's size can limit detection of some interactions (e.g., MG–Neurons); users can supply supplementary L-R genes
- Merged published sources cannot distinguish membrane-ligand vs secreted-ligand mediated receptor interactions given their different expression profiles
- The paper reports inconsistent counts: 5,744 interactions in results vs 5,684 in the methods comparison table

## Evidence
