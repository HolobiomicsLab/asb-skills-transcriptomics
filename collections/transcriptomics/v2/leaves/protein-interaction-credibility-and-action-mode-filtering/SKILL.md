---
name: protein-interaction-credibility-and-action-mode-filtering
description: Use when inferring ligand-receptor communication between two scRNA-seq
  cell clusters (e.g., vCAFs to malignant ICC cells) and you must remove biologically
  unlikely or low-confidence protein pairs before ranking by interaction power.
when_to_use_negative:
- The input is not scRNA-seq cluster-level expression but a precomputed interaction
  network or feature table — InterCellDB filtering requires gene expression per cluster
  plus protein localization annotations.
- The biological question concerns interactions other than ligand-receptor binding
  (e.g., receptor-receptor or vesicle-cell types the database also encodes) and 'binding'
  action mode between released factors and membrane receptors does not apply.
- Only predicted (neither experiment validated nor pathway curated) protein interactions
  are acceptable as evidence for the claim being made.
edam_operation: http://edamontology.org/operation_2949
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellDB
  role: R package providing the user-defined gene and interaction databases and the
    filtering/intercellular analysis functions that apply credibility, action-mode,
    and localization filters
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: R
  role: Environment in which InterCellDB data calculation and visualization are run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:33ccdc1649d7f8d86551ec5c0f069c3fa1ee621c0fc0a9a95006e1ccc0346036
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - InterCellDB
  - R
  edam_operation: http://edamontology.org/operation_2949
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# protein-interaction-credibility-and-action-mode-filtering

## Summary

A filtering strategy from InterCellDB that narrows candidate ligand-receptor gene pairs by interaction credibility (experiment validated or pathway curated STRING subsets, score >= 400 / medium confidence) and action mode/effect ('binding' with extracellular ligand to plasma-membrane receptor), then confirms significance via a cell-label permutation test (N = 100, p < 0.05).

## When to use

Use when inferring ligand-receptor communication between two scRNA-seq cell clusters (e.g., vCAFs to malignant ICC cells) and you must remove biologically unlikely or low-confidence protein pairs before ranking by interaction power.

## When NOT to use

- The input is not scRNA-seq cluster-level expression but a precomputed interaction network or feature table — InterCellDB filtering requires gene expression per cluster plus protein localization annotations.
- The biological question concerns interactions other than ligand-receptor binding (e.g., receptor-receptor or vesicle-cell types the database also encodes) and 'binding' action mode between released factors and membrane receptors does not apply.
- Only predicted (neither experiment validated nor pathway curated) protein interactions are acceptable as evidence for the claim being made.

## Inputs

- Normalized scRNA-seq expression matrix with annotated cell clusters (e.g., Seurat-processed GSE138709 human ICC data)
- InterCellDB human gene-coded protein database and protein-protein interaction database (figshare 10.6084/m9.figshare.17057339, release 0.9.2)
- Assignment of ligand-releasing (y-axis) and receptor-bearing (x-axis) cell clusters

## Outputs

- Ranked table of significant ligand-receptor pairs with sender cluster, receiver cluster, action mode, action effect, interaction power, and credibility score
- Permutation test p-values for each candidate gene pair
- Subset of interleukin ligand-receptor pairs (e.g., IL6-IL6R) passing the localization and credibility filters

## How to apply

Load the InterCellDB gene and interaction databases (human resources from figshare 10.6084/m9.figshare.17057339, release 0.9.2), which encode interaction properties (type, direction, effect) and gene-product properties (subcellular locations, molecular functions, biological processes). Restrict candidate protein pairs to the 'experiment validated' (STRING 'experiments > 0') or 'pathway curated' (STRING 'database > 0') subsets at medium confidence or above (combined score >= 400). Set action mode to 'binding' so factors released from ligand-releasing cells (y-axis, e.g., fibroblast/vCAF clusters) can bind receptors on receptor-bearing cells (x-axis, e.g., malignant ICC cells), and require ligands annotated to the extracellular region and receptors at the plasma membrane using COMPARTMENTS localization annotations. Filter out biologically unlikely interactions by applying the COMPARTMENTS confidence-score cut-off, which users can adjust to their experimental requirements. Compute interaction power as the product of normalized expression of the two proteins, then match statistically significant gene pairs (cell-label permutation test, N = 100 permutations, p < 0.05) back to the interaction database — only matched gene pairs are considered potential interactions between the two target cells. Rank surviving pairs by interaction power and credibility score, annotating sender cluster, receiver cluster, action mode, and action effect.

## Related tools

- **InterCellDB** (R package providing the user-defined gene and interaction databases and the filtering/intercellular analysis functions that apply credibility, action-mode, and localization filters) — https://github.com/ZJUDBlab/InterCellDB
- **R** (Environment in which InterCellDB data calculation and visualization are run)

## Examples

```
remove.packages("GO.db"); BiocManager::install("GO.db"); devtools::install_github("ZJUDBlab/InterCellDB")  # use release 0.9.2; then filter gene pairs with action mode = "binding" from the experiment validated/pathway curated subsets
```

## Evaluation signals

- The output table must contain the reported pairs IL6-IL6R, NOTCH3-JAG1, NOTCH3-JAG2, PDGFD-PDGFRB, and PGF-NRP2 with permutation p-value < 0.05.
- Every retained pair must have action mode 'binding' with ligands annotated to the extracellular region and receptors at the plasma membrane.
- All reported interactions must come from the experiment validated ('experiments > 0') or pathway curated ('database > 0') STRING subsets at medium confidence or above (score >= 400).
- Permutation test used N = 100 cell-label permutations and only pairs with p < 0.05 are kept.
- R code was executed with package-release version 0.9.2 of InterCellDB, as required by the tutorials.

## Limitations

- Cellular localization cut-off values are user-adjustable, so results are sensitive to the chosen COMPARTMENTS confidence threshold.
- Restricting to 'binding' action mode and validated/curated subsets excludes predicted interactions and other interaction types, potentially missing real communications.
- Interaction power is the product of normalized expression of the two proteins, so pairs with low but real expression may be under-ranked; significance depends on the permutation test (N = 100).
- The tutorials require InterCellDB package-release version 0.9.2, and installation requires removing and reinstalling GO.db from Bioconductor, which may break other R library setups.

## Evidence

- [methods] Action mode of protein pairs was set as "binding" to ensure that factors released from cells of the y‐axis could bind to their receptors in cells of the x‐axis.: [evidence span withheld — anchor status: not_found]
- [methods] The first subset contained those protein interactions that were validated by experiments. These protein interactions were labelled with "experiments > 0" in the STRING database.: [evidence span withheld — anchor status: not_found]
- [results] the statistically significant gene pairs were matched to the interaction database. Only matched gene pairs were considered as potential interactions between two target cells.: "the statistically significant gene pairs were matched to the interaction database. Only matched gene pairs were considered as potential interactions between two target cells."
- [results] To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database: "To filter out biologically unlikely interactions, we incorporated the cellular localization information of all gene‐encoded proteins from the COMPARTMENTS database"
- [results] permutation test is performed to calculate the confidence of interaction: "permutation test is performed to calculate the confidence of interaction"
- [readme] Please use **package-release version 0.9.2** to run all the codes.: "Please use **package-release version 0.9.2** to run all the codes."
