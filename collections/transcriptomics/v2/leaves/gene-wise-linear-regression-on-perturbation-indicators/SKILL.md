---
name: gene-wise-linear-regression-on-perturbation-indicators
description: 'Use when building consensus pathway-response signatures from a compendium
  of perturbation experiments: per-gene z-scores and a binary perturbation indicator
  matrix are available and pathway-responsiveness coefficients are needed.'
when_to_use_negative:
- Input is raw microarray intensities that have not yet been normalized, QC'd per
  series, and z-scored per perturbation experiment
- The goal is to score pathway activity in new expression data rather than to build
  signatures — use the resulting PROGENy matrix with a scoring method (e.g. decoupler)
  instead
- Only a single tissue- or context-specific dataset is available and a context-specific
  signature would more closely reflect pathway activation than the consensus model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: progeny
  role: R package distributing the pathway signatures produced by this regression
    workflow
  repo: https://github.com/saezlab/progeny
- name: knitr
  role: Used to auto-generate supplementary report figures for the footprints analysis
  repo: https://github.com/saezlab/footprints
- name: rmarkdown
  role: Vignette engine used with knitr for the progeny package documentation
  repo: https://github.com/saezlab/progeny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:849cc2bbed232824695b6d8e36b5bcf09d8487978efd9363ae008517a57e6357
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - knitr
  - progeny
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# Gene-wise linear regression on perturbation indicators

## Summary

Fits a gene-wise linear model of z-scored perturbation-expression data on a binary pathway-perturbation design matrix to estimate pathway-responsiveness coefficients, from which the top 100 responsive genes per pathway form the PROGENy consensus signatures.

## When to use

Use when building consensus pathway-response signatures from a compendium of perturbation experiments: per-gene z-scores and a binary perturbation indicator matrix are available and pathway-responsiveness coefficients are needed.

## When NOT to use

- Input is raw microarray intensities that have not yet been normalized, QC'd per series, and z-scored per perturbation experiment
- The goal is to score pathway activity in new expression data rather than to build signatures — use the resulting PROGENy matrix with a scoring method (e.g. decoupler) instead
- Only a single tissue- or context-specific dataset is available and a context-specific signature would more closely reflect pathway activation than the consensus model

## Inputs

- Per-experiment z-scores of gene expression from perturbation experiments (ArrayExpress compendium, 208 submissions / 580 experiments across 11 pathways)
- Binary matrix indicating pathway perturbations, including perturbations of multiple pathways (YAML index of experiments in the footprints repository)

## Outputs

- Gene-wise pathway-responsiveness regression coefficients
- Top 100 non-zero pathway-responsive genes per pathway
- 11 x 100 gene-by-pathway coefficient matrix (e.g. the 'speed_matrix' model used in the publication)

## How to apply

First normalize and QC each ArrayExpress series as a whole, assemble the relevant expression data, and compute per-experiment z-scores of gene expression so experiment-level effects are removed. Then construct a binary matrix indicating pathway perturbations (including perturbations of multiple pathways) as the independent variable and fit a linear model of the z-scores on these indicators gene by gene. Rank genes per pathway by the regression coefficient (selecting the 100 most significant genes) and use their z-scores as coefficients in the pathway model, assembling an 11 x 100 gene-by-pathway coefficient matrix covering EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail and VEGF. The rationale is that despite heterogeneity of individual perturbation signatures, a common core of responsive genes describes pathway activation across conditions, which a pooled regression recovers better than single-study signatures.

## Related tools

- **progeny** (R package distributing the pathway signatures produced by this regression workflow) — https://github.com/saezlab/progeny
- **knitr** (Used to auto-generate supplementary report figures for the footprints analysis) — https://github.com/saezlab/footprints
- **rmarkdown** (Vignette engine used with knitr for the progeny package documentation) — https://github.com/saezlab/progeny

## Examples

```
Rscript data/zscores.r && Rscript model/speed_matrix.r  # fit gene-wise linear model on z-scores with binary perturbation indicators and select top 100 genes per pathway
```

## Evaluation signals

- The fitted design matrix contains binary perturbation indicators for all 11 pathways, including multi-pathway perturbation columns
- Exactly the 100 most significant (top-coefficient, non-zero) genes are retained per pathway
- Resulting coefficient matrix has dimensions 11 pathways x 100 genes covering EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF
- Input z-scores were computed after per-series normalization and QC, with excluded experiments removed per the .excluded annotations
- The fitted model reproduces pathway activity associations reported in the paper: driver mutations in TCGA, GDSC drug response, and patient survival

## Limitations

- Individual expression signatures are heterogeneous; the regression captures only the common core of responsive genes, and additional transcriptional modules may not always be activated
- Consensus signatures may be less accurate than tissue-specific signatures for a specific context; PROGENy is best suited to large and heterogeneous data sets
- Experiments are excluded due to QC failure or uncertain perturbation-phenotype correspondence, which can reduce available data per pathway

## Evidence

- [readme] For this, we fit a linear model on the z-scores with a binary matrix indicating pathway perturbations (incl. perturbations of multiple pathways) as the independent variable.: [evidence span withheld — anchor status: external_doc]
- [readme] We select the 100 most significant genes and use their z-scores as coefficients in the model.: [evidence span withheld — anchor status: external_doc]
- [readme] We normalized and QC'd each series as whole (`normalize_data.r`), then assembled the relevant expression data (`expr.r`) and computed z-scores for each perturbation experiment (`zscores.r`).: [evidence span withheld — anchor status: external_doc]
- [readme] We overcome both limitations by leveraging a large compendium of publicly available perturbation experiments to define consensus signatures for pathway activity.: [evidence span withheld — anchor status: external_doc]
- [intro] we overcome the limitations of both approaches by leveraging a large compendium of publicly available perturbation experiments that yield a common core of Pathway RespOnsive GENes: "leveraging a large compendium of publicly available perturbation experiments that yield a common core of Pathway RespOnsive GENes"
