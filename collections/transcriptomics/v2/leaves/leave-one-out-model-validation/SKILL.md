---
name: leave-one-out-model-validation
description: 'Use when validating a PROGENy model built from the 11-pathway perturbation
  compendium (208 ArrayExpress submissions, 580 experiments): you need evidence that
  pathway scores recover the perturbed pathway for experiments not seen during model
  construction.'
when_to_use_negative:
- When the goal is benchmarking PROGENy against other pathway methods (GSVA, SPIA,
  Pathifier, PARADIGM) on fixed data — use the full perturbation-recall comparison
  instead
- When the pathway under study is not among the 11 perturbation pathways (e.g. pathway
  lacks sufficient ArrayExpress perturbation experiments to fit a leave-one-out model)
- When the expression data are not z-scored relative to proper controls — the regression
  is defined on per-experiment z-scores
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R package used to score pathway activity of held-out perturbation experiments
    with the rebuilt leave-one-out model
  repo: https://github.com/saezlab/progeny
- name: knitr
  role: Used for auto-generating the report/supplementary-figure scripts accompanying
    the analysis
- name: rmarkdown
  role: Vignette/report engine for the PROGENy package methods documentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:dc9623872d2cca8857357f9cac2428d689936738e013f664d6e05dcc7e419517
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - knitr
  - progeny
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# leave-one-out-model-validation

## Summary

Evaluates whether PROGENy pathway-activity scores generalize to perturbation experiments excluded from model building. Each perturbation experiment is held out, the PROGENy multivariate regression model is rebuilt without it, and the association between inferred pathway activation and the perturbed pathway is tested.

## When to use

Use when validating a PROGENy model built from the 11-pathway perturbation compendium (208 ArrayExpress submissions, 580 experiments): you need evidence that pathway scores recover the perturbed pathway for experiments not seen during model construction.

## When NOT to use

- When the goal is benchmarking PROGENy against other pathway methods (GSVA, SPIA, Pathifier, PARADIGM) on fixed data — use the full perturbation-recall comparison instead
- When the pathway under study is not among the 11 perturbation pathways (e.g. pathway lacks sufficient ArrayExpress perturbation experiments to fit a leave-one-out model)
- When the expression data are not z-scored relative to proper controls — the regression is defined on per-experiment z-scores

## Inputs

- PROGENy model coefficients matrix (linear model fitted on z-scores with a binary pathway-perturbation indicator matrix as independent variable; 100 most significant genes selected)
- Perturbation experiment gene-expression z-scores (computed per perturbation experiment from normalized ArrayExpress series)
- Pathway-to-experiment annotation for the 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF)

## Outputs

- pathway_perturbation_associations.csv (per-pathway association p-value matrix between inferred pathway scores and perturbation conditions)
- Per-pathway self-association p-values under leave-one-out validation

## How to apply

Load the PROGENy model coefficients matrix (the 'speed_matrix'-style linear model fitted on z-scores with a binary perturbation matrix as independent variable) together with the held-out perturbation gene-expression z-scores. For each perturbation experiment, rebuild the PROGENy multivariate regression model with that experiment excluded and score pathway activity using the progeny R package. Compute statistical association p-values between each pathway score and each perturbation condition across the experiment compendium, then assemble the per-pathway p-value matrix into a single table. Rationale: leave-one-out testing mirrors the perturbation-recall analysis (Fig. 2) and guards against trivial recovery, since the model was fit on a large compendium of perturbation experiments.

## Related tools

- **progeny** (R package used to score pathway activity of held-out perturbation experiments with the rebuilt leave-one-out model) — https://github.com/saezlab/progeny
- **knitr** (Used for auto-generating the report/supplementary-figure scripts accompanying the analysis)
- **rmarkdown** (Vignette/report engine for the PROGENy package methods documentation)

## Examples

```
library(progeny); pathway_scores <- progeny(zscore_matrix, scale = TRUE); write.csv(pathway_perturbation_associations, "pathway_perturbation_associations.csv")
```

## Evaluation signals

- Each pathway's self-association p-value in the leave-one-out table is below 1e-10 (below 1e-5 for the Trail pathway)
- All expected pathway labels (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF) are present in pathway_perturbation_associations.csv
- The p-value matrix is complete: one row/column per pathway with a p-value for every pathway-score × perturbation-condition pair
- Each held-out experiment's model is genuinely refit without that experiment (coefficients differ from the full-data model), confirming a true leave-one-out setup

## Limitations

- Individual expression signatures are heterogeneous; only the common core of responsive genes describes pathway activation across a wide range of conditions
- Tissue-specific signatures may more closely reflect pathway activation than the consensus PROGENy signatures in a specific context
- Excluded experiments (QC failure or ambiguous perturbation-to-phenotype correspondence) are marked in the index files and must not be treated as validation targets

## Evidence

- [readme] In leave-one-out validation, PROGENy's inferred pathway activation is strongly associated with the experimentally perturbed pathway (p < 10^-10, except p < 10^-5 for Trail): [evidence span withheld — anchor status: not_found]
- [readme] Model construction uses a linear model on z-scores with a binary perturbation matrix, from which leave-one-out variants are derived: [evidence span withheld — anchor status: external_doc]
- [readme] The compendium scope defining what is held out is the 208 ArrayExpress submissions covering 580 experiments: [evidence span withheld — anchor status: not_found]
- [abstract] The skill's validity rests on PROGENy's generalization across conditions, stated in the abstract: "these results show that PROGENy accurately infers pathway activity from gene expression in a wide range of conditions"
