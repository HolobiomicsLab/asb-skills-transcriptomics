---
name: gene-set-averaged-expression-scoring
description: Use when you need to translate a cell-state or pathway gene signature
  (e.g. a MetaTiME MeC weight table) into a per-sample numeric score in bulk RNA-seq
  data such as TCGA, for grouping samples into high/low signature classes.
when_to_use_negative:
- Input is already a per-cell or per-sample feature/score table — re-scoring would
  double-transform the data
- The analysis requires single-cell cell-state annotation rather than a bulk per-sample
  signature score (use the MetaTiME Annotator directly on scRNA-seq instead)
- The gene set has no pretrained weight table or clear top-gene ranking, since the
  method depends on selecting the top 20 z-weighted genes
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene-weight table and the mecmapper/plmapper modules
    used to extract top-20 signature genes and score bulk samples in the MeC space
  repo: github.com/yi-zhang/MetaTiME
- name: lifelines
  role: Python library used to fit Kaplan–Meier survival curves for high vs. low signature
    groups and compute log-rank tests
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:22e10f4cec242c0b1f4d774f1baf0ee724a6c4ca1a52437f95211df7205d8cfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  - lifelines
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gene-set-averaged-expression-scoring

## Summary

Scores bulk or single-cell samples for a gene signature by averaging the expression of its top-ranked genes — here, the top 20 genes of the MetaTiME MeC-42 (M_Macrophage-IL1-NFKB) IL1B macrophage meta-component — to map samples onto the MeC space for downstream survival or phenotype association analyses.

## When to use

Use when you need to translate a cell-state or pathway gene signature (e.g. a MetaTiME MeC weight table) into a per-sample numeric score in bulk RNA-seq data such as TCGA, for grouping samples into high/low signature classes.

## When NOT to use

- Input is already a per-cell or per-sample feature/score table — re-scoring would double-transform the data
- The analysis requires single-cell cell-state annotation rather than a bulk per-sample signature score (use the MetaTiME Annotator directly on scRNA-seq instead)
- The gene set has no pretrained weight table or clear top-gene ranking, since the method depends on selecting the top 20 z-weighted genes

## Inputs

- Pretrained MeC gene-weight table from metatime.mecs (gene z-weights per meta-component)
- Bulk RNA-seq expression matrix (e.g. TCGA tumor samples)
- Clinical survival metadata (survival time, event status, cancer type) for TCGA tumors

## Outputs

- Per-sample signature scores written to tcga_mec42_scores.csv
- Kaplan–Meier survival curves comparing high vs. low signature groups, saved to tcga_mec42_km_survival.png
- Log-rank test statistics and per-cancer-type stratified survival associations

## How to apply

Extract the signature genes from the pretrained MeC gene-weight table via the metatime.mecs module, taking the top 20 genes ranked by z-weight for the meta-component of interest (e.g. MeC-42, M_Macrophage-IL1-NFKB). Load bulk expression and clinical metadata with metatime.loaddata, then compute each tumor's score as the averaged expression of the 20 signature genes, using metatime.mecmapper / metatime.plmapper to map bulk samples onto the MeC space. Split tumors into high- and low-signature groups by the score, fit Kaplan–Meier curves with lifelines, compare groups with a log-rank test, and stratify by cancer type. Averaging top-weight genes is a simple, robust signature-scoring choice that transfers single-cell-derived MeC programs to bulk data without requiring the full deconvolution model.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene-weight table and the mecmapper/plmapper modules used to extract top-20 signature genes and score bulk samples in the MeC space) — github.com/yi-zhang/MetaTiME
- **lifelines** (Python library used to fit Kaplan–Meier survival curves for high vs. low signature groups and compute log-rank tests)

## Examples

```
from metatime import mecs, loaddata, mecmapper; genes = mecs.top_genes('M_Macrophage-IL1-NFKB', n=20); scores = loaddata.tcga_expr[genes].mean(axis=1); scores.to_csv('tcga_mec42_scores.csv')
```

## Evaluation signals

- The scored high-MeC-42 group shows lower survival than the low group, with the effect most pronounced in LGG and KIRC, matching the paper's reported direction
- Log-rank test yields significant separation of Kaplan–Meier curves in the driving cancer types
- tcga_mec42_scores.csv contains one score per tumor sample with no missing values after expression mapping
- The top-20 gene list extracted from MeC-42 z-weights is consistent with an IL1B/NFKB macrophage program (IL1B-pathway genes at the top of the ranking)

## Limitations

- Averaged-expression scoring is a simple signature summary; in bulk tumors it reflects the signature-bearing cell population (IL1B macrophages) admixed with tumor cells, not per-cell states
- Survival associations are observational and cancer-type dependent; a pan-cancer signal may be driven by a subset of tumor types such as LGG and KIRC
- The approach requires bulk expression genes to overlap the 20-gene signature; datasets with different gene annotations or normalization may need mapping before scoring

## Evidence

- [results] Higher expression of the MeC-42 (M_Macrophage-IL1-NFKB) IL1B signature, scored in TCGA tumors using the averaged expression of its top 20 genes, is associated with lower survival rate: [evidence span withheld — anchor status: not_found]
- [methods] Score every TCGA tumor as the averaged expression of the 20 MeC-42 signature genes, mapping bulk samples onto the MeC space: [evidence span withheld — anchor status: not_found]
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [abstract] By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data."
- [methods] Split tumors into high- and low-signature groups and fit Kaplan–Meier survival curves in Python (lifelines), comparing groups with a log-rank test: [evidence span withheld — anchor status: not_found]
