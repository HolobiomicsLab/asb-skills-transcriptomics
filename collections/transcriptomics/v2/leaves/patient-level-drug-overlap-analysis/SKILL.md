---
name: patient-level-drug-overlap-analysis
description: Use when you have per-cluster differential expression from multiple diseased
  scRNA-seq samples (PDX or patient) plus a tissue-specific LINCS L1000 drug reference,
  and you need patient-level drug recommendations and cross-cohort overlap of significant
  drugs.
when_to_use_negative:
- Input is bulk-cell expression with no cell clusters or per-cluster DE lists — ASGARD's
  DrugScore requires multi-cluster input and bulk methods (CLUE, DrInsight) are the
  comparators, not the target
- No normal/reference tissue controls are obtainable for the disease — the article
  flags that substitutes must be found before patient-level scoring is meaningful
- Goal is single-cluster drug ranking rather than patient-level aggregation and cross-cohort
  overlap — GetDrug/TopDrug alone suffices
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0202
tools:
- name: ASGARD
  role: Provides DrugScore for multi-cluster patient-level drug scoring and TopDrug
    for per-cluster FDA-only candidates; the overlap analysis is computed from its
    significant-drug outputs
  repo: github.com/lanagarmire/Asgard
- name: Seurat
  role: Supplies the integrated scRNA-seq object with cell metadata and cluster annotations
    consumed by DrugScore
  repo: github.com/lanagarmire/Single-cell-drug-repositioning
- name: limma
  role: Computes per-cluster differential expression (Bayes) contrasts that feed the
    drug repurposing step; best average AUC among DE methods tested
- name: cmapR
  role: Reads the LINCS L1000 Level5 .gctx perturbation signatures (GSE70138, GSE92742)
    used by GetDrugRef/DrugScore
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:e7e0ff87ecec9978cdfe6c27f842cf780fe499f4e52ce82b34d1b755773d8bb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ASGARD
  - Seurat
  - cmapR
  - limma
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# patient-level-drug-overlap-analysis

## Summary

ASGARD-based analysis that aggregates per-cluster drug rankings into a multi-cluster, patient-level DrugScore and then intersects the resulting significant drug lists across independent patient/PDX cohorts (e.g., metastatic TNBC PDX models vs. TNBC patient scRNA-seq) to identify convergent repurposing candidates.

## When to use

Use when you have per-cluster differential expression from multiple diseased scRNA-seq samples (PDX or patient) plus a tissue-specific LINCS L1000 drug reference, and you need patient-level drug recommendations and cross-cohort overlap of significant drugs.

## When NOT to use

- Input is bulk-cell expression with no cell clusters or per-cluster DE lists — ASGARD's DrugScore requires multi-cluster input and bulk methods (CLUE, DrInsight) are the comparators, not the target
- No normal/reference tissue controls are obtainable for the disease — the article flags that substitutes must be found before patient-level scoring is meaningful
- Goal is single-cluster drug ranking rather than patient-level aggregation and cross-cohort overlap — GetDrug/TopDrug alone suffices

## Inputs

- Integrated Seurat object (anchor-based integration, anchor.features=2000, dims 1:15, cell-cycle regression, Louvain resolution 0.4)
- Per-cluster differential expression gene lists from limma (contrast typeCase - typeControl, min 3 cells per type)
- Tissue-specific drug reference from PrepareReference on GSE70138 and GSE92742 L1000 Level5 profiles (e.g., breast_rankMatrix.txt, breast_gene_info.txt, breast_drug_info.txt)
- GSE92742_Broad_LINCS_Level5_COMPZ.MODZ_n473647x12328.gctx and GSE70138_Broad_LINCS_Level5_COMPZ_n118050x12328_2017-03-06.gctx
- Case/control sample metadata (e.g., PDX-110/PDX-332 vs. breast epithelial controls; GSE161529 TNBC patients vs. healthy controls)

## Outputs

