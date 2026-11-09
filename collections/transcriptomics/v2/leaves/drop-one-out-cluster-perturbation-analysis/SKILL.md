---
name: drop-one-out-cluster-perturbation-analysis
description: Use when you have per-patient ASGARD drug scores across annotated cell
  clusters and want to quantify how sensitive those scores are to each cell type by
  excluding one cluster at a time.
when_to_use_negative:
- The goal is to validate ASGARD's predictions against an external drug-response method
  (that is TRANSACT correlation, not cluster perturbation).
- The input is a bulk RNA-seq or expression matrix without annotated single-cell clusters,
  since the perturbation operates on individual cell types within the tumor microenvironment.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ASGARD
  role: Provides GetDrug and DrugScore functions to recompute per-patient standardized
    drug scores after each cell type is excluded
  repo: https://github.com/lanagarmire/ASGARD
- name: Seurat
  role: Preprocessing, Louvain clustering (resolution 0.4), and cell-type object management
    for the scRNA-seq cohorts
- name: SingleR
  role: Automatic annotation of single cells used to define the cell types that are
    dropped one at a time
- name: Limma
  role: Computes the per-cluster differential gene profiles (case vs. control, BH-adjusted)
    that serve as GetDrug input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:57d2e560fd9ab5e33346595eacb4872ae5f70d6a21be588fd1534ed597ce3c4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/ASGARD
  tools:
  - ASGARD
  - Limma
  - Seurat
  - SingleR
  - TRANSACT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# drop-one-out-cluster-perturbation-analysis

## Summary

An in silico perturbation procedure that removes each annotated cell type from a single-cell tumor microenvironment one at a time, recomputes ASGARD per-patient drug scores (DrugScore), and measures which cluster's exclusion most strongly and variably shifts the recommended drugs — used to assess cell-cluster influence on ASGARD recommendations in the GSE161529 TNBC cohort.

## When to use

Use when you have per-patient ASGARD drug scores across annotated cell clusters and want to quantify how sensitive those scores are to each cell type by excluding one cluster at a time.

## When NOT to use

- The goal is to validate ASGARD's predictions against an external drug-response method (that is TRANSACT correlation, not cluster perturbation).
- The input is a bulk RNA-seq or expression matrix without annotated single-cell clusters, since the perturbation operates on individual cell types within the tumor microenvironment.

## Inputs

- GSE161529 TNBC scRNA-seq count matrices (4 TNBC patients, 4 healthy controls)
- Seurat object with Louvain clusters (resolution 0.4) and SingleR cell-type annotations
- Per-cluster Limma differential gene profiles (case vs. control, BH-adjusted P values)
- Breast tissue DrugReference files: breast_rankMatrix.txt, breast_gene_info.txt, breast_drug_info.txt

## Outputs

- Per-drug ASGARD DrugScore table for each cell-type-excluded condition (cell type removed x drug score deltas)
- Heatmap of drop-one-out drug-score changes (cell type removed vs. drug)
- Identification of the most influential cell type (T cells in the TNBC cohort)

## How to apply

Preprocess the GSE161529 TNBC scRNA-seq data in Seurat (remove genes in fewer than 3 cells and cells with fewer than 200 unique genes), cluster with Louvain (resolution 0.4), and annotate cell types with SingleR against a reference single-cell panel. Compute per-cluster differential gene profiles per patient with Limma (case vs. control, BH-adjusted P values) and run the full ASGARD GetDrug against the breast tissue DrugReference with repurposing.unit='drug' and connectivity='negative', then call DrugScore to obtain baseline standardized drug scores across all clusters. Then, one annotated cell type at a time, remove that cluster, rerun GetDrug/DrugScore, and record the per-drug score changes relative to the baseline. Render the deltas as a heatmap (cell type removed vs. drug); in the TNBC validation, the T-cell-removed condition shows the largest and most variable drug-score changes, which is the expected direction of the finding. This rationale follows from ASGARD's patient-level drug score, which aggregates cluster-level efficacy, so dropping a cluster directly perturbs the standardized score.

## Related tools

- **ASGARD** (Provides GetDrug and DrugScore functions to recompute per-patient standardized drug scores after each cell type is excluded) — https://github.com/lanagarmire/ASGARD
- **Seurat** (Preprocessing, Louvain clustering (resolution 0.4), and cell-type object management for the scRNA-seq cohorts)
- **SingleR** (Automatic annotation of single cells used to define the cell types that are dropped one at a time)
- **Limma** (Computes the per-cluster differential gene profiles (case vs. control, BH-adjusted) that serve as GetDrug input)

## Examples

```
devtools::install_github("lanagarmire/Asgard"); Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list)
```

## Evaluation signals

- The output drop-one-out table contains finite per-drug score deltas for every annotated cell type and drug combination.
- The heatmap file exists and its rows/columns cover all annotated cell types removed vs. drugs scored.
- The T-cell-removed row/column exhibits the largest spread (variance/range) of drug-score deltas among all removed cell types.
- Baseline ASGARD drug scores (before any exclusion) match the standard DrugScore output with repurposing.unit='drug' and connectivity='negative' against the breast DrugReference.

## Limitations

- The drug reference is limited to 98 cell lines from the LINCS L1000 project, which may not represent the target disease's tissue.
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes feeding into GetDrug.
- Differential expression method choice affects downstream performance; Limma (Bayes) showed the best average AUC among the tested methods.

## Evidence

- [discussion] ASGARD defines a drug score at the individual patient level (Formula 1), which calculates the drug efficacy across all single-cell clusters: "ASGARD defines a drug score at the individual patient level (Formula 1), which calculates the drug efficacy across all single-cell clusters"
- [discussion] SingleR annotation: "we ran an automatic annotation of single cells based on similarity to the referenced single-cell panel using the SingleR package75"
- [methods] DrugScore method call: "Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list,"
- [readme] ASGARD install command: "devtools::install_github("lanagarmire/Asgard")"
- [discussion] Sparsity caveat: "Sparsity and heterogeneity are two major challenges in analyzing single-cell data, and usually cause false discoveries of differentially expressed genes"
