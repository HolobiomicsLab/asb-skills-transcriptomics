---
name: tumor-purity-association-analysis
description: Use when scoring a bulk PDAC RNA-seq signature in TCGA-PAAD and asking
  whether its expression reflects tumor content rather than stromal/normal contamination,
  via correlation with ABSOLUTE purity.
when_to_use_negative:
- Analysis goal is subtyping patients (e.g., PurIST or basal-like/classical calls)
  rather than testing whether a signature tracks tumor cellularity/purity.
- Input is single-cell data (e.g., the rmoffitt/scOh atlas), where purity confounding
  of bulk signatures does not apply.
- The dataset has no sample-level purity or histologic confirmation track (e.g., no
  'Decision' whitelist), so the 150-sample curated PDAC cohort cannot be reproduced.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: Supplies the parsed TCGA-PAAD expression matrix, sample metadata (Decision/whitelist
    track), gene signatures object, and heatmap/Cartesian plotting functions used
    to score ADEX and correlate it with purity
  repo: https://github.com/rmoffitt/pdacR.git
- name: devtools
  role: Installs the pdacR package from GitHub
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Runtime environment in which the signature scoring, Spearman correlation,
    and scatter plotting are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:0433af82fdeeee1fd169e6badf21c8046b13f80ab8caff757ad3b9e5634974d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tumor-purity-association-analysis

## Summary

Quantifies whether a PDAC gene-expression signature (e.g., the Bailey ADEX geneset) tracks tumor cellularity by correlating signature scores with ABSOLUTE-estimated tumor purity in the curated TCGA-PAAD cohort from pdacR, exposing purity confounders in bulk transcriptomic subtyping.

## When to use

Use when scoring a bulk PDAC RNA-seq signature in TCGA-PAAD and asking whether its expression reflects tumor content rather than stromal/normal contamination, via correlation with ABSOLUTE purity.

## When NOT to use

- Analysis goal is subtyping patients (e.g., PurIST or basal-like/classical calls) rather than testing whether a signature tracks tumor cellularity/purity.
- Input is single-cell data (e.g., the rmoffitt/scOh atlas), where purity confounding of bulk signatures does not apply.
- The dataset has no sample-level purity or histologic confirmation track (e.g., no 'Decision' whitelist), so the 150-sample curated PDAC cohort cannot be reproduced.

## Inputs

- pdacR parsed TCGA-PAAD dataset ($ex log2 expression matrix and $sampInfo sample metadata)
- pdacR gene signatures object containing the Bailey ADEX geneset (SYMBOL-matched gene rows)
- ABSOLUTE-estimated tumor purity values keyed by TCGA sample ID

## Outputs

- Spearman correlation statistics (rho, p-value, n) between ADEX signature score and tumor purity (expected: rho = -0.260, p = 0.0014, n = 150)
- Scatter plot of ADEX score versus ABSOLUTE purity annotated with rho and p-value (Fig. 2a reproduction)

## How to apply

Install pdacR via devtools::install_github('rmoffitt/pdacR') and load the parsed TCGA-PAAD dataset ($ex expression matrix, $sampInfo metadata) plus the gene signatures object from the package data/ directory. Apply the 'Decision' sample track filter, removing all labels except 'whitelist', which retains the 150 histologically confirmed PDAC samples out of the original 182 — this step is essential because the other 32 samples are non-PDAC tumors. Score the signature per sample as the mean log2 expression across the signature's genes matched by SYMBOL, merge ABSOLUTE-estimated tumor purity onto the metadata by sample ID, and compute the Spearman rank correlation (rho, p-value, n=150). Plot signature score versus purity as a scatter annotated with rho and p-value (Fig. 2a style) to judge whether the signature is purity-confounded.

## Related tools

- **pdacR** (Supplies the parsed TCGA-PAAD expression matrix, sample metadata (Decision/whitelist track), gene signatures object, and heatmap/Cartesian plotting functions used to score ADEX and correlate it with purity) — https://github.com/rmoffitt/pdacR.git
- **devtools** (Installs the pdacR package from GitHub) — github.com/rmoffitt/pdacR
- **R** (Runtime environment in which the signature scoring, Spearman correlation, and scatter plotting are executed)

## Examples

```
devtools::install_github('rmoffitt/pdacR'); score <- colMeans(ex[match(bailey_adex, rownames(ex)), ]); cor.test(score, purity, method='spearman')
```

## Evaluation signals

- Output cohort size is exactly n = 150 whitelisted PDAC tumors after applying the Decision/whitelist filter (182 TCGA-PAAD samples in, 32 non-PDAC removed).
- Reported Spearman rho is within [-1, 1] with an associated p-value, and rho/p/n are consistent with the reported Fig. 2a values (rho = -0.260, p = 0.0014).
- ADEX genes are matched to dataset rows by SYMBOL, and the per-sample score equals the mean log2 expression across matched signature genes.
- Scatter plot axes are ADEX score vs ABSOLUTE purity and the annotation shows both rho and p-value.

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signature–purity correlations cannot be definitively attributed to tumor cells.
- Inappropriate use of TCGA-PAAD without the Decision/whitelist filter (keeping all 182 samples, including 32 non-PDAC) misleads signature validation.
- Datasets cannot be robustly merged across platforms (bulk RNA-seq vs microarrays), so this correlation should be run within a single curated dataset.

## Evidence

- [methods] Filter to 150 whitelisted PDAC samples: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [readme] Decision/whitelist filter step: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [readme] Filtering mechanism: "Checking a box in this list will remove all samples with that label from all analyses."
- [intro] Pitfall of non-PDAC contamination: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [discussion] Bulk data limitation: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
