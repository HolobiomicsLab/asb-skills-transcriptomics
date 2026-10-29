---
name: cross-method-drug-sensitivity-correlation
description: Use when validating a single-cell drug repurposing pipeline against an
  independent bulk drug-response predictor (e.g., TRANSACT vs. ASGARD) or when quantifying
  how removing one annotated cell type changes per-patient drug scores.
when_to_use_negative:
- The goal is bulk-RNA-seq drug repurposing without single-cell cluster structure
  — ASGARD's multi-cluster DrugScore and per-cluster Limma DEGs require scRNA-seq
  with annotated cell types.
- No normal-tissue control samples are available for computing case-vs-control differential
  expression — the article notes obtaining proper normal controls may not be feasible
  for some diseases.
- The drug of interest has no LINCS L1000 reference profile and no GDSC response measurement,
  so neither ASGARD scoring nor the TRANSACT correlation subset applies.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0154
tools:
- name: ASGARD
  role: Computes per-cluster drug repurposing (GetDrug) and per-patient standardized
    multi-cluster drug scores (DrugScore) against the breast DrugReference; installed
    via devtools::install_github("lanagarmire/Asgard")
  repo: https://github.com/lanagarmire/ASGARD
- name: TRANSACT
  role: Independent drug-response prediction method used to validate ASGARD by correlating
    its predicted drug sensitivities with ASGARD drug scores on GDSC-subset drugs
- name: Seurat
  role: Preprocesses GSE161529 scRNA-seq (CreateSeuratObject with min.cells=3, min.features=200),
    Louvain clustering at resolution 0.4 for cell-type discovery
- name: Limma
  role: Computes per-cluster differential gene profiles (case vs. control with BH-adjusted
    P values) used as GetDrug input
- name: SingleR
  role: Automatically annotates Seurat clusters with cell types against a reference
    single-cell panel, defining the cell types used in the drop-one-out analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:d21210184f768183f2811cf2a2dd524c062a7b7f3d6e4920983fe18ebdcf621c
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-method-drug-sensitivity-correlation

## Summary

Reproduces the ASGARD paper's TNBC validation: correlating per-patient ASGARD standardized drug scores with TRANSACT-predicted drug sensitivities on the GDSC-overlapping drug subset, and running an in silico drop-one-out analysis of tumor microenvironment cell types to test drug-score robustness.

## When to use

Use when validating a single-cell drug repurposing pipeline against an independent bulk drug-response predictor (e.g., TRANSACT vs. ASGARD) or when quantifying how removing one annotated cell type changes per-patient drug scores.

## When NOT to use

- The goal is bulk-RNA-seq drug repurposing without single-cell cluster structure — ASGARD's multi-cluster DrugScore and per-cluster Limma DEGs require scRNA-seq with annotated cell types.
- No normal-tissue control samples are available for computing case-vs-control differential expression — the article notes obtaining proper normal controls may not be feasible for some diseases.
- The drug of interest has no LINCS L1000 reference profile and no GDSC response measurement, so neither ASGARD scoring nor the TRANSACT correlation subset applies.

## Inputs

- GSE161529 TNBC scRNA-seq count matrices (4 TNBC patients, 4 healthy controls)
- Breast tissue DrugReference files: breast_rankMatrix.txt, breast_gene_info.txt, breast_drug_info.txt (built from LINCS L1000 GSE70138/GSE92742 via PrepareReference)
- GDSC drug response measurements (for restricting the TRANSACT-vs-ASGARD comparison drug set)
- Seurat-annotated cell metadata (cell type labels per cluster)

## Outputs

- Per-patient standardized ASGARD drug scores across all clusters (DrugScore output)
- TRANSACT-predicted drug sensitivities per patient
- Correlation statistics table (Pearson/Spearman coefficients and P values) on the GDSC-subset drugs
- Heatmap of drop-one-out drug-score changes (cell type removed vs. drug)

## How to apply

