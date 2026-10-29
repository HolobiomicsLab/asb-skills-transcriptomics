---
name: cms-prediction-template-generation
description: Use when CMS-classifying pre-clinical CRC expression data (PDX, cell
  lines, organoids) and you need to derive NTP templates or assess how robust CMScaller
  assignments are to template-derivation thresholds and marker-selection choices.
when_to_use_negative:
- Classifying samples with a human stromal component different from primary CRCs,
  e.g. patient biopsies of metastatic CRC — the article states CMScaller is not recommended
  for such samples
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller
  is not optimized
- Datasets with fewer than approximately forty samples — centering and scaling input
  expression data inescapably introduces prediction uncertainty for small datasets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: Provides nearest-template prediction (NTP) classification of samples into
    CMS1–CMS4 using the generated subtype-enriched marker templates, and functions
    for evaluating and visualizing results
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Language in which the reference and alternative templates are built and in
    which pairwise concordance and consistent-assignment counts are computed
- name: Biobase
  role: Used to load the target gene expression dataset into an ExpressionSet for
    CMScaller input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:6144997543b11f945b625391ff816e1a235063d0f9e41e566b41b0c561a6ca62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cms-prediction-template-generation

## Summary

Generate and stress-test CMScaller nearest-template prediction (NTP) templates for consensus molecular subtyping (CMS) of colorectal cancer pre-clinical models by building a reference template plus 1+4 alternative templates, each varying one derivation parameter (adjusted p-value cutoff 1e-4, alternative cell line panel, LFC threshold of 1, removal of genes lost between the Isella et al. panel and the primary template).

## When to use

Use when CMS-classifying pre-clinical CRC expression data (PDX, cell lines, organoids) and you need to derive NTP templates or assess how robust CMScaller assignments are to template-derivation thresholds and marker-selection choices.

## When NOT to use

- Classifying samples with a human stromal component different from primary CRCs, e.g. patient biopsies of metastatic CRC — the article states CMScaller is not recommended for such samples
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller is not optimized
- Datasets with fewer than approximately forty samples — centering and scaling input expression data inescapably introduces prediction uncertainty for small datasets

## Inputs

- Gene expression matrix for pre-clinical colorectal cancer models (e.g. the Gao et al. PDX dataset), loaded as a Biobase ExpressionSet
- Cell line panel(s) with known CMS labels used to derive subtype-enriched marker templates (primary panel plus one alternative panel)
- Template-derivation parameters: adjusted p-value cutoff (1e-4 alternative), LFC threshold (1 as alternative), gene-lost filter relative to the Isella et al. panel

## Outputs

- Five sets of CMS assignments (CMS1–CMS4 per sample) from the reference plus four alternative NTP templates
- Pairwise classification concordance matrix across the five templates with values in [0, 1]
- Count of samples consistently assigned to a single CMS by all templates (e.g. 47/50 = 94% in the Gao et al. PDX models), written to cms_concordance_report.csv

## How to apply

Build a reference CMScaller template and four alternative prediction templates in R, deliberately varying exactly one derivation parameter per alternative: a stricter adjusted p-value cutoff of 1e-4 for marker selection, an alternative cell line panel for template derivation, a log-fold-change (LFC) threshold of 1, and removal of genes lost between the Isella et al. panel and the primary template. Load the target expression dataset (e.g. the Gao et al. PDX models) as an ExpressionSet via Biobase and center and scale the expression matrix, which is required for nearest-template prediction. Classify samples with each of the five templates using CMScaller NTP, then compute pairwise classification concordance across templates and count samples assigned to a single CMS by all templates. Rationale: CMScaller relies on cancer cell-intrinsic subtype-enriched markers precisely because the original CMS classifier depends on immune and stromal signals and fails to identify CMS1/CMS4 in cell lines, organoids and xenografts; concordance of 0.95–0.98 (47/50 samples, 94%) across templates demonstrates insensitivity to the varied parameters.

## Related tools

- **CMScaller** (Provides nearest-template prediction (NTP) classification of samples into CMS1–CMS4 using the generated subtype-enriched marker templates, and functions for evaluating and visualizing results) — https://github.com/Lothelab/CMScaller
- **R** (Language in which the reference and alternative templates are built and in which pairwise concordance and consistent-assignment counts are computed)
- **Biobase** (Used to load the target gene expression dataset into an ExpressionSet for CMScaller input)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Pairwise classification concordance across the five templates falls in [0, 1], with the reference case reaching 0.95–0.98 on the Gao et al. PDX models
- Consistent-assignment count is non-negative and, when reproducing the reference result, 47/50 samples (94%) receive the same CMS from all templates
- Output file cms_concordance_report.csv exists and contains pairwise concordance entries for all five templates
- Each input expression matrix is centered and scaled before NTP prediction, as required by CMScaller
- Each alternative template differs from the reference by exactly one derivation parameter (adjusted p-value 1e-4, alternative cell line panel, LFC threshold 1, or removal of genes lost from the Isella et al. panel)

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries
- The prediction template gene set is planned to be further improved using larger, higher-quality datasets as they become available

## Evidence

- [readme] NTP algorithm basis: [evidence span withheld — anchor status: not_found]
- [abstract] CMScaller recapitulates subtypes in pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [intro] Original classifier fails on pre-clinical models: "it was recently shown that it fails to identify the CMS1-immune and CMS4-mesenchymal groups in cell lines,"
- [discussion] Small-dataset uncertainty caveat: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [discussion] Metastatic biopsy caveat: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [readme] R quick-start invocation: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
