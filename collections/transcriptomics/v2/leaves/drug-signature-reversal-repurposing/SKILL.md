---
name: drug-signature-reversal-repurposing
description: Use when you have case-vs-control scRNA-seq clusters with differential
  expression profiles and want FDA-approved drug candidates that reverse disease signatures
  across multiple cell populations, e.g. TNBC, leukemia, or COVID-19 BALF data.
when_to_use_negative:
- You only have bulk RNA-seq or a single homogenous cell population — ASGARD's multi-cluster
  drug score adds value specifically for heterogeneous multi-cluster scRNA-seq data,
  and bulk-based methods like CLUE/DrInsight suffice otherwise
- No reference drug profile exists for the target tissue — the LINCS L1000 reference
  covers only 98 cell lines and a matching tissue reference may be unavailable
- No proper normal/control samples are obtainable for the disease, since case-vs-control
  DE per cluster is a required input
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3577
tools:
- name: ASGARD
  role: Core package providing PrepareReference, GetDrugRef, GetDrug, DrugScore, TopDrug,
    and DrugCombination for signature-reversal drug ranking across cell clusters
  repo: https://github.com/lanagarmire/ASGARD
- name: limma
  role: Computes case-vs-control differential expression t-statistics fed to GetDrug;
    best AUC (0.90-0.92) among tested DE methods
- name: Seurat
  role: Normalizes, integrates, clusters, and annotates scRNA-seq data and supplies
    cell metadata for DrugScore aggregation
- name: cmapR
  role: Reads LINCS L1000 GCT perturbational matrices used to build the drug reference
- name: SingleR
  role: Annotates cell clusters (e.g. neutrophil, NK, T cell, monocyte) against HumanPrimaryCellAtlasData
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:622c64958f18754ae9f972b7909eb62800629d4abc9ec094d7c3d658282952e3
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
  - pROC
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# drug-signature-reversal-repurposing

## Summary

Ranks repurposing candidates by reversing disease differential-expression signatures against LINCS L1000 perturbational profiles using ASGARD's GetDrug/DrugScore, aggregating per-cell-cluster drug rankings into a multi-cluster drug score that addresses intrapatient cellular heterogeneity.

## When to use

Use when you have case-vs-control scRNA-seq clusters with differential expression profiles and want FDA-approved drug candidates that reverse disease signatures across multiple cell populations, e.g. TNBC, leukemia, or COVID-19 BALF data.

## When NOT to use

- You only have bulk RNA-seq or a single homogenous cell population — ASGARD's multi-cluster drug score adds value specifically for heterogeneous multi-cluster scRNA-seq data, and bulk-based methods like CLUE/DrInsight suffice otherwise
- No reference drug profile exists for the target tissue — the LINCS L1000 reference covers only 98 cell lines and a matching tissue reference may be unavailable
- No proper normal/control samples are obtainable for the disease, since case-vs-control DE per cluster is a required input

## Inputs

- Case-vs-control scRNA-seq count matrices (e.g. GEO GSE145926 BALF, GSE161529 TNBC, GSE132509 Pre-T ALL)
- Per-cluster differential expression gene lists (Gene.list with t-statistics and p-values)
- Seurat-integrated cell metadata with cell-type annotations and case/control labels
- LINCS L1000 perturbational profiles (GSE70138 and GSE92742 GCT files with cell/sig/gene info)

## Outputs

- Per-cluster drug rankings with concordantly expressed target gene sets (Drug.ident.res)
- Multi-cluster drug score table with FDR (e.g. covid19_balf_drug_score_ranking.csv)
- Top-ranked FDA-approved drug candidates per patient/disease (Final.drugs from TopDrug)
- Predicted drug combinations (Drug.combinations)

## How to apply