Download and preprocess GSE161529 TNBC scRNA-seq data (4 patients, 4 healthy controls) in Seurat with min.cells=3 and min.features=200, cluster via Louvain at resolution 0.4, and annotate cell types with SingleR. Compute per-cluster differential expression per patient with Limma (case vs. control, BH-adjusted P values) and feed the cluster DEG lists to ASGARD GetDrug using the breast tissue DrugReference (breast_rankMatrix.txt, breast_gene_info.txt, breast_drug_info.txt) with repurposing.unit='drug' and connectivity='negative', then call DrugScore for per-patient standardized drug scores. Run TRANSACT on the same cohort, restrict to drugs with GDSC response measurements, and compute Pearson/Spearman correlations (with P values) between ASGARD drug scores and TRANSACT drug sensitivities. Finally, for the drop-one-out analysis, remove each annotated cell type (e.g., T cells, B cells, macrophages) in turn, recompute DrugScore, record per-drug score deltas, and render a heatmap of cell-type-removed vs. drug to identify the most influential cell type.

## Related tools

- **ASGARD** (Computes per-cluster drug repurposing (GetDrug) and per-patient standardized multi-cluster drug scores (DrugScore) against the breast DrugReference; installed via devtools::install_github("lanagarmire/Asgard")) — https://github.com/lanagarmire/ASGARD
- **TRANSACT** (Independent drug-response prediction method used to validate ASGARD by correlating its predicted drug sensitivities with ASGARD drug scores on GDSC-subset drugs)
- **Seurat** (Preprocesses GSE161529 scRNA-seq (CreateSeuratObject with min.cells=3, min.features=200), Louvain clustering at resolution 0.4 for cell-type discovery)
- **Limma** (Computes per-cluster differential gene profiles (case vs. control with BH-adjusted P values) used as GetDrug input)
- **SingleR** (Automatically annotates Seurat clusters with cell types against a reference single-cell panel, defining the cell types used in the drop-one-out analysis)

## Examples

```
devtools::install_github("lanagarmire/Asgard"); Drug.ident.res <- GetDrug(gene.data = Gene.list, drug.ref.profiles = GetDrugRef(drug.response.path = 'DrugReference/breast_rankMatrix.txt'), repurposing.unit = 'drug', connectivity = 'negative')
```

## Evaluation signals

- The correlation table contains finite coefficients in [-1, 1] with P values, and drug names match the GDSC-subset drug list.
- The drop-one-out heatmap file exists and the T-cell-removed column/row exhibits the largest spread (variability) of per-drug score deltas among all removed cell types.
- DrugScore outputs cover all annotated cell clusters for each of the 4 TNBC patients and are standardized as per Formula 2 of the paper.
- Preprocessing invariants hold: genes in fewer than 3 cells and cells with fewer than 200 unique genes are removed before clustering.
- GetDrug was run with repurposing.unit='drug' and connectivity='negative' against breast_rankMatrix.txt, matching the paper's breast-tissue reference configuration.

## Limitations

- Drug reference is limited to 98 cell lines from LINCS L1000; no proper cell-type-specific reference may exist for a target disease, which bounds transferability of the breast reference.
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, affecting the Limma DEG inputs to GetDrug.
- Differential expression method choice affects downstream performance — Limma (Bayes) showed the best average AUC among tested methods, but DE methods should be selected per dataset.
- Obtaining proper normal-tissue controls may not be feasible for some diseases; the GSE161529 healthy-control design may not generalize.
- No ASGARD software version is stated in the text; results may vary across package versions.

## Evidence

- [abstract] TRANSACT validation on TNBC samples: "we validate ASGARD using the drug response prediction method TRANSACT with Triple-Negative-Breast-Cancer patient samples"
- [discussion] Multi-cluster drug score definition: "ASGARD defines a drug score at the individual patient level (Formula 1), which calculates the drug efficacy across all single-cell clusters"
- [discussion] SingleR cell-type annotation: "we ran an automatic annotation of single cells based on similarity to the referenced single-cell panel using the SingleR package75"
- [discussion] TNBC dataset source: "ScRNA-seq data of cells from 4 Triple-Negative-Breast-Cancer (TNBC) patients and 4 healthy controls are from “GSE161529”"
- [discussion] Cell/gene filtering thresholds: "cells from TNBC patients with fewer than 200 unique genes are also removed from the dataset"
- [methods] ASGARD installation: "devtools::install_github("lanagarmire/Asgard")"
- [methods] DrugScore call in the pipeline: "Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list,"
- [discussion] Differential expression method performance: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
