---
name: multi-cluster-drug-score-aggregation
description: Use when you have per-cell-type drug rankings from ASGARD GetDrug across
  multiple clusters of a patient's scRNA-seq data and need a single patient-level
  drug recommendation that accounts for heterogeneous cell populations.
when_to_use_negative:
- When the disease is driven by a single homogeneous cell population — per-cluster
  predictions alone may suffice, though the paper still reports multi-cluster AUC
  superiority
- When no normal-tissue control samples exist to compute case-vs-control differential
  signatures (the article notes substitutes are needed until normal tissue data are
  available)
- When the drug reference lacks a proper tissue-matched cell-line panel — the reference
  is limited to 98 LINCS L1000 cell lines
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0154
tools:
- name: ASGARD
  role: Provides the DrugScore function that aggregates per-cluster GetDrug results
    into the patient-level standardized drug score, and TopDrug for final candidate
    selection
  repo: https://github.com/lanagarmire/ASGARD
- name: Seurat
  role: Produces the integrated single-cell object (anchor.features=2000, dims 1:15,
    Louvain resolution 0.4) whose clusters and cell metadata define the aggregation
    units
  repo: https://github.com/lanagarmire/ASGARD
- name: limma
  role: Computes the per-cluster differential expression contrasts (typeCase - typeControl)
    that feed GetDrug and thus DrugScore
  repo: https://github.com/lanagarmire/ASGARD
- name: cmapR
  role: Reads the LINCS L1000 Level5 gctx perturbation profiles used to build the
    drug reference underlying the score
  repo: https://github.com/lanagarmire/ASGARD
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:08ff9b120ef9640fa706679930235aa739109e84c551dca215960851348949ec
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
  - TRANSACT
  - celldex
  - cmapR
  - limma
  edam_operation: http://edamontology.org/operation_3803
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

# multi-cluster-drug-score-aggregation

## Summary

Aggregates ASGARD per-cluster drug repurposing rankings into a patient-level drug therapeutic score (DrugScore) that considers all diseased cell clusters, addressing intratumoral cellular heterogeneity before TopDrug selects significant FDA-approved candidates.

## When to use

Use when you have per-cell-type drug rankings from ASGARD GetDrug across multiple clusters of a patient's scRNA-seq data and need a single patient-level drug recommendation that accounts for heterogeneous cell populations.

## When NOT to use

- When the disease is driven by a single homogeneous cell population — per-cluster predictions alone may suffice, though the paper still reports multi-cluster AUC superiority
- When no normal-tissue control samples exist to compute case-vs-control differential signatures (the article notes substitutes are needed until normal tissue data are available)
- When the drug reference lacks a proper tissue-matched cell-line panel — the reference is limited to 98 LINCS L1000 cell lines

## Inputs

- Per-cluster differential expression gene lists (limma moderated t, contrast typeCase - typeControl)
- Per-cluster ASGARD GetDrug results (Drug.ident.res) with repurposing.unit='drug', connectivity='negative', drug.type='FDA'
- Cell metadata (cell_metadata) with cluster annotations from Seurat-integrated object
- Tissue-specific drug reference from GetDrugRef (e.g., DrugReference/breast_rankMatrix.txt)
- LINCS L1000 Level5 gctx files: GSE92742_Broad_LINCS_Level5_COMPZ.MODZ_n473647x12328.gctx and GSE70138_Broad_LINCS_Level5_COMPZ_n118050x12328_2017-03-06.gctx
- Case sample identifiers (e.g., PDX-110/PDX-332, TNBC patients, deceased COVID-19 patients)

## Outputs

- Patient-level standardized multi-cluster drug score (DrugScore) with therapeutic score and FDR per drug
- Ranked drug list (e.g., covid19_balf_drug_score_ranking.csv)
- Significant drug set (therapeutic score > 0.99 quantile, FDR < 0.05) via TopDrug with FDA.drug.only=TRUE
- Per-cluster drug rankings and concordantly expressed gene sets

## How to apply

