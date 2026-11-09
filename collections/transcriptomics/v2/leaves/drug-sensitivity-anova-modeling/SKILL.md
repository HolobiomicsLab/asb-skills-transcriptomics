---
name: drug-sensitivity-anova-modeling
description: Use when you have matched drug-sensitivity measurements (e.g., IC50 values
  from GDSC) and gene-expression-derived pathway scores for the same cell lines and
  want to test which pathways explain drug response.
when_to_use_negative:
- Your response data are not quantitative drug-sensitivity values (e.g., binary resistance
  calls or mutation-drug screens) — the ANOVA on IC50 assumes continuous response
- You lack cell-line covariates (MSI status and tissue of origin); without conditioning
  on these, associations will be confounded by lineage
- The cell lines have no matched gene-expression profiles from which to compute PROGENy
  scores
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R/Bioconductor package used to score the pathway-responsive gene signatures
    in each cell line's expression profile, yielding the pathway activity scores used
    as ANOVA predictors
  repo: https://github.com/saezlab/progeny
- name: saezlab/footprints
  role: Analysis code repository containing the original drug-sensitivity association
    analyses (analyses/drug_assocs) that this skill reproduces
  repo: https://github.com/saezlab/footprints
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:3230ad355c0533b3d562b9ae3ae1cb6cc027bcbfd00831a131148d84a98ce53f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - progeny
  - saezlab/footprints
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# drug-sensitivity-anova-modeling

## Summary

Fit per-drug ANOVA models relating GDSC IC50 drug-sensitivity values to PROGENy pathway activity scores, conditioning on MSI status and tissue of origin, to identify drug-pathway associations at 10% FDR. This reproduces the paper's Fig. 4 analysis showing PROGENy scores explain drug sensitivity in 805 cancer cell lines.

## When to use

Use when you have matched drug-sensitivity measurements (e.g., IC50 values from GDSC) and gene-expression-derived pathway scores for the same cell lines and want to test which pathways explain drug response.

## When NOT to use

- Your response data are not quantitative drug-sensitivity values (e.g., binary resistance calls or mutation-drug screens) — the ANOVA on IC50 assumes continuous response
- You lack cell-line covariates (MSI status and tissue of origin); without conditioning on these, associations will be confounded by lineage
- The cell lines have no matched gene-expression profiles from which to compute PROGENy scores

## Inputs

- GDSC v17a microarray gene expression matrix for 805 cancer cell lines (ArrayExpress E-MTAB-3610)
- GDSC drug-sensitivity IC50 table for 265 drugs across 805 cell lines
- MSI status annotations per cell line
- Tissue-of-origin annotations per cell line
- PROGENy pathway-responsive gene signature matrix (11 pathways, e.g. EGFR, MAPK, p53/DDR, PI3K)

## Outputs

- drug_pathway_associations.csv containing significant drug-pathway association rows with effect sizes and FDR values (178 associations at 10% FDR in the paper)
- Per-pathway activity scores for each cell line

## How to apply

Download the GDSC v17a microarray expression dataset (ArrayExpress E-MTAB-3610) and the GDSC IC50 table covering 265 drugs in 805 cell lines. Score the PROGENy pathway-responsive gene signatures in each cell line's expression profile using the progeny package to obtain pathway activity scores. For each drug-pathway pair, fit an ANOVA model with IC50 as the response, the pathway score as predictor, and MSI status and tissue of origin as covariates — conditioning on these confounders is the key design decision so associations reflect pathway activity rather than lineage or microsatellite status. Adjust the ANOVA p-values for multiple testing and retain associations at 10% FDR. Write effect sizes and FDR values to a drug-pathway association table.

## Related tools

- **progeny** (R/Bioconductor package used to score the pathway-responsive gene signatures in each cell line's expression profile, yielding the pathway activity scores used as ANOVA predictors) — https://github.com/saezlab/progeny
- **saezlab/footprints** (Analysis code repository containing the original drug-sensitivity association analyses (analyses/drug_assocs) that this skill reproduces) — https://github.com/saezlab/footprints

## Examples

```
scores <- progeny(expr, scale = TRUE); fit <- aov(IC50 ~ MAPK_score + MSI_status + tissue, data = gdsc); summary(fit)
```

## Evaluation signals

- The output association table contains exactly 178 significant drug-pathway rows when run on the GDSC v17a / 265-drug / 805-cell-line setup
- All reported FDR values in the output table fall within [0, 0.1]
- Nutlin-3a is the strongest drug-pathway hit, associated with the p53 pathway score
- Associations are dominated by MAPK/EGFR activity associations with MAPK-targeting drugs
- Every tested drug-pathway model includes MSI status and tissue of origin as covariates (no marginal, unadjusted models in the table)

## Limitations

- Tissue-specific signatures may more closely reflect pathway activation than the consensus PROGENy signatures in a given context
- Pathway scores from gene expression do not account for post-translational modifications, so the ANOVA tests transcriptional footprints of signaling rather than direct signaling activity
- Requiring FDR significance at 10% may discard weaker but biologically relevant associations given the multiple-testing burden of 265 drugs x 11 pathways

## Evidence

- [readme] GDSC biomarker workflow: "#### Explaining drug sensitivity (Fig. 4)

Analyses available in [analyses/drug_assocs](analyses/drug_assocs)."
- [abstract] drug indication finding: [evidence span withheld — anchor status: not_found]
- [intro] GDSC/CCLE context: "the GDSC4,5 and CCLE6 have focussed on preclinical biomarkers of drug sensitivity in cancer cell lines"
- [full_text] conditioning covariates: [evidence span withheld — anchor status: not_found]
- [discussion] tissue-specific caveat: "signatures derived from, for instance, a specific tissue may still more closely reflect activation"
