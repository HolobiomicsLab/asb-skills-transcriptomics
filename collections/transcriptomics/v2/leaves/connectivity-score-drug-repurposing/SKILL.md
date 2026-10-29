---
name: connectivity-score-drug-repurposing
description: Use when case/control scRNA-seq differential expression signatures (e.g.,
  limma moderated t-statistics) are available per cell cluster and the goal is to
  rank repurposed drugs, optionally restricted to FDA-approved candidates, across
  heterogeneous cell types.
when_to_use_negative:
- Input is bulk RNA-seq data without cell clusters — ASGARD's advantage (multi-cluster
  drug score) was benchmarked against bulk methods like CLUE and DrInsight precisely
  for heterogeneous single-cell data
- Disease is driven by a single cell type and no normal-tissue control exists — the
  article notes obtaining proper normal samples may not be feasible for some diseases
- A tissue-specific LINCS L1000 drug reference is unavailable for the target disease,
  since the reference covers only 98 cell lines
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0154
tools:
- name: ASGARD
  role: 'Core pipeline: PrepareReference, GetDrugRef, GetDrug, DrugScore, TopDrug,
    DrugCombination functions execute connectivity-based repurposing'
  repo: https://github.com/lanagarmire/ASGARD
- name: limma
  role: Computes per-cluster moderated t-statistic differential expression signatures
    fed into GetDrug
- name: Seurat
  role: Preprocesses scRNA-seq data (filtering, integration, clustering) that provides
    the cell clusters for repurposing
- name: SingleR
  role: Annotates T-cell clusters against the HumanPrimaryCellAtlasData (celldex)
    reference prior to per-cluster GetDrug calls
- name: cmapR
  role: Reads LINCS L1000 .gctx perturbation matrices used to build the drug reference
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:d8e6934e8c217d2e7adf050780e1df1f1698af20d9b6424fecdaf7edd51245c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/ASGARD
  tools:
  - ASGARD
  - CLUE
  - DrInsight
  - Seurat
  - SingleR
  - celldex
  - cmapR
  - limma
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# connectivity-score-drug-repurposing

## Summary

Performs single-cell drug repurposing by reversing disease differential-expression signatures against LINCS L1000 Level 5 perturbation profiles, ranking FDA-approved drugs per cell cluster with a connectivity/outlier-sum statistic (ASGARD GetDrug), then aggregating into a patient-level multi-cluster drug score (DrugScore).

## When to use

Use when case/control scRNA-seq differential expression signatures (e.g., limma moderated t-statistics) are available per cell cluster and the goal is to rank repurposed drugs, optionally restricted to FDA-approved candidates, across heterogeneous cell types.

## When NOT to use

- Input is bulk RNA-seq data without cell clusters — ASGARD's advantage (multi-cluster drug score) was benchmarked against bulk methods like CLUE and DrInsight precisely for heterogeneous single-cell data
- Disease is driven by a single cell type and no normal-tissue control exists — the article notes obtaining proper normal samples may not be feasible for some diseases
- A tissue-specific LINCS L1000 drug reference is unavailable for the target disease, since the reference covers only 98 cell lines

## Inputs

- Per-cluster differential expression gene lists (e.g., limma moderated t-statistics; cross-checks with Seurat Wilcoxon, DESeq2, edgeR)
- Seurat integrated single-cell object with cell-type annotations (SingleR/celldex HumanPrimaryCellAtlasData)
- Tissue-specific drug reference built from LINCS L1000 Level 5 GSE70138 and GSE92742 .gctx perturbation profiles (12328 genes)

## Outputs

- Per-cell-type ranked drug/compound list with connectivity test statistics and FDR values
- Patient-level multi-cluster drug therapeutic scores (DrugScore)
- Final FDA-approved drug shortlist (TopDrug) with FDR thresholds, e.g., tretinoin top-1 for Pre-T ALL

## How to apply

First build a tissue-specific drug reference from LINCS L1000 Level 5 perturbation signatures GSE70138 and GSE92742 (.gctx files, 12328 genes) using PrepareReference and load it with GetDrugRef; this requires >64GB RAM and takes more than one hour. Then compute per-cluster DEG signatures (limma moderated t-statistics, minimum min.cells = 3 cells per group) and run GetDrug per cell type with repurposing.unit = 'drug', connectivity = 'negative' (reverse signatures), and drug.type = 'FDA'. The connectivity scoring applies an outlier-sum statistic evaluated with a Kolmogorov–Smirnov test and Benjamini–Hochberg FDR. Aggregate cluster-level drug ranks into the patient-level drug score with DrugScore, then call TopDrug to select final drugs (e.g., Drug.FDR = 0.1, FDA.drug.only = TRUE, or 99th-percentile therapeutic score with FDR < 0.05 for validation tasks such as Pre-T ALL tretinoin). Optionally combine drugs with DrugCombination.

## Related tools

- **ASGARD** (Core pipeline: PrepareReference, GetDrugRef, GetDrug, DrugScore, TopDrug, DrugCombination functions execute connectivity-based repurposing) — https://github.com/lanagarmire/ASGARD
- **limma** (Computes per-cluster moderated t-statistic differential expression signatures fed into GetDrug)
- **Seurat** (Preprocesses scRNA-seq data (filtering, integration, clustering) that provides the cell clusters for repurposing)
- **SingleR** (Annotates T-cell clusters against the HumanPrimaryCellAtlasData (celldex) reference prior to per-cluster GetDrug calls)
- **cmapR** (Reads LINCS L1000 .gctx perturbation matrices used to build the drug reference)

## Examples

```
Drug.ident.res = GetDrug(gene.data = Gene.list, drug.ref.profiles = drug.ref, repurposing.unit = 'drug', connectivity = 'negative', drug.type = 'FDA')
```

## Evaluation signals

- Top-ranked FDA-approved drug matches known disease therapy: tretinoin ranks top-1 for Pre-T ALL and is FDA-approved for leukemia
- Final drug selection passes the 99th-percentile therapeutic score threshold with FDR < 0.05 (or Drug.FDR = 0.1 for TopDrug)
- Reference build completes from both GSE70138 (118050 x 12328) and GSE92742 (473647 x 12328) Level 5 COMPZ matrices
- Multi-cluster drug score yields significantly better AUC than per-cluster predictions (Student's t-test, P < 0.05) when benchmarked with pROC
- limma (Bayes) DE method shows the best average AUC; verify DE method choice does not degrade downstream rankings

## Limitations

- Building the drug reference requires >64GB RAM and more than one hour per tissue
- Drug reference is limited to 98 LINCS L1000 cell lines; no proper cell-type-specific reference may exist for a target disease
- Obtaining proper normal-tissue controls may not be feasible for some diseases; substitutes needed
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, propagating into drug rankings
- Differential expression method choice affects downstream AUC performance and must be selected per dataset

## Evidence

- [methods] connectivity = 'negative' reverses disease signatures: "Drug.ident.res = GetDrug(gene.data = Gene.list,"
- [discussion] K-S test on OS statistic: "The Kolmogorov–Smirnov test (K-S test) is then applied to the OS statistic"
- [abstract] multi-cluster drug score across clusters: "defines a drug score to recommend drugs by considering all cell clusters to address the intercellular heterogeneity within each patient"
- [readme] reference build time and RAM requirements: "it takes more than one hour to produce drug references in a standard computer with RAM>64GB."
- [discussion] LINCS L1000 source data: "ASGARD built the drug reference using drug responses data from LINCS L1000 project21, which were collected from 98 cell lines"
- [discussion] limma best DE method for downstream AUC: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
