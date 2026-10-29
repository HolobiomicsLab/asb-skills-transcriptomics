---
name: mutation-based-cell-line-stratification
description: Use when you have GDSC IC50 drug-sensitivity data plus cell-line mutation
  calls and gene expression, and you want to know whether pathway activity scores
  stratify drug response within mutant or wild-type sub-populations.
when_to_use_negative:
- Data are mouse or non-human cell lines without confirming PROGENy applicability
  (the package notes the original release was developed for human data)
- The question is only mutation-vs-drug association without expression data — no pathway
  scores can be computed
- Analysis targets primary tumors with tissue-matched normals for SPIA/Pathifier-style
  comparisons rather than GDSC cell-line drug response
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3053
tools:
- name: progeny
  role: Bioconductor R package holding the pathway-responsive gene signatures used
    to infer MAPK (and other) pathway activity scores from the cell-line expression
    matrix
  repo: https://github.com/saezlab/progeny
- name: knitr
  role: R engine used to knit the reproducible report chunks in the analysis workflow
- name: rmarkdown
  role: Vignette/report rendering engine for the analysis (VignetteEngine{knitr::rmarkdown})
- name: decoupler
  role: Recommended companion for inferring pathway activities from the PROGENy signatures
    with a statistical method
  repo: https://github.com/saezlab/decoupler-py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:b3f68dd748cde1ca4a33a744b9b8e356aaa507ec3bb38682b9985b2d19ae5af6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - decoupler
  - knitr
  - progeny
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# mutation-based-cell-line-stratification

## Summary

Stratify GDSC cancer cell lines by MAPK-pathway driver mutation status (BRAF/KRAS/NRAS) with TP53 tracked, then further partition each stratum by PROGENy MAPK pathway-score quartile to test whether expression-derived pathway activity adds drug-sensitivity resolution beyond mutation calls alone.

## When to use

Use when you have GDSC IC50 drug-sensitivity data plus cell-line mutation calls and gene expression, and you want to know whether pathway activity scores stratify drug response within mutant or wild-type sub-populations.

## When NOT to use

- Data are mouse or non-human cell lines without confirming PROGENy applicability (the package notes the original release was developed for human data)
- The question is only mutation-vs-drug association without expression data — no pathway scores can be computed
- Analysis targets primary tumors with tissue-matched normals for SPIA/Pathifier-style comparisons rather than GDSC cell-line drug response

## Inputs

- GDSC drug sensitivity IC50 values (from cancerrxgene.org/gdsc1000)
- Cell-line mutation calls for BRAF, KRAS, NRAS, TP53
- Cell-line gene expression compendium (bulk transcriptomics, human)

## Outputs

- mapk_drug_sensitivity_stratification.csv with per-drug median-IC50 fold-change fields for MAPK-active vs remaining lines within mutant and wild-type strata
- Per-cell-line PROGENy pathway scores including the MAPK score

## How to apply

Load GDSC IC50 values and mutation calls for BRAF, KRAS, NRAS, and TP53 in R, then compute PROGENy pathway scores over the cell-line expression compendium with the progeny package. Split cell lines into MAPK-mutant (BRAF/KRAS/NRAS) and wild-type groups, tracking TP53 status throughout. Within each group, assign each line to a MAPK pathway-score quartile so the top quartile marks 'MAPK-active' lines. Compute the fold change of the median IC50 between the MAPK-active top quartile and the remaining lines for AZ628, Trametinib, and Dabrafenib within mutants, and for Trametinib within non-mutants; write the table to mapk_drug_sensitivity_stratification.csv. The rationale is that PROGENy signaling footprints recover the functional effect of driver mutations and 'provide or improve strong markers for drug indications', e.g. MAPK-mutant lines in the active quartile are 9130-fold more sensitive to Trametinib than MAPK-inactive lines.

## Related tools

- **progeny** (Bioconductor R package holding the pathway-responsive gene signatures used to infer MAPK (and other) pathway activity scores from the cell-line expression matrix) — https://github.com/saezlab/progeny
- **knitr** (R engine used to knit the reproducible report chunks in the analysis workflow)
- **rmarkdown** (Vignette/report rendering engine for the analysis (VignetteEngine{knitr::rmarkdown}))
- **decoupler** (Recommended companion for inferring pathway activities from the PROGENy signatures with a statistical method) — https://github.com/saezlab/decoupler-py

## Examples

```
BiocManager::install("progeny"); scores <- progeny(expr_matrix, scale = TRUE); quart <- cut(scores$MAPK, quantile(scores$MAPK, 0:4/4), include.lowest = TRUE)
```

## Evaluation signals

- Reproduced fold changes: AZ628 ~65-fold, Trametinib ~9130-fold, Dabrafenib ~104-fold within MAPK-mutant lines; a Trametinib fold change also computed within non-mutants
- Every cell line in the table carries both a mutation group label (MAPK-mutant vs wild-type, BRAF/KRAS/NRAS) and a TP53 status
- MAPK pathway-score quartiles are computed WITHIN each mutation group, not across all lines pooled
- mapk_drug_sensitivity_stratification.csv exists with the expected fold-change fields and parses cleanly in R
- PROGENy scores cover the GDSC cell-line compendium (one score row per cell line, 11 pathway columns per the footprints README)

## Limitations

- Expression level of a pathway gene set should not be interpreted as its activity; only perturbation-derived footprints like PROGENy reflect signaling state (directed-path methods cover more pathways but read out abundance)
- Tissue- or context-specific signatures may more closely reflect pathway activation than the consensus PROGENy signatures in a given context
- PROGENy pathway scores from gene expression do not account for post-translational modifications; scores are inferred, not measured
- Results depend on GDSC data availability from cancerrxgene.org/gdsc1000 and on quartile binning, which is sensitive to cell-line count per stratum

## Evidence

- [abstract] Drug-response stratification findings: "PROGENy can (i) recover the effect of known driver mutations"
- [intro] GDSC data source: "the GDSC4,5 and CCLE6 have focussed on preclinical biomarkers of drug sensitivity in cancer cell lines"
- [readme] GDSC cell-line score computation: [evidence span withheld — anchor status: not_found]
- [readme] PROGENy R package availability: [evidence span withheld — anchor status: external_doc]
- [methods] Analysis environment in the article methods: "package: "`r pkg_ver('progeny')`""
