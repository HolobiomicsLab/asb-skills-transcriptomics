---
name: cross-source-pair-merge-deduplication
description: Use when building or extending a ligand-receptor reference database from
  multiple prior L-R resources and you must reconcile, deduplicate, and verify literature
  support per pair before publishing the merged set.
when_to_use_negative:
- You need to model heteromeric complexes — CellPhoneDB complex members were excluded
  and 'we do not model them in NATMI'
- Your input is already a single clean pair list — this skill is for reconciling conflicting
  multi-source resources, not consuming a finished database
- You need predicted or inferred interactions — this merge only admits pairs with
  primary literature support
tools:
- name: NATMI
  role: Consumes the merged connectomeDB2020.csv (two-column Ligand, Receptor file
    in the lrdbs folder) via --interDB during ExtractEdges.py
  repo: https://github.com/forrest-lab/NATMI/
- name: CellPhoneDB
  role: 'Source resource: 693 simple:simple pairs extracted (heteromeric complexes
    excluded), of which 121 primary-literature-supported pairs were retained'
- name: RNA-Magnet
  role: 'Source resource: contributed 50 ligand-receptor pairs'
- name: SingleCellSignalR
  role: 'Source resource: contributed 22 pairs from the LRdb file'
  repo: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- name: ICELLNET
  role: 'Source resource: contributed 9 pairs from Supplementary Table 1'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:c3f6c7da7671b59aec32a7e078b07a650fb9ce6a94f5411e6c79f103899407b5
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-source-pair-merge-deduplication

## Summary

Assembles a curated ligand-receptor database (connectomeDB2020) by merging pair lists from prior resources (CellPhoneDB v2.0, RNA-Magnet, SingleCellSignalR/LRdb, ICELLNET, the authors' 2015 resource) with new manual curation, deduplicating and verifying literature support to yield 2293 literature-supported human pairs.

## When to use

Use when building or extending a ligand-receptor reference database from multiple prior L-R resources and you must reconcile, deduplicate, and verify literature support per pair before publishing the merged set.

## When NOT to use

- You need to model heteromeric complexes — CellPhoneDB complex members were excluded and 'we do not model them in NATMI'
- Your input is already a single clean pair list — this skill is for reconciling conflicting multi-source resources, not consuming a finished database
- You need predicted or inferred interactions — this merge only admits pairs with primary literature support

## Inputs

- CellPhoneDB v2.0 interaction list (internal IDs CPI-SSxxx, simple:simple pairs only)
- RNA-Magnet ligand-receptor pair list
- SingleCellSignalR LRdb file (LRdb_122019.txt)
- ICELLNET Supplementary Table 1
- Ramilowski et al. 2015 ligand-receptor pair list (1751 pairs)
- PubMedIDs cited as literature support for each pair

## Outputs

- connectomeDB2020.csv: exactly 2293 literature-supported human ligand-receptor pairs with Ligand and Receptor columns, stored in the 'lrdbs' folder
- Per-ligand annotation as secreted or plasma-membrane

## How to apply

Collect the source pair lists: CellPhoneDB v2.0 interactions (internal IDs of format CPI-SSxxx), RNA-Magnet pairs, the SingleCellSignalR LRdb file (LRdb_122019.txt), ICELLNET Supplementary Table 1, and the authors' 2015 resource. From CellPhoneDB, extract only 693 simple:simple interaction pairs, excluding heteromeric complexes, and retain the 121 pairs with primary literature support. Merge these with the 50 RNA-Magnet pairs, 22 SingleCellSignalR pairs, and 9 ICELLNET pairs onto the 1751 pairs from the 2015 resource, then add 340 new manually curated literature-supported pairs. Critically, verify that each cited PubMedID actually supports the corresponding interaction — HPRD-derived entries whose PMID does not support the pair are dropped — and deduplicate. Annotate each ligand as secreted or plasma-membrane, and write the result as a two-column (Ligand, Receptor) CSV in the 'lrdbs' folder so NATMI's ExtractEdges.py can consume it via '--interDB'.

## Related tools

- **NATMI** (Consumes the merged connectomeDB2020.csv (two-column Ligand, Receptor file in the lrdbs folder) via --interDB during ExtractEdges.py) — https://github.com/forrest-lab/NATMI/
- **CellPhoneDB** (Source resource: 693 simple:simple pairs extracted (heteromeric complexes excluded), of which 121 primary-literature-supported pairs were retained)
- **RNA-Magnet** (Source resource: contributed 50 ligand-receptor pairs)
- **SingleCellSignalR** (Source resource: contributed 22 pairs from the LRdb file) — https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- **ICELLNET** (Source resource: contributed 9 pairs from Supplementary Table 1)

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --interDB lrdbs/connectomeDB2020.csv
```

## Evaluation signals

- The final merged connectomeDB2020.csv contains exactly 2293 literature-supported human ligand-receptor pairs
- The CSV has the required 'Ligand' and 'Receptor' columns and is consumable by ExtractEdges.py via --interDB
- Per-source contributions reconcile: 1751 (2015 resource) + 121 (CellPhoneDB v2.0) + 50 (RNA-Magnet) + 22 (SingleCellSignalR) + 9 (ICELLNET) + 340 (new manual curation), after deduplication
- All HPRD-derived entries whose cited PubMedID does not support the interaction have been dropped
- Each ligand is annotated as secreted or plasma-membrane

## Limitations

- Pair inclusion is limited to primary literature support; entries from HPRD where the PubMedID did not actually support the interaction had to be removed, so citation claims in upstream resources cannot be trusted without verification
- Heteromeric complexes from CellPhoneDB are excluded, so complex-mediated interactions are absent from the merged set
- The database is human-specific; cross-species use in NATMI relies on NCBI HomoloGene homologues

## Evidence

- [abstract] Database size invariant: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] SingleCellSignalR source pairs: "22 pairs from SingleCellSignalR20 (source: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt)"
- [discussion] ICELLNET source pairs: "9 pairs from Supplementary Table 1 of ICELLNET21"
- [discussion] CellPhoneDB extraction filter: [evidence span withheld — anchor status: not_found]
- [discussion] Literature-support verification failure mode: "mostly from HPRD, where the PubMedID did not actually support the interaction"
- [readme] README confirmation of curated database: "NATMI uses connectomeDB2020 (the most up-to-date manually curated ligand-receptor interaction list)"
