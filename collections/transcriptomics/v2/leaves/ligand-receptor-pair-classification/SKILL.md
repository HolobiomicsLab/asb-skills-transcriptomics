---
name: ligand-receptor-pair-classification
description: Use when building a cell-cell communication network from scRNA-seq data
  and you must split a curated L-R pair database into membrane-ligand, secreted-ligand,
  and membrane-receptor classes before computing interaction weights.
when_to_use_negative:
- Published methods' single undistinguished L-R list is sufficient — i.e., you do
  not need to separate membrane-ligand from secreted-ligand significance calculations
- Your organism is not supported by the homologene human/mouse homolog database and
  you cannot provide custom gene-identifier mappings
- You only have a single stationary-condition expression map and no cell-type structure
  — the localization-aware split and downstream per-cell-type weighting require cell-type
  labeled data
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
tools:
- name: dsCellNet
  role: Provides preparepairs() which splits L-R pairs into membrane-ligand, secreted-ligand,
    and membrane-receptor classes
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps L-R gene identifiers across species via the human/mouse homolog database
    during pair classification
- name: CellPhoneDB
  role: Source of known L-R pairs (1,396 pairs) incorporated into the default classification
    database
- name: NATMI
  role: Source of known L-R pairs (4,071 pairs) incorporated into the default classification
    database
- name: SingleCellSignalR
  role: Source of known L-R pairs (3,251 pairs) incorporated into the default classification
    database
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:133408633acb9c479aa03a2074437d1b4f7859f9835681d25de5cedd668e9586
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
  - Mfuzz
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ligand-receptor-pair-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify known ligand-receptor pairs into membrane-ligand, secreted-ligand and membrane-receptor categories using protein localization information and multi-species homology mapping, as done in dsCellNet to enable separate significance calculation for membrane vs secreted ligand-mediated interactions.

## When to use

Use when building a cell-cell communication network from scRNA-seq data and you must split a curated L-R pair database into membrane-ligand, secreted-ligand, and membrane-receptor classes before computing interaction weights.

## When NOT to use

- Published methods' single undistinguished L-R list is sufficient — i.e., you do not need to separate membrane-ligand from secreted-ligand significance calculations
- Your organism is not supported by the homologene human/mouse homolog database and you cannot provide custom gene-identifier mappings
- You only have a single stationary-condition expression map and no cell-type structure — the localization-aware split and downstream per-cell-type weighting require cell-type labeled data

## Inputs

- Gene expression matrix from scRNA-seq (e.g., E15.5 mouse embryonic cerebral cortex, 11,670 single cells, GSE153164)
- Curated ligand-receptor pair databases (Ramilowski et al. 2015, CellPhoneDB, NATMI, SingleCellSignalR)
- Public protein localization information
- Species name (for homologene human/mouse homolog mapping)

## Outputs

- memsercpros object: L-R pairs partitioned into membrane-ligand, secreted-ligand, and membrane-receptor classes
- Species-mapped L-R pair database usable by singlenet for candidate sender-receiver enumeration

## How to apply

Collect known L-R pairs from published resources (Ramilowski et al. 2015 with 2,557 pairs, CellPhoneDB with 1,396 pairs, NATMI with 4,071 pairs, SingleCellSignalR with 3,251 pairs) together with public protein localization information to build a multi-species supported membrane or secreted L-R database; in dsCellNet this yields 5,744 interactions in the default list. Run preparepairs(inpexp, species, dbloc, mainlab) to split these pairs into membrane-ligand, secreted-ligand, and membrane-receptor classes, mapping gene identifiers across species with the homologene human/mouse homolog database. The rationale is that membrane-ligand and secreted-ligand pairs have different expression profiles, so dsCellNet computes significance separately for each class, weighting membrane-ligand edges as W = Mean(Ligand) x Mean(Receptor) and secreted-ligand edges as W = Sum(Ligand) x Mean(Receptor). Users can supplement the default list with their own L-R genes if the default database misses expected interactions.

## Related tools

- **dsCellNet** (Provides preparepairs() which splits L-R pairs into membrane-ligand, secreted-ligand, and membrane-receptor classes) — github.com/songzh523/dsCellNet
- **homologene** (Maps L-R gene identifiers across species via the human/mouse homolog database during pair classification)
- **CellPhoneDB** (Source of known L-R pairs (1,396 pairs) incorporated into the default classification database)
- **NATMI** (Source of known L-R pairs (4,071 pairs) incorporated into the default classification database)
- **SingleCellSignalR** (Source of known L-R pairs (3,251 pairs) incorporated into the default classification database)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab)
```

## Evaluation signals

- preparepairs returns a memsercpros object whose L-R pairs are each assigned to exactly one of the three classes: membrane-ligand, secreted-ligand, or membrane-receptor
- Total classified pairs should be consistent with the expected default list size (~5,744 interactions, listed as 5,684 in the tools comparison table)
- All gene identifiers in the classified pairs resolve correctly after homologene human/mouse mapping (no unmatched identifiers for the chosen species)
- Downstream singlenet edges respect the class-specific weight formulas: membrane-ligand W = Mean(Ligand) x Mean(Receptor); secreted-ligand W = Sum(Ligand) x Mean(Receptor)
- Classified pairs feed a permutation test (100 shuffles of cell-type labels) and only interactions with p < 0.05 are retained in the communication network

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g., MG–Neurons); users can supply supplementary L-R genes
- The cause of missing MG–Neurons interactions in individual samples remains unexplained
- Published resources could not distinguish membrane-ligand vs secreted-ligand mediated receptor interactions given their different expression profiles, which is why the classification step is necessary

## Evidence
