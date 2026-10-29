---
name: cross-species-gene-homolog-mapping
description: Use when scRNA-seq gene identifiers come from a species (e.g., mouse)
  whose names must be mapped to homologs (e.g., human) before matching against a default
  L-R pair database such as dsCellNet's 5,744-pair list.
when_to_use_negative:
- When the expression matrix and the L-R pair database already share the same species
  namespace — mapping adds noise or drops genes with ambiguous one-to-many homologs.
- When the study needs interactions involving ligands or receptors absent from the
  default L-R gene list; the paper notes this size limitation can cause missing interactions
  (e.g., MG–Neurons) and recommends supplying supplementary L-R genes instead.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3170
tools:
- name: homologene
  role: Provides the default human/mouse homolog database used by preparepairs to
    map gene identifiers to a consistent species namespace
- name: dsCellNet
  role: Hosts the preparepairs function and the default multi-species supported membrane-or-secreted
    L-R database
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:5f992b49b5d5483dfbfa802399cbb29804a776733e9aa08e6249045a38270658
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
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cross-species-gene-homolog-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps ligand and receptor gene identifiers to a consistent species namespace using the homologene human/mouse homolog database so that dsCellNet can classify L-R pairs into membrane-ligand, secreted-ligand, and membrane-receptor classes before network inference.

## When to use

Use when scRNA-seq gene identifiers come from a species (e.g., mouse) whose names must be mapped to homologs (e.g., human) before matching against a default L-R pair database such as dsCellNet's 5,744-pair list.

## When NOT to use

- When the expression matrix and the L-R pair database already share the same species namespace — mapping adds noise or drops genes with ambiguous one-to-many homologs.
- When the study needs interactions involving ligands or receptors absent from the default L-R gene list; the paper notes this size limitation can cause missing interactions (e.g., MG–Neurons) and recommends supplying supplementary L-R genes instead.

## Inputs

- scRNA-seq expression matrix loaded via datainput (inpexp, data frame)
- species name parameter (e.g., human or mouse)
- path to the L-R pair database (dbloc) with default human/mouse homologs
- result file prefix (mainlab)

## Outputs

- memsercpros: classified multi-species supported membrane-or-secreted L-R pair set (membrane-ligand, secreted-ligand, membrane-receptor classes)
- default L-R pair table (default_lr_pairs.csv) with 5,744 interactions after merging source databases and localization filtering

## How to apply

Run the preparepairs(inpexp, species, dbloc, mainlab) step of dsCellNet after loading the expression matrix with datainput(datloc, expnam, typnam, ...); preparepairs uses the homologene package (>= 1.4.68, default human/mouse homologs) to map gene identifiers against the database at dbloc. The mapped ligand and receptor genes are then annotated with GO Cellular Compartment terms — GO:0005576 (extracellular region), GO:0005615 (extracellular space), GO:0009986 (cell surface), GO:0005886 (plasma membrane) — and filtered so only secreted or plasma-membrane ligands and plasma-membrane receptors are kept. The rationale is that known L-R pairs from Ramilowski et al. 2015, CellPhoneDB, NATMI, and SingleCellSignalR are largely human-curated, so cross-species homolog mapping is required to apply them to mouse data such as the E15.5 embryonic cerebral cortex dataset (GSE153164). The output, memsercpros, feeds singlenet() as the classified membrane/secreted ligand-receptor pair set for downstream per-time-point network inference.

## Related tools

- **homologene** (Provides the default human/mouse homolog database used by preparepairs to map gene identifiers to a consistent species namespace)
- **dsCellNet** (Hosts the preparepairs function and the default multi-species supported membrane-or-secreted L-R database) — github.com/songzh523/dsCellNet

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab)
```

## Evaluation signals

- The default L-R pair list contains exactly 5,744 interactions, matching the count reported in the paper (note: the tools comparison table lists 5,684).
- Retained ligands are annotated with GO:0005576/GO:0005615 (secreted) or GO:0005886 (plasma membrane); retained receptors are annotated with GO:0005886 only.
- Every downstream inferred interaction involves ligand-receptor pairs whose genes were successfully mapped by homologene; unmapped genes must not appear in memsercpros.
- All four source pair lists (Ramilowski 2,557; CellPhoneDB 1,396; NATMI 4,071; SingleCellSignalR 3,251) were merged and de-duplicated before localization filtering.
- The memsercpros output is accepted by singlenet() without identifier-mismatch errors when run on the E15.5 mouse cortex data.

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g., MG–Neurons); users can supply their own L-R genes as supplementary support.
- Homolog mapping depends on the homologene human/mouse homolog database; species outside human/mouse defaults are not addressed in the article.
- Genes lacking GO Cellular Compartment annotations (GO:0005576, GO:0005615, GO:0009986, GO:0005886) are dropped during localization filtering even if biologically plausible.

## Evidence
