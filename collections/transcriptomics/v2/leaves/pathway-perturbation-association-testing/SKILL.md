---
name: pathway-perturbation-association-testing
description: Use when you have a matrix of perturbation gene-expression experiments
  and a PROGENy model, and need to test that inferred pathway activations are specific
  to the experimentally perturbed pathway even when that experiment is held out of
  model building.
when_to_use_negative:
- Input data are not gene-expression perturbation experiments (e.g. methylation or
  proteomics) — PROGENy footprints were derived from ArrayExpress perturbation transcriptomics.
- Pathway scores are already computed and the task is only a generic correlation analysis
  without any experiment being held out — leave-one-out rebuilding would be pointless.
- Species outside human or mouse, since PROGENy signatures cover only human and mouse.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R/Bioconductor package holding the pathway signatures and scoring pathway
    activity from the leave-one-out rebuilt models
  repo: https://github.com/saezlab/progeny
- name: knitr
  role: used to auto-generate the supplementary report scripts wrapping the analyses
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:2da0a83788ae8a3c8935888b29f2c681d376e9e825cde68baed66a4f2bf7b11d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - knitr
  - progeny
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# pathway-perturbation-association-testing

## Summary

Leave-one-out validation of PROGENy pathway activity inference: for each perturbation experiment, the multivariate regression model is rebuilt with that experiment excluded, pathway scores are computed with the progeny R package, and statistical association p-values between pathway scores and perturbation conditions are assembled to test whether the model recovers the perturbed pathway it never saw.

## When to use

Use when you have a matrix of perturbation gene-expression experiments and a PROGENy model, and need to test that inferred pathway activations are specific to the experimentally perturbed pathway even when that experiment is held out of model building.

## When NOT to use

- Input data are not gene-expression perturbation experiments (e.g. methylation or proteomics) — PROGENy footprints were derived from ArrayExpress perturbation transcriptomics.
- Pathway scores are already computed and the task is only a generic correlation analysis without any experiment being held out — leave-one-out rebuilding would be pointless.
- Species outside human or mouse, since PROGENy signatures cover only human and mouse.

## Inputs

- PROGENy model coefficients matrix (linear model fit on perturbation z-scores)
- Perturbation gene-expression experiments with per-experiment perturbed-pathway labels
- Binary pathway-perturbation indicator matrix (which experiments perturbed which of the 11 pathways)

## Outputs

- pathway_perturbation_associations.csv (per-pathway association p-value matrix)
- Leave-one-out pathway activity scores per experiment

## How to apply

Load the PROGENy model coefficients matrix (the linear model fit on z-scores of perturbation experiments with a binary pathway-perturbation indicator as independent variable, using the 100 most significant genes) together with the held-out perturbation gene-expression experiments. For each perturbation experiment, rebuild the PROGENy model with that experiment excluded (leave-one-out) and score pathway activity with the progeny R package. Compute association p-values between each pathway score and each perturbation condition across the experiment compendium, covering the 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF). Assemble the per-pathway p-value matrix into a single association table and write it to pathway_perturbation_associations.csv. Rationale: perturbation signatures are heterogeneous individually, so the common core of Pathway RespOnsive GENes must generalize beyond the experiments used to fit the model; leave-one-out is the direct test of this.

## Related tools

- **progeny** (R/Bioconductor package holding the pathway signatures and scoring pathway activity from the leave-one-out rebuilt models) — https://github.com/saezlab/progeny
- **knitr** (used to auto-generate the supplementary report scripts wrapping the analyses)

## Examples

```
scores <- progeny::progeny(expr_zscores, scale = TRUE, top = 100)
```

## Evaluation signals

- Each pathway's self-association p-value (pathway score vs. its own perturbation condition) is below 1e-10.
- The Trail pathway's self-association p-value is below 1e-5 (the known relaxed threshold for this pathway).
- All expected pathway labels (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) are present in the association table.
- pathway_perturbation_associations.csv exists and contains a p-value for every pathway x perturbation-condition pair.
- Off-diagonal (cross-pathway) associations are weaker than self-associations, indicating pathway-specific recovery.

## Limitations

- Individual perturbation signatures are heterogeneous, so a perturbation experiment may not be consistently recovered.
- Consensus PROGENy signatures may underperform tissue-specific signatures in a specific context (e.g. the Gatza et al. breast cancer signatures on TCGA).
- Files/experiments with QC failure or ambiguous perturbation-to-phenotype correspondence were excluded from model building, so compendium coverage is partial (208 submissions, 580 experiments).

## Evidence

- [results] LOO validation finding: [evidence span withheld — anchor status: not_found]
- [readme] model construction: [evidence span withheld — anchor status: external_doc]
- [readme] compendium scope: [evidence span withheld — anchor status: external_doc]
- [abstract] rationale: "Here we present PROGENy, a method that overcomes both limitations by leveraging a large compendium of publicly available perturbation experiments"
