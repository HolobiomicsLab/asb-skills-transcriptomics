---
name: literature-support-verification
description: Use when building or validating a curated ligand-receptor interaction
  database where each pair must carry primary literature support, or when auditing
  merged resources for pairs whose cited PMID does not actually back the interaction.
when_to_use_negative:
- You need computationally predicted interactions or downstream signalling consequences
  rather than literature-supported direct pairs (the authors point to NicheNet/SoptSC
  for signalling consequences and note NATMI does not model heteromeric complexes)
- The input is already a verified, deduplicated curated database and no PMID audit
  or merging of resources is required
- You need ligand-receptor pairs that include heteromeric complexes, which were deliberately
  excluded during extraction from CellPhoneDB
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3068
tools:
- name: NATMI
  role: 'Consumer of the verified database: ExtractEdges.py ingests the curated two-column
    CSV placed in the ''lrdbs'' folder via ''--interDB'''
  repo: https://github.com/forrest-lab/NATMI/
- name: CellPhoneDB
  role: Source of 121 simple:simple literature-supported pairs (CPI-SSxxx IDs), after
    excluding heteromeric complexes
- name: RNA-Magnet
  role: Source of 50 ligand-receptor pairs merged into connectomeDB2020
- name: SingleCellSignalR
  role: Source of 22 pairs via its LRdb file
  repo: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- name: ICELLNET
  role: Source of 9 pairs from its Supplementary Table 1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:bf5467f1e0759f1fbc35742e826e32b652baa019d9e866bfbc8caba38f93c397
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
  - http://edamontology.org/topic_3068
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# literature-support-verification

## Summary

A curation skill for assembling or auditing literature-supported ligand-receptor databases (e.g., connectomeDB2020) by merging pair lists from prior resources (CellPhoneDB, RNA-Magnet, SingleCellSignalR, ICELLNET) and verifying that each cited PubMedID actually supports the claimed interaction, dropping unsupported entries (e.g., HPRD-derived pairs).

## When to use

Use when building or validating a curated ligand-receptor interaction database where each pair must carry primary literature support, or when auditing merged resources for pairs whose cited PMID does not actually back the interaction.

## When NOT to use

- You need computationally predicted interactions or downstream signalling consequences rather than literature-supported direct pairs (the authors point to NicheNet/SoptSC for signalling consequences and note NATMI does not model heteromeric complexes)
- The input is already a verified, deduplicated curated database and no PMID audit or merging of resources is required
- You need ligand-receptor pairs that include heteromeric complexes, which were deliberately excluded during extraction from CellPhoneDB

## Inputs

- CellPhoneDB v2.0 interaction list with internal IDs (CPI-SSxxx format), restricted to simple:simple pairs
- RNA-Magnet ligand-receptor pair list
- SingleCellSignalR LRdb file (LRdb_122019.txt)
- ICELLNET Supplementary Table 1
- Authors' 2015 ligand-receptor resource (Ramilowski et al.)
- PubMedIDs cited for each candidate ligand-receptor pair

## Outputs

- connectomeDB2020.csv: a two-column (Ligand, Receptor) table of 2293 literature-supported human ligand-receptor pairs
- Per-pair provenance annotations (source resource, PMID support status)
- Ligand class annotations (secreted vs plasma-membrane)

## How to apply

Collect the constituent source pair lists: CellPhoneDB v2.0 simple:simple interactions (internal IDs of format CPI-SSxxx, excluding heteromeric complexes), RNA-Magnet pairs, the SingleCellSignalR LRdb file (LRdb_122019.txt from github.com/SCA-IRCM/LRdb), and ICELLNET Supplementary Table 1. For each candidate pair, retrieve the cited PubMedID and read the cited paper to confirm it experimentally demonstrates the ligand-receptor interaction; drop entries where the PMID does not support the pair, which the NATMI authors found was common for HPRD-derived records. Merge the verified contributions (121 CellPhoneDB, 50 RNA-Magnet, 22 SingleCellSignalR, 9 ICELLNET pairs) with the authors' earlier 1751-pair 2015 resource and 340 new manually curated pairs, then deduplicate to reach the final set. Annotate each ligand as secreted or plasma-membrane, and write the result as a two-column (Ligand, Receptor) CSV in the 'lrdbs' folder so NATMI's ExtractEdges.py can consume it via the '--interDB' argument.

## Related tools

- **NATMI** (Consumer of the verified database: ExtractEdges.py ingests the curated two-column CSV placed in the 'lrdbs' folder via '--interDB') — https://github.com/forrest-lab/NATMI/
- **CellPhoneDB** (Source of 121 simple:simple literature-supported pairs (CPI-SSxxx IDs), after excluding heteromeric complexes)
- **RNA-Magnet** (Source of 50 ligand-receptor pairs merged into connectomeDB2020)
- **SingleCellSignalR** (Source of 22 pairs via its LRdb file) — https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- **ICELLNET** (Source of 9 pairs from its Supplementary Table 1)

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --interDB lrdbs/connectomeDB2020.csv
```

## Evaluation signals

- The output connectomeDB2020.csv contains exactly 2293 literature-supported human ligand-receptor pairs
- The file has the required 'Ligand' and 'Receptor' columns and parses as a two-column CSV
- Every retained pair has a PMID whose cited article demonstrably supports the interaction; HPRD-derived entries with non-supporting PMIDs were dropped
- Contributions reconcile with the expected composition: 1751 from the 2015 resource + 121 CellPhoneDB + 50 RNA-Magnet + 22 SingleCellSignalR + 9 ICELLNET + 340 new pairs (after deduplication)
- Each ligand is annotated as secreted or plasma-membrane and ExtractEdges.py can consume the file without format errors

## Limitations

- Pairs are only as reliable as manual curation; the authors found many HPRD-derived records had PubMedIDs that did not actually support the interaction, requiring individual PMID verification
- Heteromeric complexes are excluded (CellPhoneDB complex entries and HPRD complexes), and NATMI does not model heterodimerization or co-receptor requirements
- The database covers human interactions; non-human species use relies on NCBI HomoloGene homology matching of the human pairs

## Evidence

- [discussion] Exclusion of heteromeric complexes from CellPhoneDB interactions: [evidence span withheld — anchor status: not_found]
- [discussion] 22 pairs from SingleCellSignalR: "22 pairs from SingleCellSignalR20 (source: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt)"
- [discussion] HPRD PMID verification: "mostly from HPRD, where the PubMedID did not actually support the interaction"
- [abstract] connectomeDB2020 size: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [readme] Availability and purpose of curated lists: "NATMI and our updated ligand-receptor lists are freely available to the research community."