Compute case-vs-control differential expression per cell type with limma (model.matrix ~0+type, lmFit, makeContrasts typeCase-typeControl, eBayes) after removing genes detected in fewer than 3 cells, keeping only clusters with more than min.cells=3 cells per group. Build a tissue-specific drug reference from LINCS L1000 GSE70138/GSE92742 perturbational profiles with PrepareReference/GetDrugRef (requires >64 GB RAM), then run GetDrug per cluster with repurposing.unit='drug', connectivity='negative', drug.type='FDA', which applies a reversed-signature outlier-sum statistic with a K-S test and BH FDR. Finally call DrugScore with the integrated cell metadata and case labels to aggregate per-cluster rankings into a patient-level multi-cluster drug score, rank drugs with FDR, and verify known positives (e.g. for severe COVID-19 BALF GSE145926, the ACE inhibitors rescinnamine and enalapril rank 2nd and 4th, Fig. 6d).

## Related tools

- **ASGARD** (Core package providing PrepareReference, GetDrugRef, GetDrug, DrugScore, TopDrug, and DrugCombination for signature-reversal drug ranking across cell clusters) — https://github.com/lanagarmire/ASGARD
- **limma** (Computes case-vs-control differential expression t-statistics fed to GetDrug; best AUC (0.90-0.92) among tested DE methods)
- **Seurat** (Normalizes, integrates, clusters, and annotates scRNA-seq data and supplies cell metadata for DrugScore aggregation)
- **cmapR** (Reads LINCS L1000 GCT perturbational matrices used to build the drug reference)
- **SingleR** (Annotates cell clusters (e.g. neutrophil, NK, T cell, monocyte) against HumanPrimaryCellAtlasData)

## Examples

```
Drug.ident.res <- GetDrug(gene.data = Gene.list, drug.ref.profiles = drug.ref.profiles, repurposing.unit = 'drug', connectivity = 'negative', drug.type = 'FDA')
```

## Evaluation signals

- For COVID-19 BALF (GSE145926 deceased vs cured), rescinnamine must rank 2nd and enalapril 4th in the multi-cluster drug score, and both must be ACE inhibitors (Fig. 6d)
- Multi-cluster DrugScore AUC must significantly exceed (P < 0.05, Student's t test) predictions from any individual cluster
- With limma DE input, average AUC across breast cancer, leukemia, and COVID-19 datasets should fall in the reported ~0.90-0.92 range
- GetDrug output must contain connectivity='negative' drug rankings and concordantly expressed gene sets for every cluster passing min.cells=3
- Drug reference construction completes on the GSE70138/GSE92742 GCT files and yields the tissue-specific rankMatrix (takes >1 hour on RAM >64 GB)

## Limitations

- Building the drug reference from LINCS L1000 requires >64 GB RAM and over one hour; the reference covers only 98 cell lines, and a proper cell-type-specific reference may not exist for a target disease
- Systematic differences between cell lines and primary tissues are not corrected; transfer-learning adjustment is proposed but not demonstrated
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, and DE method choice (limma, Seurat, DESeq2, edgeR) materially affects downstream AUC
- Obtaining proper normal-tissue controls may not be feasible for some diseases

## Evidence

- [abstract] ASGARD defines a drug score to recommend drugs by considering all cell clusters: "we propose A Single-cell Guided Pipeline to Aid Repurposing of Drugs (ASGARD) that defines a drug score to recommend drugs by considering all cell clusters"
- [discussion] The Kolmogorov–Smirnov test is applied to the outlier-sum statistic: "The Kolmogorov–Smirnov test (K-S test) is then applied to the OS statistic"
- [methods] GetDrug call signature with repurposing.unit, connectivity, and drug.type parameters: "Drug.ident.res = GetDrug(gene.data = Gene.list,"
- [readme] PrepareReference builds drug references from LINCS L1000 GSE70138/GSE92742 files, taking over an hour with RAM>64GB: "Please note that it takes more than one hour to produce drug references in a standard computer with RAM>64GB."
- [discussion] Drug reference is limited to 98 cell lines from LINCS L1000: "built the drug reference using drug responses data from LINCS L1000 project21, which were collected from 98 cell lines"
