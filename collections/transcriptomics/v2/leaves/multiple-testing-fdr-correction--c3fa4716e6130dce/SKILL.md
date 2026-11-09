---
name: multiple-testing-fdr-correction
description: Use when an analysis tests many pathways (or pathway-by-alteration, pathway-by-drug,
  pathway-per-cohort combinations) simultaneously and a p-value family must be converted
  to FDR before declaring significance, e.g. the 10% FDR GDSC screen or pan-cancer
  driver/survival tables.
when_to_use_negative:
- When only a single hypothesis is tested (e.g. one pathway in one cohort) — no multiplicity
  correction is needed
- When the p-value family spans different data types or effect directions that should
  not be pooled into one BH correction; correct within each cohort/analysis instead
- When the goal is effect-size ranking rather than significance calling (e.g. reporting
  hazard ratios ordered by HR)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R package that produces the per-sample pathway activity scores whose ANOVA
    p-values are then FDR-corrected across pathways, alterations, and drugs
  repo: https://github.com/saezlab/progeny
- name: survival
  role: R survival package (coxph) used to fit covariate-adjusted Cox models per pathway;
    its p-values are Benjamini-Hochberg corrected within each cohort
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:28cc0266ad477e37e67c681cb1bc7463d24413a5de65b97d5cd9d98bc0d6fadc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - progeny
  - survival
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multiple-testing-fdr-correction

## Summary

Benjamini-Hochberg FDR correction applied to the p-value families generated in the PROGENy analyses (driver-mutation ANOVAs in TCGA, covariate-adjusted Cox survival models, GDSC drug-sensitivity ANOVAs) so that pathway-association calls are controlled for the many pathway x alteration/drug/cohort tests run in parallel.

## When to use

Use when an analysis tests many pathways (or pathway-by-alteration, pathway-by-drug, pathway-per-cohort combinations) simultaneously and a p-value family must be converted to FDR before declaring significance, e.g. the 10% FDR GDSC screen or pan-cancer driver/survival tables.

## When NOT to use

- When only a single hypothesis is tested (e.g. one pathway in one cohort) — no multiplicity correction is needed
- When the p-value family spans different data types or effect directions that should not be pooled into one BH correction; correct within each cohort/analysis instead
- When the goal is effect-size ranking rather than significance calling (e.g. reporting hazard ratios ordered by HR)

## Inputs

- Raw (uncorrected) p-value vector from ANOVA of alteration status vs PROGENy pathway score per tumor type
- Cox proportional hazards model p-values per pathway per cohort (survival package coxph)
- ANOVA p-values per drug x pathway combination from GDSC IC50 models conditioned on MSI status and tissue of origin

## Outputs

- pancancer_driver_pathway_fdr.csv with alteration, pathway, and FDR columns
- survival_associations.csv with tidy (pathway, cohort, HR, FDR) rows
- drug_pathway_associations.csv with effect sizes and FDR values for associations significant at 10% FDR (178 rows)

## How to apply

Assemble all raw p-values from a single hypothesis family into one table (e.g. ANOVA p-values of alteration status against each PROGENy pathway score within each tumor type, or Cox-model p-values for each pathway within each of the 34 cancer types), then apply Benjamini-Hochberg correction across that family, keeping cohorts/families separate rather than pooling everything. Retain the corrected FDR alongside the effect size (ANOVA coefficient or Cox hazard ratio) in a tidy output table with explicit alteration/pathway/cohort/FDR columns. The significance threshold is set by the analysis goal: driver associations were verified at FDR < 10^-9 (EGFR amplification), survival associations at FDR < 10^-3, and GDSC drug-pathway hits were retained at 10% FDR. Rationale: testing 14 PROGENy pathways across many alterations, drugs, or cancer types inflates the false-positive rate, so uncorrected p-values cannot separate the strong EGFR/MAPK activations from noise.

## Related tools

- **progeny** (R package that produces the per-sample pathway activity scores whose ANOVA p-values are then FDR-corrected across pathways, alterations, and drugs) — https://github.com/saezlab/progeny
- **survival** (R survival package (coxph) used to fit covariate-adjusted Cox models per pathway; its p-values are Benjamini-Hochberg corrected within each cohort)

## Examples

```
fdr <- p.adjust(pvals, method = "BH")  # Benjamini-Hochberg across pathways within each cohort
```

## Evaluation signals

- Driver table: EGFR amplification–EGFR and –MAPK FDR < 10^-9; BRAF mutation < 10^-10; TP53 mutation < 10^-64; VHL mutation–hypoxia < 10^-200
- Survival table: significant increased hazard for EGFR, MAPK, PI3K, Hypoxia pan-cancer; decreased hazard for Trail; p53 significant in adrenocortical carcinoma, all at FDR < 10^-3
- GDSC table contains exactly 178 significant drug-pathway rows, all FDR values in [0, 0.1], with 'Nutlin-3a' as the strongest hit
- Schema check: every output row carries pathway, cohort/alteration/drug, effect size, and corrected FDR columns with FDR in [0, 1]
- FDR values are monotone with raw p-values within each corrected family (BH property)

## Limitations

- FDR-corrected significance is only as good as the underlying model: the GDSC ANOVAs must condition on MSI status and tissue of origin, and survival Cox models must adjust for study of origin and age, or residual confounding inflates apparent associations
- Cutting at a loose threshold (10% FDR in GDSC) yields a large hit list (178 associations) that still requires biological prioritization, e.g. MAPK/EGFR dominance and the Nutlin-3a–p53 interaction
- Benjamini-Hochberg controls the expected false discovery rate, not the per-test family-wise error; single extreme hits (e.g. VHL–hypoxia FDR < 10^-200) should not be extrapolated to weaker associations near the threshold

## Evidence

- [methods] Apply Benjamini-Hochberg FDR correction across pathways within each cohort and record hazard ratios and FDR values in a tidy table: [evidence span withheld — anchor status: not_found]
- [results] Adjust the ANOVA p-values for multiple testing and retain associations significant at 10% FDR: [evidence span withheld — anchor status: not_found]
- [full_text] Collect the ANOVA p-values, correct them for multiple testing to FDR, and assemble a pan-cancer association table: [evidence span withheld — anchor status: not_found]
- [methods] For each drug and pathway, fit an ANOVA model relating IC50 to the pathway score while conditioning on MSI status and tissue of origin: [evidence span withheld — anchor status: not_found]
