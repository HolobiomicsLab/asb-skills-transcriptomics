---
name: heteromeric-complex-filtering
description: Use when constructing or extending a ligand-receptor pair list (e.g.,
  connectomeDB2020) from CellPhoneDB (v2.0, IDs 'CPI-SSxxx') and the downstream analysis
  tool cannot represent heteromeric complexes, so only simple protein-to-protein pairs
  should be retained.
when_to_use_negative:
- When the downstream tool models heteromeric complexes (e.g., CellPhoneDB or RNA-Magnet
  themselves); exclusion is only appropriate because NATMI does not model them
- When building a database of predicted interactions rather than literature-supported
  pairs — PMID verification and simple:simple curation are specific to the literature-supported
  connectomeDB2020 standard
- When analyzing spatial transcriptomics or downstream signalling consequences — this
  skill is a database-curation step, not an analysis method
edam_operation: http://edamontology.org/operation_3677
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: NATMI
  role: The consuming tool whose '--interDB' option loads the curated pair list; heteromeric
    complexes are excluded because NATMI does not model them
  repo: https://github.com/forrest-lab/NATMI/
- name: CellPhoneDB
  role: Source resource whose v2.0 simple:simple entries (CPI-SSxxx IDs) are filtered
    to retain 121 literature-supported pairs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:9f8528aae57e76289598377842b5100509ab6cae57d885e2b9f6de49e771bc26
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
  edam_operation: http://edamontology.org/operation_3677
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# heteromeric-complex-filtering

## Summary

A curation step for building literature-supported ligand-receptor databases (e.g., connectomeDB2020) in which only simple:simple interaction entries are extracted from CellPhoneDB and heteromeric complexes are excluded, because the downstream tool (NATMI) does not model heteromeric complexes.

## When to use

Use when constructing or extending a ligand-receptor pair list (e.g., connectomeDB2020) from CellPhoneDB (v2.0, IDs 'CPI-SSxxx') and the downstream analysis tool cannot represent heteromeric complexes, so only simple protein-to-protein pairs should be retained.

## When NOT to use

- When the downstream tool models heteromeric complexes (e.g., CellPhoneDB or RNA-Magnet themselves); exclusion is only appropriate because NATMI does not model them
- When building a database of predicted interactions rather than literature-supported pairs — PMID verification and simple:simple curation are specific to the literature-supported connectomeDB2020 standard
- When analyzing spatial transcriptomics or downstream signalling consequences — this skill is a database-curation step, not an analysis method

## Inputs

- CellPhoneDB v2.0 interaction list (entries with internal IDs of format 'CPI-SSxxx')
- PubMedIDs cited by CellPhoneDB/HPRD entries for literature-support verification

## Outputs

- Set of 693 simple:simple CellPhoneDB interaction pairs (heteromeric complexes excluded)
- Subset of 121 literature-supported CellPhoneDB pairs contributing to connectomeDB2020
- connectomeDB2020.csv: two-column (Ligand, Receptor) file with 2,293 literature-supported human ligand-receptor pairs, placed in the 'lrdbs' folder

## How to apply

Download the CellPhoneDB v2.0 interaction list and identify each entry by its internal ID of format 'CPI-SSxxx'. Retain only interaction pairs annotated as simple:simple (a single ligand protein interacting with a single receptor protein), excluding heteromeric complexes, which yields 693 simple:simple pairs from which 121 pairs with primary literature support are kept. For each retained pair, verify that the cited PubMedID actually supports the interaction, dropping HPRD-derived entries whose PMID does not support the pair. This exclusion is intentional: NATMI does not model heteromeric complexes, so including them would produce unsupported edges in ExtractEdges.py. The surviving simple pairs are merged with the other sources (1751 from the authors' 2015 resource, 50 RNA-Magnet, 22 SingleCellSignalR, 9 ICELLNET, 340 new manual pairs) and annotated as secreted or plasma-membrane ligands before writing the two-column (Ligand, Receptor) csv into the 'lrdbs' folder for use via '--interDB'.

## Related tools

- **NATMI** (The consuming tool whose '--interDB' option loads the curated pair list; heteromeric complexes are excluded because NATMI does not model them) — https://github.com/forrest-lab/NATMI/
- **CellPhoneDB** (Source resource whose v2.0 simple:simple entries (CPI-SSxxx IDs) are filtered to retain 121 literature-supported pairs)

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --interDB lrdbs/connectomeDB2020.csv
```

## Evaluation signals

- From CellPhoneDB v2.0, exactly 693 simple:simple interaction pairs are extracted after excluding heteromeric complexes
- Of those, exactly 121 pairs have primary literature support and are carried into connectomeDB2020
- HPRD-derived entries whose cited PubMedID does not actually support the interaction are dropped before merging
- The final connectomeDB2020.csv contains exactly 2,293 literature-supported human ligand-receptor pairs
- The output file has the required 'Ligand' and 'Receptor' columns and is consumable by ExtractEdges.py via '--interDB'

## Limitations

- NATMI does not model heteromeric complexes, so filtering them out is a constraint of the consuming tool, not a biological statement that complexes are irrelevant
- Some prior entries (mostly HPRD-derived) had PubMedIDs that did not actually support the interaction and must be dropped during curation
- NATMI also does not model heterodimerization or requirements for co-receptors, so the filtered simple-pair list remains an approximation of receptor biology

## Evidence

- [discussion] Exclusion of heteromeric complexes from CellPhoneDB interactions: [evidence span withheld — anchor status: not_found]
- [discussion] NATMI does not model heteromeric complexes: "the handling of heteromeric complexes (CellPhoneDB18, RNA-Magnet19)"
- [abstract] connectomeDB2020 size and curation standard: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] HPRD-derived entries dropped on PMID verification: "mostly from HPRD, where the PubMedID did not actually support the interaction"
- [readme] Custom pair lists can be supplied to NATMI: "users can also provide their own custom ligand–receptor pairs lists, as described at https://github.com/forrest-lab/NATMI"
