---
name: go-cellular-compartment-annotation-filtering
description: Use when building or curating a ligand-receptor database from published
  L-R sources and you need to verify subcellular localization (secreted vs membrane)
  before accepting a pair.
when_to_use_negative:
- When the user supplies a custom L-R gene list to supplement the default database
  — the article notes the default list size can miss some interactions (e.g., MG–Neurons)
  and 'users can support their L-R genes as supplementary', bypassing this filtering.
- When the analysis goal is cell-cell communication inference from expression data
  rather than database construction — use dsCellNet's singlenet/developcorrelation
  steps instead.
edam_operation: http://edamontology.org/operation_3559
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0602
tools:
- name: dsCellNet
  role: Package whose preparepairs step consumes the GO-localization-filtered L-R
    database; the default list is built by this filtering procedure
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps L-R pair gene identifiers to default human/mouse homologs before GO annotation
    and filtering
- name: CellPhoneDB
  role: Source of 1,396 published L-R pairs merged into the default database prior
    to localization filtering
- name: NATMI
  role: Source of 4,071 published L-R pairs merged into the default database prior
    to localization filtering
- name: SingleCellSignalR
  role: Source of 3,251 published L-R pairs merged into the default database prior
    to localization filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:feb888cc4a55c541ffeb3047745dbcd948c2f5be3761b65e6e183eac4238a8c4
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
  edam_operation: http://edamontology.org/operation_3559
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# go-cellular-compartment-annotation-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates candidate ligand and receptor genes with GO Cellular Compartment terms (extracellular region, extracellular space, cell surface, plasma membrane) and filters L-R pairs so that only secreted or membrane ligands paired with plasma-membrane receptors are retained, yielding dsCellNet's default multi-species L-R database of 5,744 interactions.

## When to use

Use when building or curating a ligand-receptor database from published L-R sources and you need to verify subcellular localization (secreted vs membrane) before accepting a pair.

## When NOT to use

- When the user supplies a custom L-R gene list to supplement the default database — the article notes the default list size can miss some interactions (e.g., MG–Neurons) and 'users can support their L-R genes as supplementary', bypassing this filtering.
- When the analysis goal is cell-cell communication inference from expression data rather than database construction — use dsCellNet's singlenet/developcorrelation steps instead.

## Inputs

- Published L-R pair lists: Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396 pairs), NATMI (4,071 pairs), SingleCellSignalR (3,251 pairs)
- Gene identifier mappings across species (human/mouse homologs via homologene)
- GO Cellular Compartment annotations for ligand and receptor genes

## Outputs

- default_lr_pairs.csv: merged, deduplicated multi-species supported membrane-or-secreted L-R pair database (5,744 interactions)
- Per-gene subcellular localization annotation (secreted vs plasma-membrane) used downstream by preparepairs

## How to apply

Merge published L-R pair lists — Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396 pairs), NATMI (4,071 pairs), SingleCellSignalR (3,251 pairs) — and map gene identifiers to a consistent species namespace with the homologene package (default human/mouse homologs). Annotate every ligand and receptor gene with GO Cellular Compartment terms: GO:0005576 (extracellular region), GO:0005615 (extracellular space), GO:0009986 (cell surface), and GO:0005886 (plasma membrane). Keep ligands annotated as secreted (GO:0005576/GO:0005615) or plasma-membrane (GO:0005886), and keep only receptors annotated at the plasma membrane (GO:0005886); drop all other pairs. Merge the filtered lists, remove duplicates across sources, and write the default multi-species supported membrane-or-secreted L-R table (default_lr_pairs.csv). This localization filtering is the basis of dsCellNet's design that it 'identifies interactions according to protein localizations'.

## Related tools

- **dsCellNet** (Package whose preparepairs step consumes the GO-localization-filtered L-R database; the default list is built by this filtering procedure) — github.com/songzh523/dsCellNet
- **homologene** (Maps L-R pair gene identifiers to default human/mouse homologs before GO annotation and filtering)
- **CellPhoneDB** (Source of 1,396 published L-R pairs merged into the default database prior to localization filtering)
- **NATMI** (Source of 4,071 published L-R pairs merged into the default database prior to localization filtering)
- **SingleCellSignalR** (Source of 3,251 published L-R pairs merged into the default database prior to localization filtering)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab)
```

## Evaluation signals

- The retained default L-R pair list contains exactly 5,744 interactions, matching the count reported in the paper ('In total, 5,744 interactions were kept for the default list of L-R pairs.').
- Every ligand in the output is annotated with GO:0005576, GO:0005615, or GO:0005886; every receptor is annotated with GO:0005886 (plasma membrane).
- No duplicate L-R pairs remain after merging the four source databases.
- The output file default_lr_pairs.csv is produced and consumed successfully by dsCellNet's preparepairs step (memsercpros <- preparepairs(inpexp, species, dbloc, mainlab)).
- Gene identifiers in the final table resolve consistently under the homologene human/mouse homolog mapping.

## Limitations

- The size of the default L-R gene list can limit detection of some interactions (e.g., MG–Neurons); users can supply supplementary L-R genes.
- Localization filtering depends on GO Cellular Compartment annotation coverage; poorly annotated genes will be dropped from the default list.
- The paper's tools comparison table lists the LR database size as 5,684, which differs from the 5,744 count reported in the results text — verify which figure applies to the dsCellNet version in use.
- Cross-species mapping relies on default human/mouse homologs from homologene, which may miss one-to-many orthologs.

## Evidence
