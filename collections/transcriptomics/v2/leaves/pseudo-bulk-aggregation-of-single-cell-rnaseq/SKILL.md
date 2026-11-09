---
name: pseudo-bulk-aggregation-of-single-cell-rnaseq
description: Use when scRNA-seq case/control data (e.g., GSE113197, GSE123926, GSE132509,
  GSE145926, GSE158055) must be collapsed per patient or disease class into pseudo-bulk
  profiles to run bulk baselines.
when_to_use_negative:
- When the goal is patient-level multi-cluster drug recommendation — use ASGARD's
  per-cluster DE plus DrugScore directly on the single-cell clusters instead of collapsing
  to pseudo-bulk
- When no paired normal/disease samples exist per disease class, since pseudo-bulk
  case/control contrast cannot be formed (the paper notes obtaining normal samples
  may not be feasible for some diseases)
- When only cell- or cluster-level predictions are required and patient-level aggregation
  would mask the intercellular heterogeneity ASGARD is designed to address
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Loads and filters single-cell count matrices (CreateSeuratObject with min.cells=3,
    min.features=200) before per-patient aggregation; its Wilcoxon test is one of
    the DE methods applied
- name: limma
  role: Bulk differential expression (limma Bayes) run on pseudo-bulk profiles; showed
    the best average AUC among tested DE methods
- name: DESeq2
  role: Alternative bulk DE method for pseudo-bulk signatures
- name: edgeR
  role: Alternative bulk DE method for pseudo-bulk signatures
- name: CLUE
  role: 'Bulk baseline: Connectivity Map signature-gene-based drug ranking run on
    the pseudo-bulk DE signatures'
- name: DrInsight 0.1.1
  role: 'Bulk baseline: concordantly expressed genes (CEG)-based ranking run on the
    pseudo-bulk DE signatures'
- name: ASGARD
  role: Multi-cluster drug score pipeline whose AUC is compared against pseudo-bulk
    baselines
  repo: https://github.com/lanagarmire/Asgard
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:73f8459c37ba743a2d70fa91e92a5651ae44044844ecf99d64cd5d7c8b505c51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - CLUE
  - DESeq2
  - DrInsight
  - DrInsight 0.1.1
  - Seurat
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# pseudo-bulk-aggregation-of-single-cell-rnaseq

## Summary

Aggregates per-patient or per-disease-class scRNA-seq counts into pseudo-bulk RNA-Seq profiles so bulk-cell drug-repurposing baselines (CLUE, DrInsight) can be benchmarked against ASGARD's multi-cluster drug score.

## When to use

Use when scRNA-seq case/control data (e.g., GSE113197, GSE123926, GSE132509, GSE145926, GSE158055) must be collapsed per patient or disease class into pseudo-bulk profiles to run bulk baselines.

## When NOT to use

- When the goal is patient-level multi-cluster drug recommendation — use ASGARD's per-cluster DE plus DrugScore directly on the single-cell clusters instead of collapsing to pseudo-bulk
- When no paired normal/disease samples exist per disease class, since pseudo-bulk case/control contrast cannot be formed (the paper notes obtaining normal samples may not be feasible for some diseases)
- When only cell- or cluster-level predictions are required and patient-level aggregation would mask the intercellular heterogeneity ASGARD is designed to address

## Inputs

- Raw scRNA-seq count matrices from GEO (e.g., GSE113197, GSE123926, GSE132509, GSE145926, GSE158055), as tab-delimited expression matrix files (e.g., GSM3099847_Ind5_Expression_Matrix.txt)
- Cell-level metadata/annotation tables mapping cells to patient ID, sample, and disease class (e.g., Normal_celltype.txt)
- LINCS L1000 drug reference built from GSE70138 and GSE92742 (PrepareReference output) when comparing to ASGARD
- Positive-case drug list: drugs/compounds in advanced trials (Supplementary Data 4)

## Outputs

- Pseudo-bulk expression matrix with one profile per patient or disease class (not per cell)
- Bulk differential expression signatures computed on pseudo-bulk profiles (limma Bayes, DESeq2, edgeR, or Seurat Wilcoxon)
- ROC curves and AUC values per pipeline (CLUE, DrInsight, ASGARD multi-cluster) from the pROC package

## How to apply

