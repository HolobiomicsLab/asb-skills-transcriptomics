---
name: bulk-rna-seq-subtype-classification
description: Use when bulk RNA-seq tumor expression matrices must be classified into
  known molecular subtypes using single-cell-derived gene signatures, and when subtype
  survival differences (e.g., basal-like vs. classical in PAAD) need to be quantified.
when_to_use_negative:
- The input is single-cell data rather than bulk RNA-seq — SCISSORS' native scope
  is reclustering single cells within broad Seurat clusters, not scoring bulk samples.
- Samples already carry trusted subtype labels from the reference schema, making signature-based
  rescoring redundant.
- No subtype gene signatures (or no overlap with the Moffitt reference genes) are
  available to anchor the fold-change-ranked marker lists.
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCISSORS
  role: Derives highly specific subtype marker gene signatures via iterative semi-supervised
    reclustering of heterogeneous clusters, providing the gene lists applied to bulk
    data
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell infrastructure (NormalizeData, FindVariableFeatures
    vst/2000 features, ScaleData, RunPCA) used to preprocess scRNA-seq before signature
    derivation
- name: cluster
  role: Provides clustering tools (e.g., silhouette computation) used to estimate
    cluster heterogeneity during reclustering
- name: dplyr
  role: Tidy data manipulation for joining signature scores, subtype calls, and survival
    metadata
- name: ggplot2
  role: Plotting Kaplan-Meier survival curves and visualizations of per-tumor scores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:4c54a68e07cf5621005d843ab1893de51ebebc171df7c417f17c48807239d181
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bulk-rna-seq-subtype-classification

## Summary

Assigns bulk RNA-seq tumor samples to molecular subtypes by scoring each sample against single-cell-derived (SCISSORS) marker gene signatures and picking the higher-scoring signature, then tests subtype-associated survival with Kaplan-Meier curves and a log-rank test. Validated on TCGA PAAD (150 tumors) against the Moffitt basal-like/classical schema.

## When to use

Use when bulk RNA-seq tumor expression matrices must be classified into known molecular subtypes using single-cell-derived gene signatures, and when subtype survival differences (e.g., basal-like vs. classical in PAAD) need to be quantified.

## When NOT to use

- The input is single-cell data rather than bulk RNA-seq — SCISSORS' native scope is reclustering single cells within broad Seurat clusters, not scoring bulk samples.
- Samples already carry trusted subtype labels from the reference schema, making signature-based rescoring redundant.
- No subtype gene signatures (or no overlap with the Moffitt reference genes) are available to anchor the fold-change-ranked marker lists.

## Inputs

- Bulk tumor RNA-seq expression matrix (e.g., TCGA PAAD, 150 tumors, gene x sample)
- Top-10 fold-change-ranked SCISSORS basal-like and classical gene signatures derived from scRNA-seq reclustering
- Reference subtype calls for concordance checking (e.g., Moffitt schema labels per sample)
- Per-sample survival metadata (time-to-event and censoring status) for Kaplan-Meier analysis

## Outputs

- Per-tumor subtype calls for all 150 samples (basal-like vs. classical)
- Concordance table versus the Moffitt schema (target: 139/150 consistent; 33 basal-like, 106 classical)
- Kaplan-Meier survival curves comparing basal-like vs. classical patients
- Log-rank test P-value and hazard ratio (reported: P = .025, HR = 1.723)

## How to apply

Derive subtype signatures from scRNA-seq by running SCISSORS' iterative reclustering on heterogeneous clusters, ranking marker genes by fold change, and keeping the top-10 genes per subtype anchored by their overlap with reference (Moffitt) genes. Load the bulk expression matrix (e.g., TCGA PAAD, 150 tumors), score each tumor against both signatures, and assign the subtype of the higher-scoring signature. Tabulate the per-tumor calls and compute concordance with the published schema, targeting 139/150 consistent calls (33 basal-like, 106 classical). Fit Kaplan-Meier curves for the two subtypes with survival/ggplot2 and evaluate separation via log-rank test, reporting hazard ratio (expected: P = .025, HR = 1.723 for shorter basal-like survival).

## Related tools

- **SCISSORS** (Derives highly specific subtype marker gene signatures via iterative semi-supervised reclustering of heterogeneous clusters, providing the gene lists applied to bulk data) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell infrastructure (NormalizeData, FindVariableFeatures vst/2000 features, ScaleData, RunPCA) used to preprocess scRNA-seq before signature derivation)
- **cluster** (Provides clustering tools (e.g., silhouette computation) used to estimate cluster heterogeneity during reclustering)
- **dplyr** (Tidy data manipulation for joining signature scores, subtype calls, and survival metadata)
- **ggplot2** (Plotting Kaplan-Meier survival curves and visualizations of per-tumor scores)

## Examples

```
seu_pbmc <- SeuratData::LoadData("pbmc3k") %>% PrepareData(n.HVG = 4000, n.PC = 15, which.dim.reduc = "umap", initial.resolution = 0.4, random.seed = 629)
```

## Evaluation signals

- Subtype calls achieve 139/150 (92.7%) concordance with Moffitt schema calls on TCGA PAAD, with 33 basal-like and 106 classical tumors
- Log-rank test on basal-like vs. classical Kaplan-Meier curves yields P = .025 with hazard ratio = 1.723 (basal-like shorter survival)
- Every one of the 150 tumors receives exactly one subtype call (higher-scoring signature wins; no unassigned samples)
- Top-10 signature genes per subtype show overlap with Moffitt anchor genes, confirming the signatures were correctly derived
- Silhouette scores of the source scRNA-seq clusters justify which clusters were targeted for reclustering before marker extraction

## Limitations

- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, which propagates into the derived signatures and downstream bulk calls
- The paper's discussion section was unavailable in the provided context, so no additional limitations or reproducibility gaps could be extracted from the discussion

## Evidence

- [abstract] SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Marker gene identification for signatures: "Additionally, SCISSORS provides a method for the identification of marker genes of high specificity to the cell type"
- [readme] Package scope statement: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine built as an"
- [methods] Preprocessing parameters used before clustering/signature derivation: "FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
