---
name: pathway-score-quartile-stratification
description: Use when you have cell-line drug sensitivity (IC50) data plus mutation
  calls and gene expression, and want to test whether PROGENy pathway activity further
  stratifies drug response within mutant vs wild-type sub-populations.
when_to_use_negative:
- Input is already a per-sample pathway activity table with no gene-expression data
  to score
- The question is mutation-status association with drug response alone, without pathway-activity
  stratification
- Samples lack matched drug sensitivity measurements, so no IC50 fold-change can be
  computed
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3336
tools:
- name: progeny
  role: Computes PROGENy pathway activity scores (e.g. MAPK) from the cell-line gene
    expression compendium used for quartile stratification
  repo: https://github.com/saezlab/progeny
- name: knitr
  role: Used to generate the reproducible reports/supplementary figures accompanying
    the stratification analysis
- name: rmarkdown
  role: Vignette/report engine for the analysis workflows producing the stratification
    tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:cb1a3ca6f18fe0607e4b9d3c386a10c881b495b288100e2aefde288923f9f64b
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
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pathway-score-quartile-stratification

## Summary

Stratifies cell lines into quartiles of a PROGENy pathway score (e.g. MAPK) within driver-mutation groups (BRAF/KRAS/NRAS mutant vs wild-type, tracking TP53) and computes fold-changes of median drug IC50 between the pathway-active top quartile and remaining lines, to test whether pathway activity adds stratification beyond mutation status alone.

## When to use

Use when you have cell-line drug sensitivity (IC50) data plus mutation calls and gene expression, and want to test whether PROGENy pathway activity further stratifies drug response within mutant vs wild-type sub-populations.

## When NOT to use

- Input is already a per-sample pathway activity table with no gene-expression data to score
- The question is mutation-status association with drug response alone, without pathway-activity stratification
- Samples lack matched drug sensitivity measurements, so no IC50 fold-change can be computed

## Inputs

- GDSC drug sensitivity IC50 table (cancerrxgene.org/gdsc1000)
- Cell-line mutation calls for BRAF, KRAS, NRAS, TP53
- Cell-line gene expression compendium (matrix for PROGENy scoring)

## Outputs

- Stratification table (e.g. mapk_drug_sensitivity_stratification.csv) with per-drug fold-change of median IC50 between pathway-active quartile and remaining lines, per mutation group
- Cell-line × pathway PROGENy score matrix with quartile assignments and TP53 status annotations

## How to apply

Load GDSC drug sensitivity IC50 values and cell-line mutation calls (BRAF, KRAS, NRAS, TP53) into R, then compute PROGENy pathway scores on the cell-line expression compendium using the progeny package (scores built from the 100 most significant pathway-responsive genes per pathway). Within each mutation group (MAPK-mutant vs wild-type), assign cell lines to MAPK pathway-score quartiles. Compute the fold change of median IC50 between the highest (MAPK-active) quartile and the remaining lines for the drugs of interest (e.g. AZ628, Trametinib, Dabrafenib), writing the stratification table to CSV. Rationale: PROGENy footprints recover functional consequences of driver mutations and improve drug-response markers, so pathway-active versus pathway-inactive lines within the same mutation class can differ dramatically in sensitivity (e.g. ~65-fold for AZ628, ~9130-fold for Trametinib among MAPK/BRAF-mutant lines).

## Related tools

- **progeny** (Computes PROGENy pathway activity scores (e.g. MAPK) from the cell-line gene expression compendium used for quartile stratification) — https://github.com/saezlab/progeny
- **knitr** (Used to generate the reproducible reports/supplementary figures accompanying the stratification analysis)
- **rmarkdown** (Vignette/report engine for the analysis workflows producing the stratification tables)

## Examples

```
scores <- progeny(expr_matrix, scale = TRUE, organism = "Human", top = 100)
```

## Evaluation signals

- The output CSV contains fold-change fields for AZ628, Trametinib, and Dabrafenib within MAPK-mutant lines and Trametinib within non-mutants
- Quartile assignment is computed within each mutation group (MAPK-mutant vs wild-type), not across the pooled panel
- TP53 status is tracked as a covariate/annotation for every stratified cell line
- Fold-change values are ratios of median IC50s and should reproduce the reported direction: pathway-active (top-quartile) MAPK-mutant lines are far more sensitive (lower IC50), e.g. ~65–9130 fold for the tested MAPK inhibitors
- PROGENy scores are computed with the standard signature (100 most significant responsive genes per pathway)

## Limitations

- Pathway scores from gene expression do not account for post-translational modifications; PROGENy's perturbation-derived footprints mitigate this but do not eliminate it
- Tissue-specific signatures may more closely reflect pathway activation than the consensus PROGENy signatures in a given context
- Pathway activity is not equivalent to pathway expression abundance; expression-level gene-set scores should not be interpreted as activity

## Evidence

- [readme] GDSC data used for cell-line pathway scoring: [evidence span withheld — anchor status: not_found]
- [abstract] PROGENy improves drug-indication markers: "(ii) provide or improve strong markers for drug indications"
- [intro] GDSC as preclinical drug-sensitivity resource: "the GDSC4,5 and CCLE6 have focussed on preclinical biomarkers of drug sensitivity in cancer cell lines"
- [readme] Signature construction uses 100 most significant genes: [evidence span withheld — anchor status: external_doc]
