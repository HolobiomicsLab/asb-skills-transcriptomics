---
name: nearest-template-prediction
description: Use when you need to classify pre-clinical colorectal cancer models (cell
  lines, patient-derived organoids, xenografts) into CMS1–CMS4 subtypes from a gene
  expression matrix, since the original CMS classifier fails on these stroma-free
  samples.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g. biopsies
  of metastatic CRC — CMScaller is explicitly not recommended there
- Small datasets with fewer than approximately forty samples, where required centering/scaling
  inescapably introduces prediction uncertainty
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller
  is not optimized
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: CMScaller
  role: Provides the CMScaller() NTP classifier, CMSgsa, subDEG, subVolcano functions
    and templates.CMS reference used to assign and evaluate CMS labels
  repo: github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment for the CMScaller package and downstream concordance/metric
    computations
- name: Biobase
  role: Holds the input expression data as an ExpressionSet within the Bioconductor-based
    workflow
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:5e4eda5f178d59d97a792af96fec62a5cb9331337ee24d3d66607d0ae7c153e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nearest-template-prediction

## Summary

Applies CMScaller's Nearest Template Prediction (NTP) algorithm (Hoshida 2010) to assign CMS1–CMS4 labels to colorectal cancer pre-clinical models (cell lines, organoids, PDXs) by comparing centered/scaled expression matrices against CMS-informative template gene sets.

## When to use

Use when you need to classify pre-clinical colorectal cancer models (cell lines, patient-derived organoids, xenografts) into CMS1–CMS4 subtypes from a gene expression matrix, since the original CMS classifier fails on these stroma-free samples.

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g. biopsies of metastatic CRC — CMScaller is explicitly not recommended there
- Small datasets with fewer than approximately forty samples, where required centering/scaling inescapably introduces prediction uncertainty
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller is not optimized

## Inputs

- Gene expression matrix (e.g. TCGA pCRC, Gao et al. PDX models, cell lines) as an R matrix or Biobase ExpressionSet
- CMS template gene sets (shipped templates.CMS or custom-built from marker selection with specified p-value/LFC thresholds)
- Boolean flag for data type (RNAseq=TRUE for RNA-seq counts)

## Outputs

- Per-sample CMS prediction table (CMS1–CMS4 labels with NTP confidence, e.g. res$prediction)
- Camera gene set analysis results per subtype (CMSgsa output, e.g. cam$CMS4)
- Differential expression results and volcano plots (subDEG / subVolcano)
- Concordance reports when multiple templates are compared (e.g. cms_concordance_report.csv)

## How to apply

Install CMScaller from GitHub (devtools::install_github('Lothelab/CMScaller')) with Biobase and limma dependencies. Prepare the input gene expression matrix and center and scale it, as required for nearest-template prediction; for RNA-seq data set RNAseq=TRUE. Run CMScaller::CMScaller() on the ExpressionSet or matrix, using the shipped templates.CMS reference (cancer cell-intrinsic subtype-enriched markers) so NTP assigns each sample the nearest CMS template; doPlot=TRUE visualizes prediction confidence. Templates can be rebuilt or varied (e.g. stricter adjusted p-value cutoff of 1e-4 for marker selection, alternative cell line panel, LFC threshold of 1) to test classification robustness. Classification of small datasets (fewer than ~40 samples) is unreliable because centering/scaling introduces prediction uncertainty. Optionally follow up with CMSgsa (Camera gene set analysis) and subDEG/subVolcano to evaluate and visualize the assigned classes.

## Related tools

- **CMScaller** (Provides the CMScaller() NTP classifier, CMSgsa, subDEG, subVolcano functions and templates.CMS reference used to assign and evaluate CMS labels) — github.com/Lothelab/CMScaller
- **R** (Runtime environment for the CMScaller package and downstream concordance/metric computations)
- **Biobase** (Holds the input expression data as an ExpressionSet within the Bioconductor-based workflow)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE); head(res)
```

## Evaluation signals

- On the held-out TCGA pCRC test set (n = 143), NTP-assigned labels against original CMS classifications should yield overall accuracy ~0.83 (95% CI 0.74–0.9), with CMS3 sensitivity 0.71 and CMS4 specificity 0.91
- Across the 1+4 alternative templates on the 50 Gao et al. PDX models, pairwise concordance should be 0.95–0.98 with 47/50 samples (94%) assigned the same CMS by all templates
- All predicted labels must fall within the CMS1–CMS4 class set; pairwise concordance values must lie in [0, 1]
- Verify the input expression matrix was centered and scaled before prediction, and that RNAseq=TRUE was set for RNA-seq input
- Validation of output files: pairwise concordance entries present for all five templates and a non-negative consistent-assignment count

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Not recommended for samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC; caution is warranted in highly selected datasets whose molecular distributions deviate from pCRC cohorts
- Not optimized for single-cell RNA-sequencing data, which is exceedingly noisy; intra-tumor heterogeneity may explain prediction errors at class boundaries

## Evidence

- [readme] The core algorithm is Nearest Template Prediction (NTP) algorithm proposed by Yujin Hoshida [Hoshida 2010].: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida [Hoshida 2010]."
- [abstract] The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group.: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] CMScaller recapitulated the subtypes in both in vitro and in vivo models (551 in total).: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Input gene expression data should be centered and scaled, introducing prediction uncertainty for small datasets.: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [discussion] CMScaller is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies.: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