- Patient-level DrugScore table with Drug.therapeutic.score and Drug.FDR per drug
- Significant drug list per cohort (Drug.therapeutic.score > 0.99 quantile, FDR < 0.05)
- Per-cluster top drug candidates from TopDrug (Drug.FDR=0.1, FDA.drug.only=TRUE)
- Cross-cohort overlap set of significant drugs (e.g., mebendazole, crizotinib, neratinib, vinblastine)

## How to apply

Start from the integrated Seurat object and per-cluster limma DE statistics (contrasts typeCase - typeControl, minimum 3 cells per type). Build the tissue-specific drug reference with PrepareReference from LINCS L1000 GSE70138 and GSE92742 Level5 profiles, then GetDrugRef on the breast rankMatrix. Run GetDrug per cell type with repurposing.unit='drug', connectivity='negative', and drug.type='FDA'. Aggregate per-cluster results into the patient-level DrugScore with tissue='breast' (L1000 reference covers 98 cell lines), comparing case samples (e.g., PDX-110/PDX-332 or TNBC patients) against normal controls using the GSE92742 MODZ and GSE70138 Level5 gctx files. Select significant drugs via the Drug.therapeutic.score > 0.99 quantile with FDR < 0.05, and per-cluster candidates via TopDrug with Drug.FDR=0.1 and FDA.drug.only=TRUE. Repeat for the second cohort (e.g., the 4 GSE161529 TNBC patients) and compute the overlap of significant drugs; rationale is that multi-cluster aggregation addresses cellular heterogeneity that single-cluster or bulk methods miss.

## Related tools

- **ASGARD** (Provides DrugScore for multi-cluster patient-level drug scoring and TopDrug for per-cluster FDA-only candidates; the overlap analysis is computed from its significant-drug outputs) — github.com/lanagarmire/Asgard
- **Seurat** (Supplies the integrated scRNA-seq object with cell metadata and cluster annotations consumed by DrugScore) — github.com/lanagarmire/Single-cell-drug-repositioning
- **limma** (Computes per-cluster differential expression (Bayes) contrasts that feed the drug repurposing step; best average AUC among DE methods tested)
- **cmapR** (Reads the LINCS L1000 Level5 .gctx perturbation signatures (GSE70138, GSE92742) used by GetDrugRef/DrugScore)

## Examples

```
Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list, tissue = 'breast', case = c('PDX-110','PDX-332'), repurposing.unit = 'drug', drug.type = 'FDA')
```

## Evaluation signals

- PDX run must yield 11 significant drugs with fostamatinib ranked top-1 and colchicine second
- TNBC patient (GSE161529) run must yield 13 significant drugs
- Overlap between TNBC patients and PDX predictions must contain mebendazole, crizotinib, neratinib, and vinblastine
- Significant drugs must pass the Drug.therapeutic.score > 0.99 quantile with FDR < 0.05; per-cluster candidates pass Drug.FDR=0.1 with FDA.drug.only=TRUE
- Multi-cluster drug score should outperform individual-cluster prediction (significantly better AUC by Student's t test, P < 0.05)

## Limitations

- Drug reference is limited to LINCS L1000 responses from 98 cell lines; no cell-type-specific reference may exist for the target disease
- Obtaining proper normal-tissue controls may not be feasible for some diseases
- Differential expression method choice affects downstream AUC (limma/Bayes performed best on average); sparsity and heterogeneity of single-cell data can cause false DEG discoveries

## Evidence

- [discussion] Patient-level scoring rationale: "ASGARD defines a drug score at the individual patient level (Formula 1), which calculates the drug efficacy across all single-cell clusters"
- [discussion] Multi-cluster score outperforms cluster-level prediction: "The prediction using the multi-cluster drug score shows a significantly (P-value < 0.05, Student’s t test) better AUC than the prediction based on individual clusters"
- [readme] Reference library construction from L1000: "Download L1000 Connectivity Map perturbational profiles GSE70138 and GSE92742 from GEO"
- [abstract] Validation cohort: "we validate ASGARD using the drug response prediction method TRANSACT with Triple-Negative-Breast-Cancer patient samples"
