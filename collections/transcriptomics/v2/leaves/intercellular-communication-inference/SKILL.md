---
name: intercellular-communication-inference
description: Use when the goal is to infer and benchmark intercellular/ligand–receptor
  interactions between annotated cell clusters from scRNA-seq count matrices, especially
  with user-defined gene filters such as GO terms or subcellular location cutoffs.
when_to_use_negative:
- The question is about downstream signal transduction inside the receiving cell rather
  than intercellular interaction inference — the article notes 'the NicheNet offers
  a solution to predict the downstream signal transduction in the receiving cells'
- Only ligand–receptor interactions from a fixed built-in database are needed without
  user-defined gene/pair filters — other tools like CellPhoneDB may suffice; InterCellDB's
  value is user-defined analysis
- Input is bulk RNA-seq or already-aggregated expression without single-cell cluster
  annotations, since the analysis targets interactions between defined cell clusters
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellDB
  role: 'Primary R package: matches significant gene pairs to its user-defined interaction
    database and performs network/intercellular/spatial analyses'
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: Seurat
  role: Preprocessing of scRNA-seq counts (normalize, scale, cluster, cell-type annotation)
    before interaction inference
- name: CellPhoneDB
  role: Baseline tool benchmarked on the same processed dataset and identical gene
    filtering
- name: NicheNet
  role: Baseline tool benchmarked for predicted interaction accuracy comparison
- name: CellChat
  role: Baseline tool benchmarked for predicted interaction accuracy comparison
- name: iTALK
  role: Baseline tool in the mouse brain (GSE129788) performance comparison
- name: SingleCellSignalR
  role: Baseline tool in the mouse brain (GSE129788) performance comparison
- name: R
  role: Runtime environment; 'Data calculation and visualization are available in
    InterCellDB by running the R software'
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:59f922f50ab62dc7871bd182699caaa0cbb6260d84116244cd64bad26be1cfa9
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
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# intercellular-communication-inference

## Summary

Infer cell–cell communication from scRNA-seq data using InterCellDB, a user-defined R package that matches statistically significant gene pairs to a curated protein–protein interaction database annotated with cellular localization (COMPARTMENTS), functional classification (UniProt, GO), and interaction type/direction/effect.

## When to use

Use when the goal is to infer and benchmark intercellular/ligand–receptor interactions between annotated cell clusters from scRNA-seq count matrices, especially with user-defined gene filters such as GO terms or subcellular location cutoffs.

## When NOT to use

- The question is about downstream signal transduction inside the receiving cell rather than intercellular interaction inference — the article notes 'the NicheNet offers a solution to predict the downstream signal transduction in the receiving cells'
- Only ligand–receptor interactions from a fixed built-in database are needed without user-defined gene/pair filters — other tools like CellPhoneDB may suffice; InterCellDB's value is user-defined analysis
- Input is bulk RNA-seq or already-aggregated expression without single-cell cluster annotations, since the analysis targets interactions between defined cell clusters

## Inputs

- scRNA-seq raw gene expression count matrix (e.g., GEO GSE138709 human ICC, GSE129788 mouse brain, E-EHCA-2 mouse melanoma) with cell metadata
- Seurat-processed and cell-type-annotated expression object (normalized, scaled, clustered)
- Pre-built InterCellDB mouse/human gene and interaction databases (figshare 10.6084/m9.figshare.17057342 and 17057339)

## Outputs

- Table of predicted interacting protein/gene pairs per cluster pair with attributes (action mode, action effect, database subset)
- Per-tool accuracy/precision table (fraction of predicted pairs with literature support) for InterCellDB vs CellPhoneDB, NicheNet, CellChat (and iTALK, SingleCellSignalR)
- intercell_pairs_per_method.csv with columns: method, protein pair A/B, cluster i, cluster j, overlap flag

## How to apply

