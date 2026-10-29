---
name: gene-expression-matrix-processing
description: Use when perturbation gene-expression series (e.g., ArrayExpress submissions)
  must be normalized, QC'd, assembled into an expression matrix, and converted to
  z-scores for pathway-activity modeling.
when_to_use_negative:
- Input is not gene expression from pathway-perturbation experiments (e.g., proteomics,
  methylation, or non-perturbation conditions), since PROGENy's responsive-gene core
  is derived from perturbation compendia
- The experiment perturbation phenotype is uncertain or failed QC (files marked .excluded),
  as including such series corrupts the learned signatures
- The goal is scoring a ready-made expression matrix against existing signatures rather
  than building or rebuilding the z-score/model pipeline
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R/Bioconductor package that stores the pathway signatures and scores pathway
    activity from the processed z-score/expression matrices during leave-one-out validation
  repo: https://github.com/saezlab/progeny
- name: knitr
  role: R engine used in the methods/reporting pipeline that wraps the gene-expression
    processing scripts (rmarkdown vignette engine)
- name: rmarkdown
  role: Vignette/document engine used with knitr to render the analysis report of
    the expression-processing workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:b0489b9c5bcd9dbec941d1f5dc9ed74ba03ffc95ed616e4a07a1dcbce28d092f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - knitr
  - progeny
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gene-expression-matrix-processing

## Summary

Normalize, QC, assemble, and convert publicly available perturbation gene-expression experiments into per-perturbation z-score matrices that feed PROGENy pathway-activity modeling and leave-one-out validation.

## When to use

Use when perturbation gene-expression series (e.g., ArrayExpress submissions) must be normalized, QC'd, assembled into an expression matrix, and converted to z-scores for pathway-activity modeling.

## When NOT to use

- Input is not gene expression from pathway-perturbation experiments (e.g., proteomics, methylation, or non-perturbation conditions), since PROGENy's responsive-gene core is derived from perturbation compendia
- The experiment perturbation phenotype is uncertain or failed QC (files marked .excluded), as including such series corrupts the learned signatures
- The goal is scoring a ready-made expression matrix against existing signatures rather than building or rebuilding the z-score/model pipeline

## Inputs

- ArrayExpress perturbation gene-expression series (208 submissions, 580 experiments across 11 pathways: EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF)
- YAML experiment index files with query.txt search terms and .excluded exclusion markers
- Binary pathway-perturbation indicator matrix

## Outputs

- Per-perturbation z-score matrix (zscores.r output)
- Assembled normalized expression matrix (expr.r output)
- PROGENy model coefficient matrix (speed_matrix) with 100 most significant genes per pathway
- Per-pathway association p-value matrix (pathway_perturbation_associations.csv)

## How to apply

Download each perturbation expression series from ArrayExpress and normalize and QC each series as a whole using normalize_data.r, excluding experiments flagged with the '.excluded' suffix or commented entries due to QC failure or ambiguous perturbation phenotypes. Assemble the relevant expression data with expr.r, then compute z-scores for each perturbation experiment with zscores.r, contrasting perturbed samples against basal/control samples. The resulting z-score matrix, together with a binary matrix indicating pathway perturbations (including multi-pathway perturbations), serves as input for fitting the PROGENy linear model in which the 100 most significant genes' z-scores become model coefficients. For leave-one-out validation, rebuild the model with each held-out perturbation experiment excluded and score pathway activity with the progeny R package. This processing is what lets PROGENy distinguish a common core of pathway-responsive genes from heterogeneous individual signatures.

## Related tools

- **progeny** (R/Bioconductor package that stores the pathway signatures and scores pathway activity from the processed z-score/expression matrices during leave-one-out validation) — https://github.com/saezlab/progeny
- **knitr** (R engine used in the methods/reporting pipeline that wraps the gene-expression processing scripts (rmarkdown vignette engine))
- **rmarkdown** (Vignette/document engine used with knitr to render the analysis report of the expression-processing workflow)

## Examples

```
Rscript zscores.r
```

## Evaluation signals

- Each pathway's self-association p-value in leave-one-out validation is below 1e-10 (below 1e-5 for the Trail pathway)
- All expected pathway labels (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF) are present in pathway_perturbation_associations.csv
- Each series was normalized and QC'd as a whole before z-score computation; excluded experiments (.excluded suffix or commented entries) are absent from the assembled matrix
- The model selects the 100 most significant genes per pathway from the z-scores as coefficients

## Limitations

- Experiments with QC failure or uncertain perturbation-phenotype correspondence must be excluded, reducing usable data
- Individual expression signatures are heterogeneous; only a common core of responsive genes generalizes, and tissue-specific signatures may still more closely reflect activation in a specific context than the consensus PROGENy signatures
- PROGENy provides a good estimate of pathway activity in large, heterogeneous datasets, but differences between the common response and additional transcriptional modules are not fully elucidated

## Evidence

- [readme] We normalized and QC'd each series as whole (`normalize_data.r`), then assembled the relevant expression data (`expr.r`) and computed z-scores for each perturbation experiment (`zscores.r`).: [evidence span withheld — anchor status: external_doc]
- [readme] we fit a linear model on the z-scores with a binary matrix indicating pathway perturbations (incl. perturbations of multiple pathways) as the independent variable.: [evidence span withheld — anchor status: external_doc]
- [readme] We select the 100 most significant genes and use their z-scores as coefficients in the model.: [evidence span withheld — anchor status: external_doc]
- [abstract] leveraging a large compendium of publicly available perturbation experiments to yield a common core of Pathway RespOnsive GENes: "leveraging a large compendium of publicly available perturbation experiments to yield a common core of Pathway RespOnsive GENes"