First run GetDrug per cell type on the cluster-level differential signatures (gene.data = Gene.list from limma contrasts typeCase - typeControl with min.cells = 3) with repurposing.unit='drug', connectivity='negative', drug.type='FDA' against the tissue-specific DrugReference (e.g., breast_rankMatrix.txt built by PrepareReference from LINCS L1000 GSE70138/GSE92742 Level5 gctx files). Then call DrugScore(cell_metadata, cluster_degs = Gene.list, cluster_drugs = Drug.ident.res, case = case samples) to compute the per-patient multi-cluster drug score using the article's Formula 1 (drug efficacy across all single-cell clusters) and Formula 2 (standardized drug score), together with FDR estimates. Select significant drugs via the 99th-percentile therapeutic score threshold (Drug.therapeutic.score > 0.99 quantile) with FDR < 0.05, and per-cluster candidates with TopDrug (Drug.FDR = 0.1, FDA.drug.only = TRUE). The rationale is that single-cluster predictions miss multi-cluster diseases: the multi-cluster score yielded significantly better AUC (Student's t-test P<0.05) than individual-cluster predictions (0.92 breast, 0.95 leukemia, 0.88 COVID-19).

## Related tools

- **ASGARD** (Provides the DrugScore function that aggregates per-cluster GetDrug results into the patient-level standardized drug score, and TopDrug for final candidate selection) — https://github.com/lanagarmire/ASGARD
- **Seurat** (Produces the integrated single-cell object (anchor.features=2000, dims 1:15, Louvain resolution 0.4) whose clusters and cell metadata define the aggregation units) — https://github.com/lanagarmire/ASGARD
- **limma** (Computes the per-cluster differential expression contrasts (typeCase - typeControl) that feed GetDrug and thus DrugScore) — https://github.com/lanagarmire/ASGARD
- **cmapR** (Reads the LINCS L1000 Level5 gctx perturbation profiles used to build the drug reference underlying the score) — https://github.com/lanagarmire/ASGARD

## Examples

```
Drug.score <- DrugScore(cell_metadata, cluster_degs = Gene.list, cluster_drugs = Drug.ident.res, case = deceased patients)
```

## Evaluation signals

- Significant drug set size matches benchmarks: 11 significant drugs for the breast PDX run (fostamatinib top-1, colchicine 2nd) and 13 for the TNBC patient run
- Selected drugs pass Drug.therapeutic.score > 0.99 quantile with FDR < 0.05 (or TopDrug with Drug.FDR=0.1, FDA.drug.only=TRUE)
- Multi-cluster ASGARD score yields significantly better AUC (pROC, Student's t-test P<0.05) than individual-cluster predictions and bulk baselines CLUE/DrInsight
- Disease-specific top ranks reproduce: tretinoin top-1 for Pre-T ALL; rescinnamine rank-2 and enalapril rank-4 for severe COVID-19 BALF
- All drug score values are finite with FDR values in [0, 1] for the drug universe queried

## Limitations

- Requires >64 GB RAM and more than one hour to produce the drug reference via PrepareReference
- Drug reference is limited to 98 cell lines from LINCS L1000; a proper cell-type-specific reference may not exist for the target disease
- Systematic differences between cell line and primary tissues are not adjusted for (transfer learning proposed but not demonstrated)
- DE method choice affects downstream AUC; limma (Bayes) showed the best average AUC but must be selected per dataset

## Evidence

- [abstract] we propose A Single-cell Guided Pipeline to Aid Repurposing of Drugs (ASGARD) that defines a drug score to recommend drugs by considering all cell clusters: "we propose A Single-cell Guided Pipeline to Aid Repurposing of Drugs (ASGARD) that defines a drug score to recommend drugs by considering all cell clusters"
- [discussion] ASGARD defines a drug score at the individual patient level (Formula 1), which calculates the drug efficacy across all single-cell clusters: "ASGARD defines a drug score at the individual patient level (Formula 1), which calculates the drug efficacy across all single-cell clusters"
- [discussion] The prediction using the multi-cluster drug score shows a significantly (P-value < 0.05, Student’s t test) better AUC than the prediction based on individual clusters: "The prediction using the multi-cluster drug score shows a significantly (P-value < 0.05, Student’s t test) better AUC than the prediction based on individual clusters"
- [methods] Final.drugs<-TopDrug(SC.integrated=SC.integrated: "Final.drugs<-TopDrug(SC.integrated=SC.integrated"
- [readme] #Note: the file names here maybe different after unzipping.
#Please note that it takes more than one hour to produce drug references in a standard computer with RAM>64GB.: "it takes more than one hour to produce drug references in a standard computer with RAM>64GB"
