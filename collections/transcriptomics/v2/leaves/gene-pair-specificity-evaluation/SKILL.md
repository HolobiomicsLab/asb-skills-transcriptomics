---
name: gene-pair-specificity-evaluation
description: Use when a set of candidate ligand-receptor pairs has been inferred between
  two cell types and you must determine whether each pair is restricted to that sender-receiver
  combination or also appears in other cell-type interactions.
when_to_use_negative:
- When no prior cell-cell interaction inference has been run — specificity evaluation
  requires precomputed InterCellDB interaction results with confidence from the permutation
  framework, not raw expression matrices.
- When the goal is de novo discovery of all interacting pairs rather than assessing
  the restrictedness of a predefined candidate list — use InterCellDB's full network
  analysis instead.
- When pairs are expected to act via non-binding modes (e.g., predicted rather than
  experiment-validated interactions), since the specificity assessment relies on the
  interaction database built from STRING protein-protein interactions.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: InterCellDB
  role: Provides the intercellular analysis on specificity module that evaluates co-occurrences
    of gene pairs across interactions, plus the permutation-based confidence framework
    underlying it
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: R
  role: Environment in which InterCellDB data calculation and visualization are run;
    used to subset interaction tables and write specificity_table.csv
- name: Seurat
  role: Used upstream to process the GSE138709 scRNA-seq data (v2.3.4, 1000 variable
    genes, resolution 0.3) whose interaction results feed the specificity evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:b4b89debce4aae4b7ea5699159b18adf7a9a4700c8c2d1b182446dbbb7277915
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - InterCellDB
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-pair-specificity-evaluation

## Summary

Evaluates whether candidate ligand-receptor gene pairs (e.g., C3-Itgb2, C3-Itgam, C3-C5ar1, C3-C3ar1) are specific to one sender-receiver cell-type pair or participate in crosstalk elsewhere, using InterCellDB's intercellular analysis on specificity, which evaluates co-occurrences of gene pairs across interactions.

## When to use

Use when a set of candidate ligand-receptor pairs has been inferred between two cell types and you must determine whether each pair is restricted to that sender-receiver combination or also appears in other cell-type interactions.

## When NOT to use

- When no prior cell-cell interaction inference has been run — specificity evaluation requires precomputed InterCellDB interaction results with confidence from the permutation framework, not raw expression matrices.
- When the goal is de novo discovery of all interacting pairs rather than assessing the restrictedness of a predefined candidate list — use InterCellDB's full network analysis instead.
- When pairs are expected to act via non-binding modes (e.g., predicted rather than experiment-validated interactions), since the specificity assessment relies on the interaction database built from STRING protein-protein interactions.

## Inputs

- Prior InterCellDB interaction results (sender-receiver interaction table with significant gene pairs) computed on GSE138709 human ICC scRNA-seq data (Seurat v2.3.4, 1000 variable genes, resolution 0.3)
- List of candidate gene pairs to test (e.g., C3-Itgb2, C3-Itgam, C3-C5ar1, C3-C3ar1)
- Cell-type labels for sender and receiver clusters (e.g., CAF1 fibroblast, myeloid/macrophage)

## Outputs

- specificity_table.csv listing each gene pair, its restricted sender cell type, receiver cell type, and specificity status (4 rows, one per C3-containing pair)
- Per-pair co-occurrence assessment across all sender-receiver cell-type combinations

## How to apply

Load the prior InterCellDB interaction results computed on the scRNA-seq dataset of interest (here the human intrahepatic cholangiocarcinoma data from GEO accession GSE138709, processed with Seurat v2.3.4, 1000 variable genes, resolution 0.3) in R. Subset the interaction table to the target gene pairs (e.g., C3-Itgb2, C3-Itgam, C3-C5ar1, C3-C3ar1) and run the InterCellDB intercellular analysis on specificity, which evaluates co-occurrences of each gene pair across every sender-receiver cell-type combination, using the cell-label permutation framework (N = 100 permutations, significance at p-value < 0.05) already applied when interaction confidence was calculated. For each pair, record the restricted sender and receiver cell types — expected to be the CAF1 (fibroblast) sender to myeloid/macrophage receiver — and assemble a specificity table with pair, sender, receiver, and specificity status. Pairs assigned to a single sender-receiver combination consistent with the focal interaction (CAF1-myeloid) are considered relatively specific and good candidates for further biological study.

## Related tools

- **InterCellDB** (Provides the intercellular analysis on specificity module that evaluates co-occurrences of gene pairs across interactions, plus the permutation-based confidence framework underlying it) — https://github.com/ZJUDBlab/InterCellDB
- **R** (Environment in which InterCellDB data calculation and visualization are run; used to subset interaction tables and write specificity_table.csv)
- **Seurat** (Used upstream to process the GSE138709 scRNA-seq data (v2.3.4, 1000 variable genes, resolution 0.3) whose interaction results feed the specificity evaluation)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")  # then run specificity module with release 0.9.2 on GSE138709 interaction results
```

## Evaluation signals

- Output table specificity_table.csv exists and contains exactly 4 rows (one per C3-containing pair).
- Each pair is assigned to a single sender-receiver combination consistent with CAF1-myeloid restriction.
- Specificity co-occurrence computed across every sender-receiver cell-type combination, not just the focal pair.
- Permutation framework parameters match the original confidence calculation: N = 100 permutations, significance at p-value < 0.05.
- InterCellDB release version used is 0.9.2, as required to run the tutorial codes.

## Limitations

- Specificity is evaluated against prior interaction results; results depend on upstream parameters (Seurat v2.3.4, 1000 variable genes, resolution 0.3) and the permutation threshold (p < 0.05, N = 100).
- InterCellDB builds its interaction database from STRING protein-protein interactions with subsets (experiment validated, pathway curated, predicted); predicted interactions may be less reliable for specificity claims.
- No version information was stated for InterCellDB in the article itself; users must track package-release version 0.9.2 for reproducibility.

## Evidence

- [readme] Intercellular analysis on specificity, which evaluates co-occurences of gene pair across interactions: "Intercellular analysis on specificity, which evaluates co-occurences of gene pair across interactions"
- [results] permutation test to calculate confidence of interaction: "permutation test is performed to calculate the confidence of interaction"
- [results] statistically significant gene pairs matched to interaction database: "the statistically significant gene pairs were matched to the interaction database. Only matched gene pairs were considered as potential interactions between two target cells."
- [methods] three STRING-derived interaction subsets: "We named these three subsets as experiment validated, pathway curated, and predicted databases."
- [results] Data calculation and visualization run in R: "Data calculation and visualization are available in InterCellDB by running the R software."
