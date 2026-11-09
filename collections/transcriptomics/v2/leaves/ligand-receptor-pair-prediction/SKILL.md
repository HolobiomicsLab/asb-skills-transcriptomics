---
name: ligand-receptor-pair-prediction
description: Use when you have cluster-annotated scRNA-seq data (human or mouse) and
  need to infer candidate protein pairs mediating communication between two specific
  cell types, e.g. CAF1–myeloid interactions in melanoma.
when_to_use_negative:
- When the goal is downstream signal transduction prediction inside the receiver cells
  rather than intercellular pair prediction — NicheNet is designed for that purpose
- When genes lack subcellular-localization or GO annotation, since the COMPARTMENTS/UniProt/GO
  filtering is central to excluding biologically unlikely interactions
- When you need interactions for a non-human/non-mouse organism — InterCellDB covers
  only human (18,990 genes) and mouse (20,938 genes)
tools:
- name: InterCellDB
  role: Primary R package that matches significant gene pairs to its user-defined
    interaction database, applies localization/action-mode/effect filters, and runs
    permutation-based confidence and specificity modules
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: CellPhoneDB
  role: Baseline tool whose predicted pairs are compared against InterCellDB candidates
    for overlap assessment
- name: NicheNet
  role: Baseline tool for overlap comparison of predicted pairs on shared datasets
- name: CellChat
  role: Baseline tool for overlap comparison of predicted pairs on shared datasets
- name: Seurat
  role: Preprocessing of scRNA-seq data (clustering, variable genes, cell annotation)
    before pair prediction
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:f4c8efa0326b105eccd99710ab897b46f4fed823c95ade80882ebb6809fa2943
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - CellChat
  - CellPhoneDB
  - InterCellDB
  - NicheNet
  - R
  - Seurat
  - SingleCellSignalR
  - iTALK
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# ligand-receptor-pair-prediction

## Summary

Predict candidate intercellular protein pairs (ligand–receptor, receptor–receptor, ECM–receptor) between two scRNA-seq cell clusters using InterCellDB: statistically significant differentially expressed gene pairs are matched against a curated protein–protein interaction database annotated with subcellular localization (COMPARTMENTS), action mode, effect, and GO functional classification.

## When to use

Use when you have cluster-annotated scRNA-seq data (human or mouse) and need to infer candidate protein pairs mediating communication between two specific cell types, e.g. CAF1–myeloid interactions in melanoma.

## When NOT to use

- When the goal is downstream signal transduction prediction inside the receiver cells rather than intercellular pair prediction — NicheNet is designed for that purpose
- When genes lack subcellular-localization or GO annotation, since the COMPARTMENTS/UniProt/GO filtering is central to excluding biologically unlikely interactions
- When you need interactions for a non-human/non-mouse organism — InterCellDB covers only human (18,990 genes) and mouse (20,938 genes)

## Inputs

- Cluster-annotated scRNA-seq count matrix (Seurat object), e.g. Davidson et al. mouse melanoma, Ximerakis et al. mouse brain GSE129788, or Zhang et al. human ICC GSE138709
- InterCellDB interaction database (mouse or human, built from STRING with COMPARTMENTS localization and UniProt/GO annotations)
- Differentially expressed gene lists per cell cluster with log2 fold changes
- Sender and receiver cell-type labels (e.g. CAF1 vs myeloid/macrophage)

## Outputs

- Table of candidate ligand-receptor protein pairs between the two target clusters (e.g. 15 pairs for CAF1–myeloid)
- Per-pair method-membership / overlap table (lr_pair_overlap_matrix.csv)
- Pair specificity table assigning each pair to a single sender-receiver combination (specificity_table.csv)
- Per-method predicted-pair table with overlap flags (intercell_pairs_per_method.csv)

## How to apply

First identify differentially expressed genes per cluster and retain pairs with log2 fold change > 0.25 on both sides; optionally restrict genes to a function of interest, such as immune response genes (GO: 0006955). Match the statistically significant gene pairs against the InterCellDB interaction database (built from STRING protein–protein interactions, with subsets 'experiment validated', 'pathway curated', and 'predicted'), so that only matched pairs are considered potential interactions. Filter biologically unlikely interactions using COMPARTMENTS cellular localization cut-offs (user-adjustable) and restrict action mode to 'binding' with 'activation'/positive effect when a ligand from sender cells must reach receptors on receiver cells. Confidence is estimated by a permutation test (default N = 100 permutations, significance at p < 0.05), and the specificity module evaluates co-occurrence of each pair across all sender–receiver cell-type combinations. Run InterCellDB (R package, release v0.9.2) and write candidate pairs and their attributes with Tool.WriteTables, e.g. to lr_pair_overlap_matrix.csv or specificity_table.csv.

## Related tools

- **InterCellDB** (Primary R package that matches significant gene pairs to its user-defined interaction database, applies localization/action-mode/effect filters, and runs permutation-based confidence and specificity modules) — https://github.com/ZJUDBlab/InterCellDB
- **CellPhoneDB** (Baseline tool whose predicted pairs are compared against InterCellDB candidates for overlap assessment)
- **NicheNet** (Baseline tool for overlap comparison of predicted pairs on shared datasets)
- **CellChat** (Baseline tool for overlap comparison of predicted pairs on shared datasets)
- **Seurat** (Preprocessing of scRNA-seq data (clustering, variable genes, cell annotation) before pair prediction)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")
```

## Evaluation signals

- Output CSV (e.g. lr_pair_overlap_matrix.csv) exists and contains the expected candidate pairs — e.g. exactly the 15 InterCellDB candidate ligand-receptor pairs for CAF1–myeloid
- Overlap counts are consistent with the published result: 9 of 15 InterCellDB pairs (60%) shared with CellPhoneDB, NicheNet, and CellChat
- Restricting to 'binding' action mode with 'activation'/positive effect returns pairs in which the sender-cell factor can reach the receiver-cell receptor
- Permutation-based significance holds at p < 0.05 across N = 100 cell-label permutations
- Specificity table has exactly one sender-receiver assignment per pair (e.g. 4 rows for C3–Itgb2, C3–Itgam, C3–C5ar1, C3–C3ar1, all CAF1→myeloid)

## Limitations

- Filtering stringency depends on the user-adjustable COMPARTMENTS localization cut-off value, so results vary with chosen thresholds
- Prediction relies on the STRING-derived interaction database; pairs absent from STRING (even if biologically real) cannot be recovered
- No version information is stated for some comparison tools (e.g. NicheNet), complicating exact reproduction of overlap analyses
- Package-release version 0.9.2 must be used to run the published code; other releases may not be compatible

## Evidence

- [results] Match significant gene pairs to interaction database: "the statistically significant gene pairs were matched to the interaction database. Only matched gene pairs were considered as potential interactions between two target cells."
- [methods] Binding action-mode restriction: [evidence span withheld — anchor status: not_found]
- [methods] Database subsets: "We named these three subsets as experiment validated, pathway curated, and predicted databases."
- [results] Localization filtering: "To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database"
- [methods] R package availability: "InterCellDB is publicly accessible as an R package in GitHub (https://github.com/ZJUDBlab/InterCellDB)."
- [readme] Version pinning: "Please use **package-release version 0.9.2** to run all the codes."
