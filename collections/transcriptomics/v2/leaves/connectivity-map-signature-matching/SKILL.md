---
name: connectivity-map-signature-matching
description: Use when cluster- or patient-level DEG lists from scRNA-seq case-vs-control
  contrasts must be converted into ranked drug candidates by comparing against L1000
  Level5 perturbational profiles (GSE70138/GSE92742 gctx).
when_to_use_negative:
- No tissue-appropriate reference exists — the drug reference is limited to LINCS
  L1000 profiles from only 98 cell lines, so unrepresented tissues need substitutes
  or transfer learning first
- No normal/control samples are obtainable for the disease, since the case-vs-control
  signature reversal requires a control contrast
- The goal is combination therapy scoring rather than single-drug ranking — use the
  separate DrugCombination step instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_3308
tools:
- name: ASGARD
  role: Provides PrepareReference, GetDrugRef, GetDrug, and DrugScore functions that
    build the L1000-derived reference and perform connectivity-based signature matching
  repo: https://github.com/lanagarmire/Asgard
- name: cmapR
  role: Reads the GSE70138/GSE92742 gctx Level5 matrices during reference preparation
- name: limma
  role: Generates the DEG signatures fed into the connectivity matching; its Bayes
    method showed the best average AUC among tested DE methods
- name: Seurat
  role: Supplies the integrated scRNA-seq object, per-cluster metadata, and cell-level
    DE gene lists that drive the matching
- name: pROC
  role: Computes ROCs and AUCs for each pipeline using drugs in advanced clinical
    trials as positives
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:cf010df25bd2b9f6417f81e987a640f4f9683c502add0799b2e017c7bd4e15be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - Seurat
  - SingleR
  - cmapR
  - limma
  - pROC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# connectivity-map-signature-matching

## Summary

Matches disease-derived differential expression signatures against LINCS L1000 Connectivity Map perturbation profiles to rank repurposed drugs, reversing the disease signature (connectivity='negative') using an outlier-sum statistic assessed by Kolmogorov–Smirnov test with BH FDR correction.

## When to use

Use when cluster- or patient-level DEG lists from scRNA-seq case-vs-control contrasts must be converted into ranked drug candidates by comparing against L1000 Level5 perturbational profiles (GSE70138/GSE92742 gctx).

## When NOT to use

- No tissue-appropriate reference exists — the drug reference is limited to LINCS L1000 profiles from only 98 cell lines, so unrepresented tissues need substitutes or transfer learning first
- No normal/control samples are obtainable for the disease, since the case-vs-control signature reversal requires a control contrast
- The goal is combination therapy scoring rather than single-drug ranking — use the separate DrugCombination step instead

## Inputs

- Case-vs-control DEG list or per-cluster DE statistics from Seurat/limma/DESeq2/edgeR (e.g. typeCase - typeContrast contrast, min 3 cells per group)
- LINCS L1000 Level5 gctx profiles: GSE70138_Broad_LINCS_Level5_COMPZ_n118050x12328.gctx and GSE92742_Broad_LINCS_Level5_COMPZ.MODZ_n473647x12328.gctx
- Accompanying LINCS cell_info, sig_info, and gene_info text tables
- Integrated Seurat object with cell metadata and cluster assignments

## Outputs

- Tissue-specific drug reference matrices (e.g. breast_rankMatrix.txt with gene_info and drug_info)
- Per-cluster ranked drug list with connectivity scores and Drug.FDR (GetDrug output)
- Patient-level multi-cluster DrugScore table with Drug.therapeutic.score and FDR
- Significant FDA-drug candidates via TopDrug (e.g. 11 drugs with fostamatinib top-1 for PDX breast cancer)

## How to apply

First build a tissue-specific drug reference with Asgard's PrepareReference on the GSE70138 and GSE92742 Broad LINCS Level5 gctx files plus their cell/gene/sig_info tables (requires RAM > 64 GB, over one hour), then load it via GetDrugRef on e.g. DrugReference/breast_rankMatrix.txt. For each cell cluster, feed its DEG statistics (best from limma Bayes on model.matrix(~0+type) contrasts) into GetDrug with repurposing.unit='drug' and connectivity='negative', which reverses the differential gene pattern and scores drugs by an outlier-sum statistic tested with the Kolmogorov–Smirnov test and BH FDR. For heterogeneous patients, aggregate per-cluster rankings into the multi-cluster DrugScore across all clusters before selecting significant drugs (Drug.therapeutic.score > 0.99 quantile, FDR < 0.05; per-cluster TopDrug with Drug.FDR=0.1, FDA.drug.only=TRUE). Benchmark rankings with ROC/AUC in pROC, treating drugs in advanced clinical trials/animal models (Supplementary Data 4) as positive cases; the multi-cluster drug score significantly outperforms cluster-only prediction (P<0.05, Student's t-test).

## Related tools

- **ASGARD** (Provides PrepareReference, GetDrugRef, GetDrug, and DrugScore functions that build the L1000-derived reference and perform connectivity-based signature matching) — https://github.com/lanagarmire/Asgard
- **cmapR** (Reads the GSE70138/GSE92742 gctx Level5 matrices during reference preparation)
- **limma** (Generates the DEG signatures fed into the connectivity matching; its Bayes method showed the best average AUC among tested DE methods)
- **Seurat** (Supplies the integrated scRNA-seq object, per-cluster metadata, and cell-level DE gene lists that drive the matching)
- **pROC** (Computes ROCs and AUCs for each pipeline using drugs in advanced clinical trials as positives)

## Examples

```
Drug.ident.res <- GetDrug(gene.data = Gene.list, drug.ref.profiles = GetDrugRef(drug.response.path = 'DrugReference/breast_rankMatrix.txt'), repurposing.unit = 'drug', connectivity = 'negative')
```

## Evaluation signals

- Breast PDX run yields 11 significant drugs with fostamatinib ranked top-1 and colchicine second
- TNBC patient run yields 13 significant drugs overlapping PDX predictions on mebendazole, crizotinib, neratinib, and vinblastine
- Averaged single-cluster AUC across 8 breast cancer clusters ≈ 0.83 (0.80–0.86), beating Alakwaa's (0.76) and Guo's (0.54) CLUE-based pipelines at P<0.05 (Student's t-test)
- Multi-cluster DrugScore AUC significantly exceeds per-cluster prediction (P-value < 0.05, Student's t test)
- Limma-based signatures should give the best average AUC among DE methods (limma, Seurat Wilcoxon, DESeq2, edgeR)

## Limitations

- Drug reference covers only 98 cell lines from LINCS L1000; systematic cell-line-vs-primary-tissue differences are not adjusted for
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, propagating into signature matches
- DE method choice affects downstream AUC and must be selected per dataset
- Quantitative AUC values are only in figures; positive-case ground truth lives in Supplementary Data 4

## Evidence

- [discussion] L1000 reference construction: "ASGARD built the drug reference using drug responses data from LINCS L1000 project21, which were collected from 98 cell lines"
- [discussion] K-S scoring statistic: "The Kolmogorov–Smirnov test (K-S test) is then applied to the OS statistic"
- [intro] CLUE baseline framing: "predicts candidate drugs for DEGs using the Connectivity Map Linked User Environment (CLUE) platform"
- [readme] Reference preparation from gctx files: "Download L1000 Connectivity Map perturbational profiles GSE70138 and GSE92742 from GEO"
- [methods] GetDrug entry point: "Drug.ident.res = GetDrug(gene.data = Gene.list,"
- [discussion] Benchmarking: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
