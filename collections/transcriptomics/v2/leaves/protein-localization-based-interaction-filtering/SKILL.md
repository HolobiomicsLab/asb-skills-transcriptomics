---
name: protein-localization-based-interaction-filtering
description: Use when constructing or curating a ligand–receptor prior database for
  cell–cell communication inference and you must restrict candidate pairs to biophysically
  plausible secreted/membrane proteins using GO compartment annotations.
when_to_use_negative:
- The user supplies their own curated L-R gene list — dsCellNet supports supplementary
  user-provided L-R genes instead of relying on the default localization-filtered
  database
- The analysis requires receptor localization beyond the plasma membrane (e.g. intracellular/nuclear
  receptors), which this GO:0005886-based filter drops by design
- Inferring interactions in a single stationary snapshot without needing a localization-filtered
  prior — though the filter still applies inside dsCellNet, other methods' pair lists
  may be used directly
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2229
tools:
- name: dsCellNet
  role: Hosts the preparepairs filtering step that merges the four published L-R sources
    and applies localization filtering via dbloc to build the default database
  repo: https://github.com/songzh523/dsCellNet
- name: homologene
  role: Maps gene identifiers to a consistent species namespace using default human/mouse
    homologs before/while building the pair list
- name: CellPhoneDB
  role: Source of 1,396 published L-R pairs merged into the default database
- name: NATMI
  role: Source of 4,071 published L-R pairs merged into the default database
- name: SingleCellSignalR
  role: Source of 3,251 published L-R pairs merged into the default database
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:c81ff01024966f1e48ddae9c7505de488b8af913a9a4a743c121aff90e71e50c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  repo_url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# protein-localization-based-interaction-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds and filters a multi-species ligand–receptor database by subcellular localization: merge published L-R sources (Ramilowski 2015, CellPhoneDB, NATMI, SingleCellSignalR), then keep only pairs whose ligand is secreted or plasma-membrane and whose receptor is plasma-membrane, yielding dsCellNet's default list of 5,744 interactions.

## When to use

Use when constructing or curating a ligand–receptor prior database for cell–cell communication inference and you must restrict candidate pairs to biophysically plausible secreted/membrane proteins using GO compartment annotations.

## When NOT to use

- The user supplies their own curated L-R gene list — dsCellNet supports supplementary user-provided L-R genes instead of relying on the default localization-filtered database
- The analysis requires receptor localization beyond the plasma membrane (e.g. intracellular/nuclear receptors), which this GO:0005886-based filter drops by design
- Inferring interactions in a single stationary snapshot without needing a localization-filtered prior — though the filter still applies inside dsCellNet, other methods' pair lists may be used directly

## Inputs

- Published L-R pair lists: Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396 pairs), NATMI (4,071 pairs), SingleCellSignalR (3,251 pairs)
- Public protein localization annotations (GO Cellular Compartment terms GO:0005576, GO:0005615, GO:0009986, GO:0005886)
- Species name for homologene gene-identifier mapping (default human/mouse homologs)

## Outputs

- Default multi-species supported membrane-or-secreted L-R database table (default_lr_pairs.csv) with 5,744 retained interactions

## How to apply

Load the four published L-R pair lists incorporated by dsCellNet: Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396 pairs), NATMI (4,071 pairs), and SingleCellSignalR (3,251 pairs). Map gene identifiers to a consistent species namespace using the homologene package with default human/mouse homologs, as used by the preparepairs step against dbloc. Annotate every ligand and receptor gene with GO Cellular Compartment terms — GO:0005576 (extracellular region), GO:0005615 (extracellular space), GO:0009986 (cell surface), and GO:0005886 (plasma membrane). Filter genes: keep ligands annotated as secreted (GO:0005576/GO:0005615) or plasma-membrane (GO:0005886) and keep only receptors annotated at the plasma membrane (GO:0005886); drop all other pairs. Merge the filtered pair lists, remove duplicates across sources, and write the default membrane-or-secreted L-R database table (default_lr_pairs.csv); the rationale is that interactions must be annotated at protein localization, distinguishing membrane-ligand vs secreted-ligand mediated receptor interactions that published methods conflate.

## Related tools

- **dsCellNet** (Hosts the preparepairs filtering step that merges the four published L-R sources and applies localization filtering via dbloc to build the default database) — https://github.com/songzh523/dsCellNet
- **homologene** (Maps gene identifiers to a consistent species namespace using default human/mouse homologs before/while building the pair list)
- **CellPhoneDB** (Source of 1,396 published L-R pairs merged into the default database)
- **NATMI** (Source of 4,071 published L-R pairs merged into the default database)
- **SingleCellSignalR** (Source of 3,251 published L-R pairs merged into the default database)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab)
```

## Evaluation signals

- The retained default L-R pair list contains exactly 5,744 interactions, matching the count reported in the paper (note the tools-comparison table lists 5,684, so cross-check which count is expected)
- Every retained receptor gene is annotated GO:0005886 (plasma membrane); every retained ligand is annotated GO:0005576/GO:0005615 (secreted) or GO:0005886 (membrane)
- No duplicate L-R pairs remain after merging across the four source databases
- Gene identifiers are successfully mapped to the target species via homologene (human/mouse homologs) before filtering
- The output table default_lr_pairs.csv is written to dbloc and consumed downstream by singlenet without errors

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g., MG–Neurons); the cause of missing MG–Neurons interactions in individual samples remains unexplained, and users can support their L-R genes as supplementary
- Published methods cannot distinguish membrane-ligand vs secreted-ligand mediated receptor interactions given their different expression profiles — the filter addresses but depends on complete GO annotation quality
- The count discrepancy between the results text (5,744) and the comparison table (5,684) means the expected output size must be verified against the actual build

## Evidence
