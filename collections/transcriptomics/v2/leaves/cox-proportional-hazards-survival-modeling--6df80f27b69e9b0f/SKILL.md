---
name: cox-proportional-hazards-survival-modeling
description: Use when pathway activity scores (e.g., PROGENy scores for EGFR, MAPK,
  PI3K, Hypoxia, Trail) must be tested against overall survival time and event status
  in a tumor cohort such as TCGA, with covariate adjustment.
when_to_use_negative:
- Input is already a per-sample survival result table rather than pathway scores plus
  survival metadata — no Cox regression is needed
- The cohort lacks event/time survival endpoints or covariates (study of origin, age),
  so a covariate-adjusted coxph model cannot be specified
- The expression data are outside the human signaling-pathway scope of PROGENy (e.g.,
  a single homogeneous condition rather than a large heterogeneous tumor cohort);
  the article notes tissue-specific signatures may better reflect activation status
  in that context
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PROGENy
  role: Source of the pathway activity scores (signaling footprints for EGFR, Hypoxia,
    JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF) that are tested against
    patient survival
  repo: https://github.com/saezlab/progeny
- name: saezlab/footprints
  role: Analysis code repository containing the tcga_survival analyses (analyses/tcga_survival,
    Fig. 5) implementing this survival workflow
  repo: https://github.com/saezlab/footprints
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:15ddedb9c40057bad0f045c4ff4889be73eb9b3b3790b4681b55b813cb39e238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - PROGENy
  - saezlab/footprints
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cox-proportional-hazards-survival-modeling

## Summary

Fits covariate-adjusted Cox proportional hazards models (R survival package, coxph) on PROGENy pathway activity scores for TCGA tumors to test whether inferred pathway activation associates with overall patient survival and to separate oncogenic from tumor-suppressive pathways.

## When to use

Use when pathway activity scores (e.g., PROGENy scores for EGFR, MAPK, PI3K, Hypoxia, Trail) must be tested against overall survival time and event status in a tumor cohort such as TCGA, with covariate adjustment.

## When NOT to use

- Input is already a per-sample survival result table rather than pathway scores plus survival metadata — no Cox regression is needed
- The cohort lacks event/time survival endpoints or covariates (study of origin, age), so a covariate-adjusted coxph model cannot be specified
- The expression data are outside the human signaling-pathway scope of PROGENy (e.g., a single homogeneous condition rather than a large heterogeneous tumor cohort); the article notes tissue-specific signatures may better reflect activation status in that context

## Inputs

- Matrix of PROGENy pathway activity scores for 7254 TCGA tumors
- Clinical survival metadata: overall survival time, event status, cancer type, study of origin, age

## Outputs

- Tidy table survival_associations.csv with columns (pathway, cohort, HR, FDR) from Benjamini-Hochberg-corrected Cox regressions per pathway per cohort

## How to apply

Load the matrix of PROGENy pathway activity scores for the 7254 TCGA tumors together with clinical survival metadata (overall survival time, event status, cancer type, study of origin, age). For each PROGENy pathway, fit a Cox proportional hazards model with the R survival package (coxph) on the pan-cancer cohort, regressing out study of origin and age as covariates, then repeat the covariate-adjusted regression within each of the 34 individual cancer types. Apply Benjamini-Hochberg FDR correction across pathways within each cohort and record hazard ratios and FDR values in a tidy table (pathway, cohort, HR, FDR) written to survival_associations.csv. The rationale is that PROGENy-derived signaling footprints better capture pathway activation than expression-level gene sets, so their scores are expected to carry directional prognostic signal: activation of EGFR, MAPK, PI3K, and Hypoxia increases hazard, whereas Trail/apoptosis activation decreases hazard.

## Related tools

- **PROGENy** (Source of the pathway activity scores (signaling footprints for EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF) that are tested against patient survival) — https://github.com/saezlab/progeny
- **saezlab/footprints** (Analysis code repository containing the tcga_survival analyses (analyses/tcga_survival, Fig. 5) implementing this survival workflow) — https://github.com/saezlab/footprints

## Examples

```
Rscript -e 'library(survival); m <- coxph(Surv(OS_time, OS_event) ~ pathway_score + study_origin + age, data = tcga_cohort); summary(m)'
```

## Evaluation signals

- Pan-cancer Cox regressions show significantly increased hazard (decreased survival) for EGFR, MAPK, PI3K, and Hypoxia pathway activation at FDR < 10^-3
- Pan-cancer regressions show significantly decreased hazard (increased survival) for Trail/apoptosis pathway activation — PROGENy is the only method to find this
- A significant p53 association appears in adrenocortical carcinoma at FDR < 10^-3
- Output table is tidy with the schema (pathway, cohort, HR, FDR) and FDR values are Benjamini-Hochberg-corrected within each cohort (pan-cancer and each of the 34 cancer types)
- Hazard ratio signs are directionally consistent with the oncogenic vs. tumor-suppressor separation reported in the article

## Limitations

- Tissue-specific signatures may more closely reflect pathway activation status than the consensus PROGENy signatures in some contexts, potentially affecting per-cancer-type associations
- Pathway scores from gene-expression-based methods reflect signaling activity imperfectly because they do not account for post-translational modifications; robustness issues were noted from heterogeneity in responses of individual signatures
- PROGENy provides a good estimate of pathway activity mainly in large and heterogeneous data sets; smaller cohorts may not show the same survival separations

## Evidence

- [abstract] PROGENy can distinguish between oncogenic and tumor suppressor pathways for patient survival: "and (iii) distinguish between oncogenic and tumor suppressor pathways for patient survival"
- [discussion] Tissue-specific signatures caveat for pathway activation in specific contexts: "While PROGENy provides a good estimate of pathway activity in large and heterogeneous data sets, signatures derived from, for instance, a specific tissue may still more closely reflect activation"
- [readme] Repository analysis code location for the survival analysis: "Effect on patient survival (Fig. 5)

Analyses available in [analyses/tcga_survival](analyses/tcga_survival)."
- [other] Survival workflow definition from the source task card: [evidence span withheld — anchor status: not_found]
