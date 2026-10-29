---
name: drug-sensitivity-fold-change-analysis
description: Use when GDSC drug sensitivity IC50 values, cell-line mutation calls,
  and a cell-line expression compendium are available and the goal is to test whether
  pathway activity scores refine drug-sensitivity stratification beyond driver mutation
  status alone.
when_to_use_negative:
- When no cell-line expression data are available to compute PROGENy pathway scores
  — mutation status alone cannot define pathway-activity quartiles
- When the analysis target is pathway activity in primary tumors rather than drug
  sensitivity in GDSC cell lines (use the tcga_mutation or tcga_survival analyses
  instead)
- When the drug of interest has no measurable IC50 values in both MAPK-mutant and
  MAPK wild-type cell-line sub-populations
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: progeny
  role: R/Bioconductor package used to compute PROGENy pathway scores from the cell-line
    expression compendium
  repo: github.com/saezlab/progeny
- name: knitr
  role: Used to auto-generate supplementary reports of the stratification analyses
- name: rmarkdown
  role: Vignette/report engine for the analysis workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:e11bdc7c53a65368fb617409a3ea2d0273fbc4dbe19ac3e1e620a304874f0058
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - knitr
  - progeny
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# drug-sensitivity-fold-change-analysis

## Summary

A stratification analysis that computes PROGENy pathway scores (e.g., MAPK) for GDSC cancer cell lines, bins cell lines into pathway-score quartiles within mutation-status groups (BRAF/KRAS/NRAS, TP53), and reports fold changes of median drug IC50 between pathway-active and pathway-inactive sub-populations for targeted inhibitors such as AZ628, Trametinib, and Dabrafenib.

## When to use

Use when GDSC drug sensitivity IC50 values, cell-line mutation calls, and a cell-line expression compendium are available and the goal is to test whether pathway activity scores refine drug-sensitivity stratification beyond driver mutation status alone.

## When NOT to use

- When no cell-line expression data are available to compute PROGENy pathway scores — mutation status alone cannot define pathway-activity quartiles
- When the analysis target is pathway activity in primary tumors rather than drug sensitivity in GDSC cell lines (use the tcga_mutation or tcga_survival analyses instead)
- When the drug of interest has no measurable IC50 values in both MAPK-mutant and MAPK wild-type cell-line sub-populations

## Inputs

- GDSC drug sensitivity IC50 values per cell line and drug (from cancerrxgene.org/gdsc1000)
- Cell-line mutation calls for BRAF, KRAS, NRAS, and TP53
- Cell-line gene expression compendium (GDSC 1000 expression data)

## Outputs

- mapk_drug_sensitivity_stratification.csv containing per-drug, per-mutation-group fold-change fields
- Fold-change of median IC50 between MAPK-active (top quartile) and remaining cell lines (e.g., AZ628 65-fold, Trametinib 9130-fold, Dabrafenib 104-fold within MAPK/BRAF mutants)

## How to apply

Load GDSC drug sensitivity IC50 values and cell-line mutation calls (BRAF, KRAS, NRAS, TP53) into R, then compute PROGENy pathway scores over the cell-line expression compendium using the progeny package. Assign each cell line to a MAPK pathway-score quartile within the MAPK-mutant (BRAF/KRAS/NRAS) group and the wild-type group, tracking TP53 status alongside. Compute the fold change of the median IC50 between the highest (MAPK-active) quartile and the remaining lines for AZ628, Trametinib, and Dabrafenib within mutants, and for Trametinib within non-mutants. Rationale: PROGENy leverages a large compendium of perturbation experiments to infer pathway activity from gene expression, recovering the functional impact of driver mutations and providing markers for drug indications in cell lines; the GDSC data are taken from cancerrxgene.org/gdsc1000. Write the stratification table with fold-change fields to mapk_drug_sensitivity_stratification.csv (report generation can use knitr/rmarkdown as in the footprints analyses).

## Related tools

- **progeny** (R/Bioconductor package used to compute PROGENy pathway scores from the cell-line expression compendium) — github.com/saezlab/progeny
- **knitr** (Used to auto-generate supplementary reports of the stratification analyses)
- **rmarkdown** (Vignette/report engine for the analysis workflow)

## Examples

```
BiocManager::install("progeny"); scores <- progeny(expr_matrix, scale=TRUE); write.csv(mapk_drug_sensitivity_stratification, "mapk_drug_sensitivity_stratification.csv")
```

## Evaluation signals

- Stratification CSV exists with fold-change fields for all required drug/mutation-group combinations (AZ628, Trametinib, Dabrafenib within mutants; Trametinib within wild-type)
- Fold changes are reported as ratios of median IC50 (top MAPK-score quartile vs. remaining lines), with MAPK-active lines more sensitive (larger fold change) for MAPK-targeted inhibitors
- Quartile assignment is performed within mutation strata (mutant and wild-type separately), not across the full cell-line panel
- Each cell line carries both a MAPK pathway-score quartile label and a TP53 status annotation
- Reproducing the PROGENy score computation on the same expression compendium should recover the same quartile memberships (deterministic scores)

## Limitations

- PROGENy provides a good estimate of pathway activity in large and heterogeneous data sets, but tissue-specific signatures may more closely reflect activation status in a specific context
- Expression-based pathway scores do not directly account for post-translational modifications, so pathway score does not always equal signaling activity
- Treats pathway activity as quartile-binned rather than continuous, which can mask dose-response-like relationships within the top quartile

## Evidence

- [abstract] PROGENy can (i) recover the effect of known driver mutations: "PROGENy can (i) recover the effect of known driver mutations"
- [abstract] drug indication markers: "(ii) provide or improve strong markers for drug indications"
- [readme] GDSC cell line pathway scores: [evidence span withheld — anchor status: not_found]
- [intro] PROGENy leverages perturbation compendium: "we overcome the limitations of both approaches by leveraging a large compendium of publicly available perturbation experiments that yield a common core of Pathway RespOnsive GENes"
- [methods] report generation with knitr: "knitr::opts_chunk$set("
