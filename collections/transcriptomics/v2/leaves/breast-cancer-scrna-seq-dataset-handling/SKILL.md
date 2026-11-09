---
name: breast-cancer-scrna-seq-dataset-handling
description: Use when a task requires the exact breast-cancer scRNA-seq inputs of
  the ASGARD study (GSE123926, GSE113197 epithelial cells) loaded via Seurat with
  the paper's cell/gene filters.
when_to_use_negative:
- When the task concerns non-epithelial or non-breast datasets (e.g. GSE132509 Pre-T
  ALL or GSE145926 COVID-19 BALF, which use different QC thresholds such as >6000
  genes or >10% mitochondrial genes)
- When only bulk-cell or drug-reference data (LINCS L1000 GSE70138/GSE92742) are needed
  for PrepareReference rather than patient scRNA-seq
- When a normalized/processed Seurat object is already provided — re-running raw loading
  would duplicate preprocessing
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Loads expression matrices and builds filtered Seurat objects (CreateSeuratObject
    with min.cells=3, min.features=200) for the breast epithelial datasets
  repo: https://github.com/lanagarmire/Asgard
- name: Asgard
  role: R package whose drug-repurposing workflow consumes these breast scRNA-seq
    objects via GetDrugRef/GetDrug
  repo: https://github.com/lanagarmire/Asgard
- name: Single-cell-drug-repositioning
  role: Provides the demo scripts and Normal_celltype.txt annotations used to reproduce
    the paper's breast-cancer analysis
  repo: https://github.com/lanagarmire/Single-cell-drug-repositioning
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:99961174b21b22feb97b7eb3a0499e0969aa1f8a2ff73627ef24547d3394010d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - Asgard
  - CLUE
  - Seurat
  - Single-cell-drug-repositioning
  - pROC
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# breast-cancer-scrna-seq-dataset-handling

## Summary

Download and load the breast cancer/normal breast epithelial scRNA-seq expression matrices (GSE123926 TNBC PDX and GSE113197 normal breast) used by the ASGARD paper, applying the paper's Seurat creation and gene/cell filtering thresholds so downstream drug repurposing and simulation analyses start from the correct processed object.

## When to use

Use when a task requires the exact breast-cancer scRNA-seq inputs of the ASGARD study (GSE123926, GSE113197 epithelial cells) loaded via Seurat with the paper's cell/gene filters.

## When NOT to use

- When the task concerns non-epithelial or non-breast datasets (e.g. GSE132509 Pre-T ALL or GSE145926 COVID-19 BALF, which use different QC thresholds such as >6000 genes or >10% mitochondrial genes)
- When only bulk-cell or drug-reference data (LINCS L1000 GSE70138/GSE92742) are needed for PrepareReference rather than patient scRNA-seq
- When a normalized/processed Seurat object is already provided — re-running raw loading would duplicate preprocessing

## Inputs

- GEO raw expression matrices from GSE123926 (GSM-level txt files, e.g. PDX TNBC epithelial cells)
- GEO raw expression matrices from GSE113197 (e.g. GSM3099847_Ind5_Expression_Matrix.txt)
- Normal_celltype.txt cell-type annotation table from lanagarmire/Single-cell-drug-repositioning (sample, celltype columns)

## Outputs

- Seurat objects for TNBC PDX epithelial cells and normal breast epithelial cells with condition and cell-type metadata
- Filtered expression matrices (genes in >=3 cells, cells with >=200 unique genes) ready for DE analysis, bootstrapped simulation, or ASGARD GetDrug

## How to apply

Download GSE123926 (epithelial cells from PDX models of 2 patients with advanced metastatic TNBC) and GSE113197 (breast epithelial cells from 3 healthy women) from GEO, e.g. the GSE123926_RAW.tar and GSE113197_RAW.tar archives, plus the Normal_celltype.txt annotation file from the Single-cell-drug-repositioning repo. Read the expression matrices with read.table, set gene IDs as rownames, and subset cells to the epithelial cell types of interest (Luminal L1.1/L1.2/L2 and Basal epithelial cells) using the sample/celltype annotations. Build Seurat objects with CreateSeuratObject using min.cells=3 and min.features=200, matching the paper's rule that genes detected in fewer than three cells and cells with fewer than 200 unique genes are removed. Keep patient/condition metadata (e.g. type='normal') in meta.data so case/control pairing and later bootstrapped simulations have correct labels. Correct application yields Seurat objects whose cells match the annotated epithelial subsets and whose matrices pass the stated count thresholds.

## Related tools

- **Seurat** (Loads expression matrices and builds filtered Seurat objects (CreateSeuratObject with min.cells=3, min.features=200) for the breast epithelial datasets) — https://github.com/lanagarmire/Asgard
- **Asgard** (R package whose drug-repurposing workflow consumes these breast scRNA-seq objects via GetDrugRef/GetDrug) — https://github.com/lanagarmire/Asgard
- **Single-cell-drug-repositioning** (Provides the demo scripts and Normal_celltype.txt annotations used to reproduce the paper's breast-cancer analysis) — https://github.com/lanagarmire/Single-cell-drug-repositioning

## Examples

```
data <- read.table(file="GSM3099847_Ind5_Expression_Matrix.txt", header=TRUE, check.names=FALSE); epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, min.features=200)
```

## Evaluation signals

- Seurat object metadata contains type='normal' for GSE113197 samples (Ind5, Ind6) and disease labels for GSE123926 PDX samples
- Cell names in the object match those in the Normal_celltype.txt annotation after intersecting colnames with annotated epithelial cells
- No genes present in fewer than three cells remain; no cells with fewer than 200 unique genes remain (min.cells=3, min.features=200)
- Retained normal cells are restricted to Luminal_L2, Luminal_L1.1, Luminal_L1.2, and Basal epithelial cell types
- Raw matrices match the GEO files GSM3099847_Ind5_Expression_Matrix.txt and the GSE123926_RAW.tar contents

## Limitations

- Normal-tissue controls may not be feasible or available for every disease, so substituting GSE113197 normal breast epithelium only works for breast-cancer-like contexts
- Only the specific GSE accession sets (GSE123926, GSE113197) are validated in the paper; other breast datasets would need re-checking of cell-type annotations
- Building downstream drug references requires a large amount of RAM (64GB) and over an hour of runtime per the README, though loading the expression data itself is lighter

## Evidence

- [discussion] Epithelial cells from Patient-Derived Xenografts (PDXs) models of 2 patients with advanced metastatic TNBCs and adult human breast epithelial cells from 3 healthy women are from GEO: "Epithelial cells from Patient-Derived Xenografts (PDXs) models of 2 patients with advanced metastatic TNBCs and adult human breast epithelial cells from 3 healthy women are from GEO"
- [discussion] genes identified in fewer than three cells are removed from the dataset: "genes identified in fewer than three cells are removed from the dataset"
- [discussion] cells from TNBC patients with fewer than 200 unique genes are also removed from the dataset: "cells from TNBC patients with fewer than 200 unique genes are also removed from the dataset"
- [readme] Load normal sample Ind5 from GSE113197 dataset: "Load normal sample Ind5 from GSE113197 dataset 
data <- read.table(file="GSM3099847_Ind5_Expression_Matrix.txt", 
                   header=TRUE, check.names=FALSE)"
- [readme] epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, min.features=200): "epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, 
									  min.features=200, meta.data=metada)"
