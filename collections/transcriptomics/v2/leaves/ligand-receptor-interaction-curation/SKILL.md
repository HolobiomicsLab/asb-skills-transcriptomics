---
name: ligand-receptor-interaction-curation
description: Use when building or updating a curated ligand-receptor pair list from
  existing resources (CellPhoneDB, RNA-Magnet, SingleCellSignalR/LRdb, ICELLNET) and
  prior connectomes, requiring PubMedID-level literature verification before network
  inference.
when_to_use_negative:
- You need heteromeric ligand or receptor complexes or co-receptor requirements modeled
  — these are excluded during curation and not modeled in NATMI
- You need predicted interactions without literature support, downstream signaling
  consequences (e.g. NicheNet-style), or spatially resolved interaction inference
- You only need to run NATMI on an existing database — connectomeDB2020 ships precompiled
  with NATMI and its Docker image
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0128
tools:
- name: NATMI
  role: Consumes the curated two-column CSV via ExtractEdges.py --interDB; provides
    ExtractEdges.py, DiffEdges.py and VisInteractions.py
  repo: https://github.com/forrest-lab/NATMI/
- name: CellPhoneDB
  role: Source resource; extract only simple:simple pairs (693 candidates), excluding
    heteromeric complexes, keeping the 121 literature-supported pairs
- name: RNA-Magnet
  role: Source resource contributing 50 pairs to connectomeDB2020
- name: SingleCellSignalR
  role: Source resource; the LRdb file LRdb_122019.txt contributes 22 pairs
  repo: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- name: ICELLNET
  role: Source resource; Supplementary Table 1 contributes 9 pairs
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:20f2d8b820d1de96163dd57fce3b65bb5b93c3042fa3593c39c0c6e96d8776cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - CellPhoneDB
  - ICELLNET
  - NATMI
  - RNA-Magnet
  - SingleCellSignalR
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ligand-receptor-interaction-curation

## Summary

Curate a literature-supported ligand-receptor interaction database (connectomeDB2020, 2,293 human pairs) by merging and manually validating pairs from CellPhoneDB v2.0, RNA-Magnet, SingleCellSignalR LRdb, ICELLNET, a prior 2015 resource, and new manual curation, for use as NATMI's '--interDB' input.

## When to use

Use when building or updating a curated ligand-receptor pair list from existing resources (CellPhoneDB, RNA-Magnet, SingleCellSignalR/LRdb, ICELLNET) and prior connectomes, requiring PubMedID-level literature verification before network inference.

## When NOT to use

- You need heteromeric ligand or receptor complexes or co-receptor requirements modeled — these are excluded during curation and not modeled in NATMI
- You need predicted interactions without literature support, downstream signaling consequences (e.g. NicheNet-style), or spatially resolved interaction inference
- You only need to run NATMI on an existing database — connectomeDB2020 ships precompiled with NATMI and its Docker image

## Inputs

- CellPhoneDB v2.0 interaction list (simple:simple pairs, CPI-SSxxx internal IDs)
- RNA-Magnet ligand-receptor pair list
- SingleCellSignalR LRdb file (LRdb_122019.txt from github.com/SCA-IRCM/LRdb)
- ICELLNET Supplementary Table 1
- Authors' 2015 ligand-receptor resource (Ramilowski et al., 1,751 pairs)
- PubMedIDs cited for each candidate interaction

## Outputs

- connectomeDB2020.csv: a two-column (Ligand, Receptor) CSV containing exactly 2,293 literature-supported human ligand-receptor pairs, placed in the 'lrdbs' folder

## How to apply

Collect source pair lists: CellPhoneDB v2.0 interactions (internal IDs CPI-SSxxx), RNA-Magnet pairs, SingleCellSignalR's LRdb_122019.txt from github.com/SCA-IRCM/LRdb, and ICELLNET Supplementary Table 1. From CellPhoneDB, extract only simple:simple pairs (693 candidates) excluding heteromeric complexes, retaining the 121 pairs with primary literature support. Merge these with 50 RNA-Magnet pairs, 22 SingleCellSignalR pairs, 9 ICELLNET pairs, and 1,751 pairs from the authors' 2015 resource, then add 340 new manually curated pairs; verify each cited PubMedID actually supports the interaction, dropping HPRD-derived entries whose PMID does not support the pair, and deduplicate. Annotate each ligand as secreted or plasma-membrane, then write the result as a two-column (Ligand, Receptor) CSV into the 'lrdbs' folder so NATMI's ExtractEdges.py can consume it via '--interDB'.

## Related tools

- **NATMI** (Consumes the curated two-column CSV via ExtractEdges.py --interDB; provides ExtractEdges.py, DiffEdges.py and VisInteractions.py) — https://github.com/forrest-lab/NATMI/
- **CellPhoneDB** (Source resource; extract only simple:simple pairs (693 candidates), excluding heteromeric complexes, keeping the 121 literature-supported pairs)
- **RNA-Magnet** (Source resource contributing 50 pairs to connectomeDB2020)
- **SingleCellSignalR** (Source resource; the LRdb file LRdb_122019.txt contributes 22 pairs) — https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- **ICELLNET** (Source resource; Supplementary Table 1 contributes 9 pairs)

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --interDB lrdbs/connectomeDB2020.csv --species human --weightType mean --detectionThreshold 0.2 --out outfolder
```

## Evaluation signals

- The final connectomeDB2020.csv contains exactly 2,293 literature-supported human ligand-receptor pairs with the required Ligand and Receptor columns present
- Pair counts per source match expectations: 1,751 from the 2015 resource, 121 from CellPhoneDB v2.0, 50 from RNA-Magnet, 22 from SingleCellSignalR, 9 from ICELLNET, 340 new manually curated pairs
- Every retained pair has a cited PubMedID that actually supports the interaction; HPRD-derived entries with non-supporting PMIDs are dropped
- No heteromeric complex pairs remain from the CellPhoneDB import (only simple:simple CPI-SSxxx IDs)
- Running NATMI ExtractEdges.py with the curated database as --interDB on a reference dataset (e.g. Skelly cardiac) reproduces 126,738 predicted edges with detectionThreshold 0.2

## Limitations

- Heteromeric complexes are excluded and not modeled, nor are heterodimerization or co-receptor requirements
- Predictions based on mRNA expression rather than protein concentration, and subject to dropouts; specificity weights are dataset-specific (local), not global
- Curated annotations depend on the original authors' cell annotations, and some source entries (mostly from HPRD) had PubMedIDs that did not actually support the interaction and must be dropped

## Evidence

- [discussion] CellPhoneDB simple:simple extraction: [evidence span withheld — anchor status: not_found]
- [discussion] SingleCellSignalR source: "22 pairs from SingleCellSignalR20 (source: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt)"
- [abstract] connectomeDB2020 definition: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] HPRD PMID cleanup: "mostly from HPRD, where the PubMedID did not actually support the interaction"
- [discussion] No heteromeric modeling: "thus we do not model them in NATMI"