Download the GEO case/control scRNA-seq datasets (breast: GSE113197 + GSE123926; Pre-T ALL: GSE132509; COVID-19 BALF: GSE145926 + GSE158055) and load raw count matrices with Seurat (min.cells=3, min.features=200). Sum counts over cells per patient or disease class, producing one pseudo-bulk profile per condition rather than one per cell. Apply the paper's preprocessing: remove genes identified in fewer than three cells, drop cells with <200 unique genes (COVID-19 BALF cells also filtered at >6000 genes or >10% mitochondrial genes), scale with cell-cycle marker genes, and keep the 2000 genes with highest standardized variance. Run bulk differential expression on the pseudo-bulk profiles with limma Bayes (best average AUC in the paper), DESeq2, edgeR, or Seurat Wilcoxon, requiring a minimum of 3 cells per condition. Feed these pseudo-bulk DE signatures to the bulk baselines CLUE (Connectivity Map signature-based ranking) and DrInsight 0.1.1 (CEG-based ranking), and compare against ASGARD's multi-cluster DrugScore using ROC/AUC computed with the pROC package, treating drugs in advanced trials (Supplementary Data 4) as positive cases.

## Related tools

- **Seurat** (Loads and filters single-cell count matrices (CreateSeuratObject with min.cells=3, min.features=200) before per-patient aggregation; its Wilcoxon test is one of the DE methods applied)
- **limma** (Bulk differential expression (limma Bayes) run on pseudo-bulk profiles; showed the best average AUC among tested DE methods)
- **DESeq2** (Alternative bulk DE method for pseudo-bulk signatures)
- **edgeR** (Alternative bulk DE method for pseudo-bulk signatures)
- **CLUE** (Bulk baseline: Connectivity Map signature-gene-based drug ranking run on the pseudo-bulk DE signatures)
- **DrInsight 0.1.1** (Bulk baseline: concordantly expressed genes (CEG)-based ranking run on the pseudo-bulk DE signatures)
- **ASGARD** (Multi-cluster drug score pipeline whose AUC is compared against pseudo-bulk baselines) — https://github.com/lanagarmire/Asgard

## Examples

```
library(Seurat); so <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, min.features=200, meta.data=metadata)
```

## Evaluation signals

- One pseudo-bulk profile per patient/disease class (e.g., 4 TNBC patients vs 4 healthy controls; 2 Pre-T ALL patients vs 3 controls; 15 severe COVID-19 BALF patients), not one column per cell
- Filtering invariants hold before aggregation: genes identified in fewer than three cells removed; cells with <200 unique genes dropped (BALF cells also dropped at >6000 genes or >10% mitochondrial genes)
- pROC-based AUCs reproduce the reported pattern: ASGARD 0.92 (breast), 0.95 (leukemia), 0.88 (COVID-19) vs CLUE 0.74/0.82/0.85 and DrInsight 0.81/0.73/0.73
- Student's t-test gives P<0.05 for the multi-cluster ASGARD score versus both pseudo-bulk baselines and individual-cluster predictions
- Positive-case set matches Supplementary Data 4 drugs in advanced trials, with FDA.drug.only=TRUE applied in TopDrug

## Limitations

- Obtaining proper normal-tissue controls may not be feasible for some diseases, requiring substitutes until such data are available
- The drug reference is limited to 98 cell lines from LINCS L1000, so a proper tissue-specific reference may not exist for a target disease
- DE method choice changes downstream AUC (limma Bayes was best); sparsity and heterogeneity of single-cell data can cause false DE discoveries
- Quantitative AUCs are only shown in figures (Figs. 2, 3, Supplementary Fig. 2), not numerically in the text, and ground-truth positives rely on Supplementary Data 4

## Evidence

- [other] When scRNA-seq data were aggregated to pseudo-bulk, ASGARD outperformed the bulk-cell-based methods across all three diseases, achieving AUCs of 0.92 (breast cancer): [evidence span withheld — anchor status: not_found]
- [discussion] Epithelial cells from Patient-Derived Xenografts (PDXs) models of 2 patients with advanced metastatic TNBCs and adult human breast epithelial cells from 3 healthy women: "Epithelial cells from Patient-Derived Xenografts (PDXs) models of 2 patients with advanced metastatic TNBCs and adult human breast epithelial cells from 3 healthy women"
- [discussion] ROCs and AUCs are calculated for each pipeline using the pROC package79: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
- [discussion] Sparsity and heterogeneity are two major challenges in analyzing single-cell data, and usually cause false discoveries of differentially expressed genes: "Sparsity and heterogeneity are two major challenges in analyzing single-cell data, and usually cause false discoveries of differentially expressed genes"
- [readme] Download L1000 Connectivity Map perturbational profiles GSE70138 and GSE92742 from GEO: "Download L1000 Connectivity Map perturbational profiles GSE70138 and GSE92742 from GEO"
