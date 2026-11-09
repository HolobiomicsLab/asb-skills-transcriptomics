---
name: single-cell-rna-seq-preprocessing
description: Use when raw single-cell count matrices (e.g. cancer scRNA-seq from 10X
  or RDS) must be quality-filtered, normalized, and scaled before cell-type annotation,
  malignancy prediction, or feeding an XGBoost model in scCancer2.
when_to_use_negative:
- The input is already a normalized and scaled Seurat object or feature table rather
  than raw counts.
- The data are bulk RNA-seq (e.g. the TCGA training matrix side of scCancer2) rather
  than single-cell counts requiring per-cell QC.
- The analysis is spatial transcriptomics, which scCancer2 handles with a separate
  module rather than this scRNA-seq preprocessing path.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCancer
  role: R toolkit whose txt directory supplies the model template and sorted gene
    list and whose scAnnotation module consumes the preprocessed matrix
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Performs QC object creation, mito.percent computation, LogNormalize, vst variable-feature
    selection, and ScaleData regression used by this skill
- name: SoupX
  role: Optional upstream removal of ambient mRNA contamination from droplet-based
    single-cell data before preprocessing
  repo: https://github.com/constantAmateur/SoupX
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:0a81a776a8303d9cf30d1e211cd71bf766205ff9230bdf1783e5398d6c1a215d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - Seurat
  - SoupX
  - scCancer
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-rna-seq-preprocessing

## Summary

Standard Seurat-based preprocessing of tumor scRNA-seq count matrices used by scCancer2: quality filtering (min.genes=200, min.cells=3), mito.percent computation, LogNormalize at scale factor 10000, vst variable-feature selection up to 5000 features, and scaling with regression on nCount_RNA and mito.percent so the matrix can be aligned to a fixed gene list for downstream classifiers.

## When to use

Use when raw single-cell count matrices (e.g. cancer scRNA-seq from 10X or RDS) must be quality-filtered, normalized, and scaled before cell-type annotation, malignancy prediction, or feeding an XGBoost model in scCancer2.

## When NOT to use

- The input is already a normalized and scaled Seurat object or feature table rather than raw counts.
- The data are bulk RNA-seq (e.g. the TCGA training matrix side of scCancer2) rather than single-cell counts requiring per-cell QC.
- The analysis is spatial transcriptomics, which scCancer2 handles with a separate module rather than this scRNA-seq preprocessing path.

## Inputs

- Raw single-cell RNA-seq count matrix (e.g. 10X cellranger output or .RDS gene-by-cell counts)
- Sorted reference gene list (genes-scRNA-tcga-sorted.txt from the scCancer package txt directory)
- Cellranger output folder (for SoupX load10X ambient-RNA correction)

## Outputs

- Quality-filtered Seurat object with LogNormalize-scaled data and regressed nCount_RNA / mito.percent
- Variable-feature-scaled expression matrix (up to 5000 vst features) aligned to the sorted TCGA gene list with zeros for missing genes
- xgb.DMatrix ready for malignancy prediction

## How to apply

Build a Seurat object with CreateSeuratObject(counts=data, min.genes=200, min.cells=3) and compute mito.percent via PercentageFeatureSet(pattern='^MT-'). Normalize with NormalizeData(normalization.method='LogNormalize', scale.factor=10000), select variable features with FindVariableFeatures(selection.method='vst', nfeatures=min(5000, n_genes)), and scale with ScaleData(vars.to.regress=c('nCount_RNA','mito.percent')) to remove depth and mitochondrial-fraction effects. When the scaled matrix feeds the malignancy XGBoost classifier, align it to the package's sorted gene list ('genes-scRNA-tcga-sorted.txt' from the scCancer txt directory), filling genes absent from a sample with zeros. Optionally pre-clean droplet data with SoupX, since droplet experiments capture ambient mRNA at roughly 2-50% contamination; autoEstCont requires clustering information. The resulting log-normalized, scaled, gene-aligned matrix is converted to an xgb.DMatrix for prediction with a 0.5 malignant/nonMalignant threshold.

## Related tools

- **scCancer** (R toolkit whose txt directory supplies the model template and sorted gene list and whose scAnnotation module consumes the preprocessed matrix) — https://github.com/czythu/scCancer
- **Seurat** (Performs QC object creation, mito.percent computation, LogNormalize, vst variable-feature selection, and ScaleData regression used by this skill)
- **SoupX** (Optional upstream removal of ambient mRNA contamination from droplet-based single-cell data before preprocessing) — https://github.com/constantAmateur/SoupX

## Examples

```
object <- NormalizeData(object, normalization.method="LogNormalize", scale.factor=10000); object <- FindVariableFeatures(object, selection.method="vst", nfeatures=5000); object <- ScaleData(object, vars.to.regress=c("nCount_RNA","mito.percent"))
```

## Evaluation signals

- Cells with fewer than 200 genes are excluded (min.genes=200, min.cells=3 in CreateSeuratObject).
- mito.percent is populated for every cell using the '^MT-' PercentageFeatureSet pattern.
- Normalized values are log-scale with scale factor 10000, and the variable-feature set is capped at min(5000, number of genes).
- Scaled matrix columns match the sorted gene list exactly; genes absent from a sample are filled with zeros rather than dropped.
- Malignancy predictions on the preprocessed matrix use predict(model.ref, testdata) with the 0.5 threshold for malignant/nonMalignant labels.

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the article's methods.
- Ambient RNA contamination (if corrected with SoupX) varies hugely between experiments, roughly 2-50%, and autoEstCont assumes enough cellular diversity to find marker genes.
- No repository URL or accession IDs are provided for the HCC scRNA-seq datasets used to exercise this workflow.

## Evidence

- [methods] LogNormalize normalization: "object <- NormalizeData(object = object, normalization.method = "LogNormalize", scale.factor = 10000,"
- [methods] vst variable feature selection: "object <- FindVariableFeatures(object, selection.method = "vst", nfeatures = min(5000, length(rownames(object))), verbose = FALSE)"
- [methods] scaling with covariate regression: "object <- ScaleData(object, vars.to.regress = c("nCount_RNA", "mito.percent"), verbose = FALSE)"
- [methods] quality filtering thresholds: "object <- CreateSeuratObject(counts = data, min.genes = 200, min.cells = 3)"
- [methods] mitochondrial percentage computation: "object[["mito.percent"]] <- PercentageFeatureSet(object, pattern = "^MT-")"
- [readme] ambient RNA contamination rationale for upstream cleanup: [evidence span withheld — anchor status: not_found]
