---
name: secreted-protein-annotation
description: Use when building or extending a ligand-receptor database (e.g. connectomeDB2020-style
  two-column Ligand/Receptor CSV) where each ligand must be classified as secreted
  or plasma-membrane before edge extraction.
when_to_use_negative:
- When the interaction list already carries ligand class annotations (e.g. an existing
  NATMI-format lrdbs CSV) — re-annotating risks introducing inconsistencies
- When modelling heteromeric ligand or receptor complexes is required — NATMI explicitly
  does not model heteromeric complexes, so this simple secreted/plasma-membrane annotation
  scheme is the wrong fit
- For non-human species without homologue mapping — connectomeDB2020 is human; other
  species require NCBI HomoloGene homologue extraction first
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2229
tools:
- name: NATMI
  role: Consumes the annotated two-column ligand-receptor CSV via ExtractEdges.py
    '--interDB' to predict cell-to-cell communication networks
  repo: https://github.com/forrest-lab/NATMI/
- name: CellPhoneDB
  role: Source of 121 literature-supported simple:simple ligand-receptor pairs (internal
    IDs CPI-SSxxx, heteromeric complexes excluded) that are merged before ligand class
    annotation
- name: SingleCellSignalR
  role: Source of 22 ligand-receptor pairs via the LRdb_122019.txt file merged into
    the annotated database
  repo: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- name: ICELLNET
  role: Source of 9 ligand-receptor pairs from its Supplementary Table 1 merged into
    the annotated database
- name: RNA-Magnet
  role: Source of 50 ligand-receptor pairs merged into the annotated database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:bf85abdc6dcfafa6fd8fb43902e3c2edc8d523c9738a1b2eb7758b316ca89290
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# secreted-protein-annotation

## Summary

Annotates each ligand in a curated ligand-receptor interaction list (e.g. connectomeDB2020) as secreted or plasma-membrane, a curation step NATMI's authors performed while assembling the 2293-pair database so downstream communication networks distinguish soluble from cell-surface signalling.

## When to use

Use when building or extending a ligand-receptor database (e.g. connectomeDB2020-style two-column Ligand/Receptor CSV) where each ligand must be classified as secreted or plasma-membrane before edge extraction.

## When NOT to use

- When the interaction list already carries ligand class annotations (e.g. an existing NATMI-format lrdbs CSV) — re-annotating risks introducing inconsistencies
- When modelling heteromeric ligand or receptor complexes is required — NATMI explicitly does not model heteromeric complexes, so this simple secreted/plasma-membrane annotation scheme is the wrong fit
- For non-human species without homologue mapping — connectomeDB2020 is human; other species require NCBI HomoloGene homologue extraction first

## Inputs

- Merged, deduplicated list of literature-supported human ligand-receptor pairs (from the authors' 2015 resource, CellPhoneDB v2.0 CPI-SSxxx simple:simple pairs, RNA-Magnet, SingleCellSignalR LRdb_122019.txt, ICELLNET Supplementary Table 1, plus 340 newly curated pairs)
- Gene/protein identity information (e.g. HGNC symbols) for each ligand to determine secretion versus membrane tethering

## Outputs

- Two-column CSV (Ligand, Receptor) with each ligand annotated as secreted or plasma-membrane, stored in the 'lrdbs' folder (e.g. connectomeDB2020.csv with 2293 pairs) for consumption by ExtractEdges.py via '--interDB'

## How to apply

After merging ligand-receptor pairs from constituent resources (e.g. 1751 pairs from the authors' 2015 resource, 121 from CellPhoneDB v2.0 simple:simple pairs, 50 from RNA-Magnet, 22 from SingleCellSignalR LRdb, 9 from ICELLNET, plus 340 new manually curated pairs), classify each ligand entity by its mode of release: secreted (e.g. growth factors, cytokines) versus tethered to the plasma membrane. This annotation reflects the paper's modelling choice to represent ligand-receptor signalling without modelling heteromeric complexes, so each pair is reduced to a simple secreted or membrane-bound ligand interacting with a receptor. The annotated pairs are written as a two-column (Ligand, Receptor) CSV into the 'lrdbs' folder so ExtractEdges.py can consume it via '--interDB'. Validate the result by confirming connectomeDB2020.csv contains exactly 2293 literature-supported human ligand-receptor pairs with the required Ligand and Receptor columns.

## Related tools

- **NATMI** (Consumes the annotated two-column ligand-receptor CSV via ExtractEdges.py '--interDB' to predict cell-to-cell communication networks) — https://github.com/forrest-lab/NATMI/
- **CellPhoneDB** (Source of 121 literature-supported simple:simple ligand-receptor pairs (internal IDs CPI-SSxxx, heteromeric complexes excluded) that are merged before ligand class annotation)
- **SingleCellSignalR** (Source of 22 ligand-receptor pairs via the LRdb_122019.txt file merged into the annotated database) — https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
- **ICELLNET** (Source of 9 ligand-receptor pairs from its Supplementary Table 1 merged into the annotated database)
- **RNA-Magnet** (Source of 50 ligand-receptor pairs merged into the annotated database)

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --interDB lrdbs/connectomeDB2020.csv --species human
```

## Evaluation signals

- connectomeDB2020.csv contains exactly 2293 literature-supported human ligand-receptor pairs (1751 + 121 + 50 + 22 + 9 + 340 after deduplication)
- Output file has the required 'Ligand' and 'Receptor' columns and is a valid two-column CSV placed in the 'lrdbs' folder
- Every ligand carries a non-empty secreted/plasma-membrane class annotation with no unclassified entries
- Pairs where the cited PubMedID does not support the interaction (e.g. HPRD-derived entries flagged in the paper) have been dropped before annotation
- ExtractEdges.py runs successfully with the annotated file passed via '--interDB' and produces edge tables for a test expression matrix

## Limitations

- Ligands are reduced to simple binary interactions: heteromeric complexes, heterodimerization, and co-receptor requirements are not modelled
- Annotation quality depends on accurate verification of cited PubMedIDs — the authors found unsupported pairs 'mostly from HPRD, where the PubMedID did not actually support the interaction'
- The database is human-specific; cross-species use requires homologue mapping via the NCBI HomoloGene Database

## Evidence

- [abstract] annotated pairs feed connectomeDB2020: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] resource composition: "22 pairs from SingleCellSignalR20 (source: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt)"
- [discussion] CellPhoneDB extraction filter: [evidence span withheld — anchor status: not_found]
- [discussion] no heteromeric modelling: "thus we do not model them in NATMI"
- [discussion] PMID verification: "mostly from HPRD, where the PubMedID did not actually support the interaction"
- [readme] README database description: "NATMI uses connectomeDB2020 (the most up-to-date manually curated ligand-receptor interaction list) and user supplied gene expression tables with cell type labels"
