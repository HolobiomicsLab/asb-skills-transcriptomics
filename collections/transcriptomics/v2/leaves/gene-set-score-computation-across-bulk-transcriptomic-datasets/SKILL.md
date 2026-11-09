---
name: gene-set-score-computation-across-bulk-transcriptomic-datasets
description: Use when you have bulk genes-by-samples expression matrices (microarray
  or RNAseq, e.g. pdacR $ex objects) with sample-level tissue metadata, and you need
  per-sample gene-set scores to compare signature activity across compartments like
  normal pancreas, primary PDAC, PDX, CAF lines, and metastases.
when_to_use_negative:
- The goal is tumor-cell-intrinsic subtyping (e.g. PurIST or basal-like/classical
  classification) rather than attributing signature expression to non-tumor tissue
  compartments — normal pancreas and CAF contamination will inflate ADEX/Exocrine
  scores in tumor datasets
- Input is single-cell data (e.g. the rmoffitt/scOh Seurat .rds atlas) rather than
  bulk $ex matrices — scoring conventions and grouping by tissue-of-origin tracks
  do not transfer directly
- Datasets lack sample-level tissue-of-origin metadata in sampInfo, making the Wilcoxon
  group comparisons undefined
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: Source of the parsed Moffitt 2015 microarray/RNAseq expression matrices ($ex),
    sampInfo tissue-of-origin metadata, and curated gene signature objects used for
    scoring
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Language in which datasets are loaded, genes are z-scored, signature scores
    computed, and Wilcoxon rank-sum tests run
- name: devtools
  role: Used to install the pdacR package from GitHub so its parsed datasets and gene
    lists become available locally
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:2549161fb80987edd7a1c748842f65db2d371172c5a3c2735ad0e3b24fe3aec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gene-set-score-computation-across-bulk-transcriptomic-datasets

## Summary

Computes per-sample gene signature scores (ADEX, Exocrine, Immunogenic) in bulk PDAC expression datasets by z-scoring each gene and averaging across signature members, then testing score differences between tissue-of-origin groups with Wilcoxon rank-sum tests to attribute signature expression to specific tissue or cell types.

## When to use

Use when you have bulk genes-by-samples expression matrices (microarray or RNAseq, e.g. pdacR $ex objects) with sample-level tissue metadata, and you need per-sample gene-set scores to compare signature activity across compartments like normal pancreas, primary PDAC, PDX, CAF lines, and metastases.

## When NOT to use

- The goal is tumor-cell-intrinsic subtyping (e.g. PurIST or basal-like/classical classification) rather than attributing signature expression to non-tumor tissue compartments — normal pancreas and CAF contamination will inflate ADEX/Exocrine scores in tumor datasets
- Input is single-cell data (e.g. the rmoffitt/scOh Seurat .rds atlas) rather than bulk $ex matrices — scoring conventions and grouping by tissue-of-origin tracks do not transfer directly
- Datasets lack sample-level tissue-of-origin metadata in sampInfo, making the Wilcoxon group comparisons undefined

## Inputs

- Parsed genes-by-samples expression matrices from pdacR data/ (.RData objects), accessed via the $ex slot, for both the Moffitt 2015 microarray and RNAseq datasets
- Sample-level metadata (sampInfo) annotating tissue of origin: primary PDAC, PDX, CAF line, cell line, normal pancreas, metastasis
- Curated gene signature lists (gene SYMBOLs) for ADEX, Exocrine, and Immunogenic sets built by R/parse_gene_lists.R

## Outputs

- adex_exocrine_immunogenic_scores.csv containing per-sample signature scores plus Wilcoxon statistics and p-values
- Grouped boxplots of signature scores by tissue type for each dataset
- Validation verdict per dataset: pass if normal pancreata show significantly higher ADEX/Exocrine/Immunogenic scores than PDACs and primaries higher than CAF lines at p < 0.0001

## How to apply

Load the parsed expression datasets from the pdacR data/ directory (parsed .RData objects) together with their sampInfo sample metadata annotating tissue of origin, and load curated gene signature lists from R/parse_gene_lists.R, ensuring identifiers are gene SYMBOLs and selecting the ADEX, Exocrine, and Immunogenic sets. For each dataset, access the $ex genes-by-samples matrix, z-score (mean-center and scale) every gene, and compute each signature score per sample as the mean scaled expression of that set's genes. Group samples by their tissue-of-origin sample track and run Wilcoxon rank-sum tests per signature per dataset (e.g. normal pancreas vs PDAC tumors, primary PDAC vs CAF lines). Write per-sample scores plus Wilcoxon statistics and p-values to adex_exocrine_immunogenic_scores.csv and render grouped boxplots of signature scores by tissue type. Rationale: the two-subtype PDAC model is confounded by stromal and normal tissue signals in bulk data, so attributing signatures to tissue compartments requires scoring them across all profiled compartments rather than tumors alone.

## Related tools

- **pdacR** (Source of the parsed Moffitt 2015 microarray/RNAseq expression matrices ($ex), sampInfo tissue-of-origin metadata, and curated gene signature objects used for scoring) — https://github.com/rmoffitt/pdacR.git
- **R** (Language in which datasets are loaded, genes are z-scored, signature scores computed, and Wilcoxon rank-sum tests run)
- **devtools** (Used to install the pdacR package from GitHub so its parsed datasets and gene lists become available locally)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); ex <- scale(pdacR::moffitt_RNAseq$ex); scores <- colMeans(ex[adex_genes, , drop=FALSE])
```

## Evaluation signals

- Wilcoxon rank-sum tests give p < 0.0001 for normal pancreas vs PDAC tumor ADEX/Exocrine (and Immunogenic) score comparisons in BOTH the microarray and RNAseq datasets
- Primary PDAC vs CAF line comparisons show primaries with significantly higher signature scores at p < 0.0001 in both platforms
- Signature identifiers resolve as gene SYMBOLs matching rows of the $ex matrix (no dropped/NA genes after subsetting)
- adex_exocrine_immunogenic_scores.csv contains one row per sample with columns for all three signature scores plus Wilcoxon statistics and p-values, and row count matches the dataset sample count
- Boxplot groups correspond one-to-one with sampInfo tissue-of-origin labels (primary PDAC, PDX, CAF line, cell line, normal pancreas, metastasis)

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signature scores reflect mixed tumor/stroma/normal-epithelium signals rather than cell-type-specific expression
- Datasets come from multiple platforms (bulk RNA-seq and microarrays), so scores are computed and tested within each dataset separately rather than merged, because cross-platform aggregation and batch correction are unreliable
- Signature specificity can be compromised when low-purity or non-adenocarcinoma samples (e.g. non-PDAC cases in TCGA-PAAD, 182 samples vs 150 confirmed PDAC) are not filtered out before scoring

## Evidence

- [abstract] we have created pdacR ( http://pdacR.bmi.stonybrook.edu , github.com/rmoffitt/pdacR), an open-source software package and web-tool with annotated datasets: "we have created pdacR ( http://pdacR.bmi.stonybrook.edu , github.com/rmoffitt/pdacR), an open-source software package and web-tool with annotated datasets"
- [discussion] Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
- [discussion] our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction: "our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction"
- [readme] These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots.: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [readme] Checking a box in this list will remove all samples with that label from all analyses.: "Checking a box in this list will remove all samples with that label from all analyses."
- [methods] devtools::install_github("rmoffitt/pdacR"): "devtools::install_github("rmoffitt/pdacR")"
