---
name: circrna-protein-interaction-mapping
description: Use when building protein–circRNA interaction counts (C values) from
  predicted RBP–RNA interactomes plus circRNA annotations, e.g. as an input feature
  for candidate regulator scoring or clustering.
when_to_use_negative:
- You need experimentally validated (e.g. CLIP-seq) RBP–circRNA interactions rather
  than catRAPID-predicted scores; CRIT separately validates predictions against CLIP
  databases.
- You only have expression or survival data and no interactome plus coordinate annotation
  files — C values cannot be computed without RNAct, CircBase and GENCODE inputs.
- Your analysis needs partial-overlap binding-site resolution rather than whole-circRNA
  containment within the host/associated RNA.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
tools:
- name: RNAct (catRAPID)
  role: Source of genome-wide predicted protein-RNA interaction scores filtered at
    score >= 30
  repo: https://rnact.crg.eu/download
- name: CircBase
  role: Provides circRNA chromosomal position annotations (hsa_hg19_circRNA.txt) for
    coordinate alignment
  repo: http://circbase.org/download/hsa_hg19_circRNA.txt
- name: GENCODE
  role: Provides RNA chromosomal position annotations (gencode.v37.annotation.gtf)
    for coordinate alignment
  repo: https://www.gencodegenes.org/human/release_37.html
- name: CRIT
  role: Pipeline whose GET_C_value.R script computes the C value and GET_feauters_MATRIX.R
    merges it into the feature matrix
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Runtime environment for CRIT's feature-collection scripts and required packages
    (biomaRt, data.table, readr)
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:21940d2bf3ebe6952535fee3188d64199aa079d0b44e6946c1cb5fad7b9ac7b1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - CircBase
  - GENCODE
  - GoSemSim
  - MEME
  - R
  - RNAct
  - RNAct (catRAPID)
  - biomaRt
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# circRNA-protein-interaction-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many circRNAs each protein binds by filtering the RNAct catRAPID human interactome to interaction scores >= 30 and aligning CircBase circRNA coordinates to GENCODE v37 RNA coordinates, yielding the C-value feature used in CRIT's feature matrix for NMF-based circRNA regulator identification.

## When to use

Use when building protein–circRNA interaction counts (C values) from predicted RBP–RNA interactomes plus circRNA annotations, e.g. as an input feature for candidate regulator scoring or clustering.

## When NOT to use

- You need experimentally validated (e.g. CLIP-seq) RBP–circRNA interactions rather than catRAPID-predicted scores; CRIT separately validates predictions against CLIP databases.
- You only have expression or survival data and no interactome plus coordinate annotation files — C values cannot be computed without RNAct, CircBase and GENCODE inputs.
- Your analysis needs partial-overlap binding-site resolution rather than whole-circRNA containment within the host/associated RNA.

## Inputs

- catrapid_human_basic.zip — RNAct/catRAPID human protein-RNA interaction prediction scores
- hsa_hg19_circRNA.txt — human circRNA chromosomal position annotations from CircBase
- gencode.v37.annotation.gtf — human RNA annotation from GENCODE release 37
- RBP_list.RData — list of 1,344 candidate RBPs (Ensembl, HGNC, Entrez, UniProt identifiers)

## Outputs

- C_value_matrix.txt — per-protein count of bound circRNAs (C value) per cancer type
- C-value column populated within MATRIX.txt — merged feature matrix for 1,344 RBPs used by NMF.R

## How to apply

Download catrapid_human_basic.zip from RNAct (20,778 canonical Human Reference Proteome proteins x 98,608 GENCODE 'basic' RNAs) and the human circRNA annotation file hsa_hg19_circRNA.txt from CircBase, plus gencode.v37.annotation.gtf. First discard weak relationships by keeping only protein-RNA interactions with catRAPID interaction score >= 30. Then align circRNAs to RNAs according to their chromosomal position annotations extracted from CircBase and GENCODE respectively: a protein is considered to interact with a circRNA only if the position of the circRNA is completely aligned (fully contained) within the corresponding RNA. Define the C value as the number of circRNAs with which each protein binds, computed per cancer type via the CRIT script GET_C_value.R, then merge the C-value column into the feature matrix (MATRIX.txt) alongside rho_m, Zp, Zs and Zg via GET_feauters_MATRIX.R for the 1,344 RBP candidates.

## Related tools

- **RNAct (catRAPID)** (Source of genome-wide predicted protein-RNA interaction scores filtered at score >= 30) — https://rnact.crg.eu/download
- **CircBase** (Provides circRNA chromosomal position annotations (hsa_hg19_circRNA.txt) for coordinate alignment) — http://circbase.org/download/hsa_hg19_circRNA.txt
- **GENCODE** (Provides RNA chromosomal position annotations (gencode.v37.annotation.gtf) for coordinate alignment) — https://www.gencodegenes.org/human/release_37.html
- **CRIT** (Pipeline whose GET_C_value.R script computes the C value and GET_feauters_MATRIX.R merges it into the feature matrix) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Runtime environment for CRIT's feature-collection scripts and required packages (biomaRt, data.table, readr))

## Examples

```
Rscript GET_C_value.R  # run after sourcing CRIT_settings.R; reads ./DATA/input/RNAct/{catrapid_human_basic.zip, gencode.v37.annotation.gtf, hsa_hg19_circRNA.txt} and writes C_value_matrix.txt
```

## Evaluation signals

- Only interactions with catRAPID score >= 30 survive the filter; re-running with a lower cutoff should increase C values and indicates the threshold was not applied.
- A circRNA is counted only when its position is completely aligned to the corresponding RNA; counts should drop if partial-overlap logic is mistakenly used.
- C_value_matrix.txt exists and contains a non-negative integer count per protein for each cancer type.
- MATRIX.txt exists with the C-value column populated for the 1,344 RBP candidates alongside rho_m, Zp, Zs and Zg columns.
- Merged identifiers (Ensembl gene ID, HGNC symbol, Entrez ID, UniProt accession) resolve consistently between catRAPID proteins and the RBP_list.RData candidate set.

## Limitations

- Interactions are catRAPID predictions, not experimental measurements; weak relationships (score < 30) are excluded, but retained interactions may still be false positives.
- The C value counts circRNAs only when the circRNA position completely aligns to the corresponding RNA, so binding is inferred by coordinate containment rather than direct binding-site evidence.
- Coordinate alignment depends on compatible genome builds/annotations (CircBase hg19 vs GENCODE v37) — mismatched coordinate systems can silently zero out counts.
- CRIT's generalization beyond cancers (e.g. to neurodegenerative disease) is untested, so C-value features built this way may not transfer to other disease contexts.

## Evidence

- [methods] From the RNAct database, excluding the weak relationship with interaction score<30 of the protein-RNA interaction data: "From the RNAct database, excluding the weak relationship with interaction score<30 of the protein-RNA interaction data"
- [methods] C value definition and alignment rule: "It is considered that a protein interacts with a circRNA if the position of the circRNA is completely aligned to the corresponding RNA."
- [readme] C value meaning: "*C* value was defined as the number of circRNA with which each protein binds, and is used to indicate the strength of a protein interacting with circRNA."
- [readme] Input sources for alignment: "then align circRNA to RNA according to their chromosomal position annotations, which were extracted respectively from Gencode or CircBase"
