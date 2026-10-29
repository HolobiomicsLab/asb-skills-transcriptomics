---
name: pdx-gene-expression-classification
description: Use when gene expression data from PDX models, cell lines, or organoids
  needs CMS classification without tumor stroma, or when assessing how sensitive subtype
  calls are to template construction choices.
when_to_use_negative:
- Classifying samples with a human stromal component different from primary CRCs,
  e.g., patient biopsies of metastatic CRC — CMScaller's current implementation is
  not recommended for these
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller
  is not optimized
- Very small datasets (fewer than approximately forty samples), where the required
  centering and scaling inescapably introduces prediction uncertainty
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: CMScaller
  role: Performs nearest-template prediction (NTP) CMS classification of pre-clinical
    models using cancer cell-intrinsic subtype-enriched marker templates
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment for building the 1+4 templates, running classification, computing
    pairwise concordance, and writing the report CSV
- name: Biobase
  role: Loads the PDX gene expression dataset into an ExpressionSet for classification
    input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:339a26588afecde5540fe772dc0154fe2509f4027935833d96383c301d9f8299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pdx-gene-expression-classification

## Summary

Classifies colorectal cancer PDX gene expression samples into Consensus Molecular Subtypes (CMS1-CMS4) using CMScaller nearest-template prediction (NTP), and probes classification robustness by comparing the reference template against alternative templates varying marker-selection cutoff, cell line panel, LFC threshold, and gene panel.

## When to use

Use when gene expression data from PDX models, cell lines, or organoids needs CMS classification without tumor stroma, or when assessing how sensitive subtype calls are to template construction choices.

## When NOT to use

- Classifying samples with a human stromal component different from primary CRCs, e.g., patient biopsies of metastatic CRC — CMScaller's current implementation is not recommended for these
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller is not optimized
- Very small datasets (fewer than approximately forty samples), where the required centering and scaling inescapably introduces prediction uncertainty

## Inputs

- PDX (or cell line/organoid) gene expression matrix (e.g., Gao et al. PDX dataset) as a gene-by-sample matrix loadable into a Biobase ExpressionSet
- CMScaller reference prediction template and four alternative templates (varying p-value cutoff 1e-4, cell line panel, LFC threshold of 1, Isella et al. gene panel losses)
- R environment with CMScaller, Biobase, and limma installed

## Outputs

- Per-sample CMS predictions (CMS1-CMS4) from each of the five templates
- Pairwise classification concordance matrix across the five templates (values in [0, 1])
- Count of samples consistently assigned to a single CMS by all templates
- cms_concordance_report.csv containing pairwise concordance entries and consistent-assignment counts

## How to apply

Build a reference CMScaller template plus four alternative templates in R, each varying one parameter: stricter adjusted p-value cutoff of 1e-4 for marker selection, an alternative cell line panel for template derivation, an LFC threshold of 1, and removal of genes lost between the Isella et al. panel and the primary template. Load the PDX expression matrix (e.g., Gao et al., 50 samples) into an ExpressionSet via Biobase, and center and scale the input expression matrix as required for nearest-template prediction since CMScaller's NTP core (Hoshida 2010) depends on template-relative distances. Classify the samples with each template using CMScaller, then compute pairwise classification concordance across the five sets of CMS assignments in R. Count samples assigned to a single consistent CMS by all templates, and write pairwise concordance and consistent-assignment counts to a report CSV. Validate that concordance values lie in [0, 1] and the consistent count is non-negative; in the Gao et al. PDX application, pairwise concordance was 0.95-0.98 with 47/50 samples (94%) consistent.

## Related tools

- **CMScaller** (Performs nearest-template prediction (NTP) CMS classification of pre-clinical models using cancer cell-intrinsic subtype-enriched marker templates) — https://github.com/Lothelab/CMScaller
- **R** (Environment for building the 1+4 templates, running classification, computing pairwise concordance, and writing the report CSV)
- **Biobase** (Loads the PDX gene expression dataset into an ExpressionSet for classification input)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(pdx_eset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- cms_concordance_report.csv exists and contains pairwise concordance entries for all five template pairs with values in [0, 1]
- Report includes a non-negative consistent-assignment count; for Gao et al. PDX, expect concordance of 0.95-0.98 and 47/50 samples (94%) consistently assigned
- Expression matrix is centered and scaled before NTP, as CMScaller requires
- Predictions cover all samples in the ExpressionSet and fall into valid CMS classes (CMS1-CMS4)
- Each alternative template differs from the reference by exactly one parameter (p-value cutoff, cell line panel, LFC threshold, or gene panel)

## Limitations

- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, including patient biopsies of metastatic CRC
- Centering and scaling of input data introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples
- Not optimized for single-cell RNA-sequencing data; classification errors may arise at class boundaries from intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic)

## Evidence

- [readme] The core algorithm is Nearest Template Prediction (NTP) algorithm proposed by Yujin Hoshida: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida"
- [abstract] CMScaller recapitulated subtypes in 551 in vitro and in vivo models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Input gene expression data must be centered and scaled: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [discussion] CMScaller not recommended for samples with different human stromal component than primary CRCs: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [readme] CMS classification of TCGA colorectal cancers via CMScaller function call: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
