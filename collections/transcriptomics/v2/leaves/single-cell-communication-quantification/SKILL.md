---
name: single-cell-communication-quantification
description: Use when you have scRNA-seq expression, cell-type labels, and (optionally)
  time-point annotations and need to infer and compare cell-cell interaction networks,
  especially to test how L-R aggregation schemes affect interaction recovery.
when_to_use_negative:
- Data lacks reliable cell-type labels or protein-localization information is irrelevant
  (e.g., purely intracellular gene programs) — the localization-aware scoring is the
  method's core rationale
- Analysis goal is temporal dynamics across developmental series rather than a single-time-point
  interaction map — use dsCellNet's developcorrelation/DTW workflow instead of singlenet
  alone
- The L-R pairs of interest are absent from the default 5,744-pair database and no
  supplementary L-R list is supplied — the default list size can cause missed interactions
  (e.g., MG–Neurons)
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: dsCellNet
  role: Primary R package implementing datainput, preparepairs, singlenet, and comparednet
    for L-R interaction inference and variant comparison
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps human L-R pairs to mouse homologs when building the candidate pair list
- name: CellPhoneDB
  role: Source of 1,396 L-R pairs incorporated into the default dsCellNet database
- name: NATMI
  role: Source of 4,071 L-R pairs incorporated into the default dsCellNet database
- name: SingleCellSignalR
  role: Source of 3,251 L-R pairs incorporated into the default dsCellNet database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:81a0462cee14237bd541b6189ef7cc97d103091d94b84e441824c320eaa7385b
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
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-communication-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies ligand-receptor-mediated cell-cell interactions from single-cell RNA-seq using dsCellNet, which scores membrane- and secreted-ligand interactions separately against protein localization, rather than a single product-of-sum or product-of-mean aggregation of L-R expression.

## When to use

Use when you have scRNA-seq expression, cell-type labels, and (optionally) time-point annotations and need to infer and compare cell-cell interaction networks, especially to test how L-R aggregation schemes affect interaction recovery.

## When NOT to use

- Data lacks reliable cell-type labels or protein-localization information is irrelevant (e.g., purely intracellular gene programs) — the localization-aware scoring is the method's core rationale
- Analysis goal is temporal dynamics across developmental series rather than a single-time-point interaction map — use dsCellNet's developcorrelation/DTW workflow instead of singlenet alone
- The L-R pairs of interest are absent from the default 5,744-pair database and no supplementary L-R list is supplied — the default list size can cause missed interactions (e.g., MG–Neurons)

## Inputs

- Single-cell gene expression matrix (e.g., 11,670 E15.5 mouse embryonic cerebral cortex cells from GSE153164)
- Cell-type label file per cell
- Time-point annotation file (optional; required for developmental-series analysis)
- L-R database file path (dbloc) pointing to the default dsCellNet pair list
- Species name for homologene mapping

## Outputs

- Inferred cell-cell interaction network per cell-type pair per time point
- Interactions split by category (membrane-ligand vs secreted-ligand mediated)
- Comparison table of interaction counts across aggregation variants (e.g., variant_interaction_counts.csv)
- Incoming/outgoing/total importance scores per cell type via importance_plot

## How to apply

Load the expression matrix, cell-type file, and time-point file with dsCellNet's datainput(datloc, expnam, typnam, timnam) in R (>=4.0.3). Build membrane-/secreted-ligand and receptor candidate pairs with preparepairs against the default dsCellNet L-R database (5,744 pairs compiled from Ramilowski et al. 2,557, CellPhoneDB 1,396, NATMI 4,071, and SingleCellSignalR 3,251, homologene-mapped). Run singlenet under each aggregation variant — product-of-sum, product-of-mean, and dsCellNet's regularized localization-aware scoring, which computes significance of membrane vs secreted L-R genes separately and reinforces interactions via dynamic time warping. Tabulate inferred interactions per cell-type pair and split by L-R category using comparednet on the control time point (e.g., E15.5 from GSE153164). Expect the product-of-sum variant to yield the fewest interactions and miss rare cell types (endothelial cells, OPCs), while regularized dsCellNet recovers more secreted-ligand/membrane-receptor interactions than product-of-mean.

## Related tools

- **dsCellNet** (Primary R package implementing datainput, preparepairs, singlenet, and comparednet for L-R interaction inference and variant comparison) — github.com/songzh523/dsCellNet
- **homologene** (Maps human L-R pairs to mouse homologs when building the candidate pair list)
- **CellPhoneDB** (Source of 1,396 L-R pairs incorporated into the default dsCellNet database)
- **NATMI** (Source of 4,071 L-R pairs incorporated into the default dsCellNet database)
- **SingleCellSignalR** (Source of 3,251 L-R pairs incorporated into the default dsCellNet database)

## Examples

```
memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- Product-of-sum variant should recover the fewest interactions and miss rare cell types such as endothelial cells and OPCs
- Regularized dsCellNet should capture more secreted-ligand/membrane-receptor interactions than the product-of-mean variant
- The default L-R database should contain 5,744 interactions after homologene mapping; preparepairs output should be consistent with this
- Counts per cell-type pair, split by membrane- vs secreted-ligand category, should reproduce the Fig. 3A/3C reference pattern on E15.5
- comparednet output on the control time point should yield a tabulated interaction count table (e.g., variant_interaction_counts.csv) with nonzero entries for all expected cell-type pairs

## Limitations

- The default L-R gene list size limits detection of some interactions (e.g., MG–Neurons); users can supply supplementary L-R genes
- The cause of missing MG–Neurons interactions in individual samples remains unexplained
- Published aggregation schemes cannot distinguish membrane-ligand vs secreted-ligand mediated receptor interactions given their different expression profiles — a reason the localization-aware variant exists
- dsCellNet version number is not stated; README references dsCellNetv1_1.0.tar.gz and requires R >= 4.0.3 with specific dependency versions

## Evidence
