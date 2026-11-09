---
name: single-cell-drug-repositioning
description: Use when scRNA-seq data from diseased tissue (e.g., TNBC PDX models or
  patient samples like GSE123926/GSE161529) needs FDA-approved drug recommendations
  that account for multiple heterogeneous cell clusters.
when_to_use_negative:
- Input is bulk RNA-seq or a feature table without single-cell resolution — ASGARD's
  drug score is defined across single-cell clusters and bulk-based methods like CLUE
  underperform in the benchmarks
- No suitable normal-tissue control exists for the disease — the article notes substitutes
  may be needed until normal tissue data are available
- No tissue-matched LINCS L1000 drug reference exists for the target disease — the
  reference is limited to 98 cell lines
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_0154
tools:
- name: ASGARD
  role: Core R package providing PrepareReference, GetDrugRef, GetDrug, DrugScore,
    TopDrug, and DrugCombination functions
  repo: https://github.com/lanagarmire/ASGARD
- name: Seurat
  role: scRNA-seq integration, cell-cycle regression, and Louvain clustering of disease
    and normal cells
- name: limma
  role: Per-cluster differential expression (contrasts typeCase - typeControl); best
    average AUC among DE methods tested
- name: cmapR
  role: Parsing LINCS L1000 GCTx perturbational profile files
- name: SingleR
  role: Automatic cell-type annotation against a referenced single-cell panel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:aa685d5010b2bfbeff3ff9631a79f5a5e743a14197e0cf375b2ecd3050ea2ef2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/ASGARD
  tools:
  - ASGARD
  - Seurat
  - SingleR
  - cmapR
  - limma
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-drug-repositioning

## Summary

Apply the ASGARD pipeline (R package) to scRNA-seq data to recommend repurposed drugs by computing a per-patient multi-cluster DrugScore across all diseased cell clusters, addressing intratumoral cellular heterogeneity that bulk-cell methods miss.

## When to use

Use when scRNA-seq data from diseased tissue (e.g., TNBC PDX models or patient samples like GSE123926/GSE161529) needs FDA-approved drug recommendations that account for multiple heterogeneous cell clusters.

## When NOT to use

- Input is bulk RNA-seq or a feature table without single-cell resolution — ASGARD's drug score is defined across single-cell clusters and bulk-based methods like CLUE underperform in the benchmarks
- No suitable normal-tissue control exists for the disease — the article notes substitutes may be needed until normal tissue data are available
- No tissue-matched LINCS L1000 drug reference exists for the target disease — the reference is limited to 98 cell lines

## Inputs

- Integrated Seurat object of disease and normal scRNA-seq cells (e.g., GSE123926 PDX epithelial cells vs GSE113197 normal breast epithelial cells)
- Per-cluster differential expression statistics computed with limma
- L1000 perturbational profiles: GSE70138_Broad_LINCS_Level5_COMPZ_n118050x12328.gctx and GSE92742_Broad_LINCS_Level5_COMPZ.MODZ_n473647x12328.gctx
- Cell/sig/gene info files from GSE70138 and GSE92742 for PrepareReference
- Tissue-specific rankMatrix (e.g., DrugReference/breast_rankMatrix.txt) with breast_gene_info.txt and breast_drug_info.txt

## Outputs

- Per-cluster drug rankings from GetDrug
- Patient-level multi-cluster DrugScore with therapeutic scores and FDR values
- Significant drug list (Drug.therapeutic.score > 0.99 quantile, FDR < 0.05)
- Final FDA-approved drug candidates and drug combinations via TopDrug and DrugCombination

## How to apply

Preprocess scRNA-seq data with Seurat (integration via anchors with anchor.features=2000, dims 1:15, cell-cycle regression, Louvain clustering at resolution 0.4), compute per-cluster differential expression with limma (contrasts typeCase - typeControl, minimum 3 cells per type), then build a tissue-specific drug reference with GetDrugRef from LINCS L1000 profiles (GSE70138 and GSE92742 Level5 GCTx files) via PrepareReference. Run GetDrug per cell type with repurposing.unit='drug', connectivity='negative', and drug.type='FDA', then aggregate per-cluster results into the patient-level DrugScore (tissue='breast') using the GSE92742 and GSE70138 GCTx files. Select significant drugs at Drug.therapeutic.score > 0.99 quantile with FDR < 0.05, and per-cluster candidates via TopDrug with Drug.FDR=0.1 and FDA.drug.only=TRUE; the K-S test on the outlier-sum statistic with BH FDR underlies the ranking, and limma (Bayes) showed the best average AUC among DE methods tested.

## Related tools

- **ASGARD** (Core R package providing PrepareReference, GetDrugRef, GetDrug, DrugScore, TopDrug, and DrugCombination functions) — https://github.com/lanagarmire/ASGARD
- **Seurat** (scRNA-seq integration, cell-cycle regression, and Louvain clustering of disease and normal cells)
- **limma** (Per-cluster differential expression (contrasts typeCase - typeControl); best average AUC among DE methods tested)
- **cmapR** (Parsing LINCS L1000 GCTx perturbational profile files)
- **SingleR** (Automatic cell-type annotation against a referenced single-cell panel)

## Examples

```
devtools::install_github("lanagarmire/Asgard"); Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list, drug.ref.profiles = drug.ref.profiles, tissue='breast')
```

## Evaluation signals

- PDX breast cancer run must yield 11 significant drugs with fostamatinib ranked top-1 and colchicine ranked second
- TNBC patient run (GSE161529) must yield 13 significant drugs, sharing mebendazole, crizotinib, neratinib, and vinblastine with the PDX predictions
- Multi-cluster DrugScore AUC must be significantly better than individual-cluster predictions (P-value < 0.05, Student's t test, via pROC)
- Drug reference construction completes without error and takes over one hour on a machine with RAM > 64GB
- ROC/AUC benchmarking against CLUE, DrInsight, Alakwaa's, and Guo's pipelines shows ASGARD outperforming all of them

## Limitations

- Drug reference built from only 98 cell lines in LINCS L1000; no proper cell-type-specific reference may exist for a target disease
- Obtaining proper normal-tissue controls may not be feasible for some diseases; substitutes needed
- AUC performance varies with cell population sizes, proportions, and DE levels; sparsity and heterogeneity of single-cell data can cause false DE discoveries
- Differential expression method choice affects downstream AUC and must be selected per dataset
- No ASGARD software version stated in the text

## Evidence

- [intro] ASGARD defines a drug score to predict drugs for multiple diseased cell clusters within each patient: "Here we propose A Single-cell Guided Pipeline to Aid Repurposing of Drugs (ASGARD) to overcome the issue above. ASGARD defines a drug score to predict drugs for multiple diseased cell clusters within"
- [methods] GetDrugRef builds tissue-specific drug reference: "drug.ref.profiles = GetDrugRef(drug.response.path = 'DrugReference/breast_rankMatrix.txt'"
- [methods] DrugScore aggregates across clusters: "Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list,"
- [readme] PrepareReference from L1000 GSE70138 and GSE92742: "#### Generate tissue specific drug references from GSE70138 and GSE92742"
- [discussion] Reversed gene patterns with K-S test and OS statistic: "The Kolmogorov–Smirnov test (K-S test) is then applied to the OS statistic"
- [discussion] Limma best average AUC among DE methods: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