Preprocess raw counts (e.g., Davidson et al. mouse melanoma E-EHCA-2 or human ICC GSE138709) with Seurat v3.2: normalize, scale, cluster, and annotate cell types (e.g., CAF1-3, myeloid, cDC, pDC, NK, T, B, endothelial). Install InterCellDB release v0.9.2 (removing and re-installing GO.db from Bioconductor first, per the README) and run intercellular analysis on chosen cluster pairs (e.g., CAF1-to-myeloid), restricting to differentially expressed genes such as immune response GO: 0006955 and filtering protein pairs by action mode and effect (e.g., 'activation' + 'positive'). Match statistically significant gene pairs to the interaction database — which draws PPIs from STRING (subsets: experiment validated 'experiments > 0', pathway curated 'database > 0', and predicted) and filters biologically unlikely interactions via COMPARTMENTS localization scores with a user-adjustable cutoff. To benchmark, run CellPhoneDB, NicheNet, CellChat, iTALK, and SingleCellSignalR on the same dataset and identical protein-pair input, then compute per-tool accuracy as the fraction of predicted pairs with literature support (Davidson dataset: InterCellDB 86.6% vs 75% for the other three) or summarize average fetched pairs via Count_k = sum(Pair_i,j^(k))/(N*N).

## Related tools

- **InterCellDB** (Primary R package: matches significant gene pairs to its user-defined interaction database and performs network/intercellular/spatial analyses) — https://github.com/ZJUDBlab/InterCellDB
- **Seurat** (Preprocessing of scRNA-seq counts (normalize, scale, cluster, cell-type annotation) before interaction inference)
- **CellPhoneDB** (Baseline tool benchmarked on the same processed dataset and identical gene filtering)
- **NicheNet** (Baseline tool benchmarked for predicted interaction accuracy comparison)
- **CellChat** (Baseline tool benchmarked for predicted interaction accuracy comparison)
- **iTALK** (Baseline tool in the mouse brain (GSE129788) performance comparison)
- **SingleCellSignalR** (Baseline tool in the mouse brain (GSE129788) performance comparison)
- **R** (Runtime environment; 'Data calculation and visualization are available in InterCellDB by running the R software')

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")
```

## Evaluation signals

- On the Davidson mouse melanoma dataset, the InterCellDB accuracy reproduces the reported 86.6% benchmark (vs 75% for CellPhoneDB, NicheNet, CellChat)
- intercell_pairs_per_method.csv exists and contains one block of predicted pairs per method, with the method column containing all tool names
- Per-method fetched-pair counts are reproducible by recomputing Count_k = sum(Pair_i,j^(k))/(N*N) from the output table
- Output table contains an accuracy value for each of the four benchmarked tools (schema/field presence check)
- InterCellDB runs with package-release version 0.9.2 as required by the README tutorials

## Limitations

- Accuracy values depend on manual literature curation of predicted gene pairs (a pair counts as supported if experimentally validated for carcinogenesis or anti-tumor function), introducing expert-judgment subjectivity
- Users must fix the InterCellDB release (v0.9.2) and reinstall GO.db from Bioconductor during installation, or the provided tutorial code may fail
- The COMPARTMENTS cellular localization cut-off is user-adjustable, so filtering stringency — and hence the predicted pair set — varies with the chosen threshold

## Evidence

- [readme] InterCellDB is a tool for analyzing intercellular communication on single-cell RNA sequencing (scRNA-seq) data.: "InterCellDB is a tool for analyzing intercellular communication on single-cell RNA sequencing (scRNA-seq) data."
- [methods] InterCellDB is publicly accessible as an R package in GitHub.: "InterCellDB is publicly accessible as an R package in GitHub (https://github.com/ZJUDBlab/InterCellDB)."
- [results] Significant gene pairs matched to interaction database.: "the statistically significant gene pairs were matched to the interaction database. Only matched gene pairs were considered as potential interactions between two target cells."
- [methods] CAF1-myeloid activation/positive pair extraction.: [evidence span withheld — anchor status: not_found]
- [results] Davidson melanoma application.: [evidence span withheld — anchor status: not_found]
- [results] User-adjustable localization cutoff.: [evidence span withheld — anchor status: not_found]
- [readme] Version 0.9.2 requirement.: "Please use **package-release version 0.9.2** to run all the codes."
