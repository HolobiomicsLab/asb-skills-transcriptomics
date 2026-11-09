---
name: bulk-expression-gene-set-scoring
description: Use when you have a bulk RNA-seq/microarray expression matrix (e.g.,
  pdacR's TCGA-PAAD $ex) and a named gene signature (e.g., Bailey ADEX), and need
  a per-sample signature score to correlate with continuous metadata like ABSOLUTE
  purity.
when_to_use_negative:
- When the goal is single-cell resolution of cellular populations — the article notes
  bulk analyses limit the ability to fully separate cellular populations, and single-cell
  work belongs in the rmoffitt/scOh atlas instead
- When samples are not curated to PDAC (e.g., using all 182 TCGA-PAAD samples without
  the Decision/whitelist filter), since non-PDAC and low-purity cases introduce non-specificity
  to gene signatures
- When attempting to merge or aggregate scores across platforms (bulk RNA-seq vs microarrays),
  since the article found aggregation across heterogeneous platforms introduces more
  problems than it solves
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: R package providing the parsed TCGA-PAAD dataset, gene signatures object (Bailey
    ADEX), sample-track filtering, and expression-signature scoring/correlation visualization
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Language in which the signature scoring (mean log2 expression) and Spearman
    correlation against purity are computed
- name: devtools
  role: Used to install pdacR from GitHub prior to scoring
  repo: github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:7b9e0e0d0fe2820a5f5073a0a8eb06a645c05082f31206e9088c813aa882f3a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bulk-expression-gene-set-scoring

## Summary

Score a curated gene signature (e.g., the Bailey ADEX set) in bulk PDAC transcriptomic samples by computing the mean log2 expression of the signature genes per sample, then relate the score to sample-level phenotypes such as ABSOLUTE tumor purity via Spearman correlation in pdacR's TCGA-PAAD data.

## When to use

Use when you have a bulk RNA-seq/microarray expression matrix (e.g., pdacR's TCGA-PAAD $ex) and a named gene signature (e.g., Bailey ADEX), and need a per-sample signature score to correlate with continuous metadata like ABSOLUTE purity.

## When NOT to use

- When the goal is single-cell resolution of cellular populations — the article notes bulk analyses limit the ability to fully separate cellular populations, and single-cell work belongs in the rmoffitt/scOh atlas instead
- When samples are not curated to PDAC (e.g., using all 182 TCGA-PAAD samples without the Decision/whitelist filter), since non-PDAC and low-purity cases introduce non-specificity to gene signatures
- When attempting to merge or aggregate scores across platforms (bulk RNA-seq vs microarrays), since the article found aggregation across heterogeneous platforms introduces more problems than it solves

## Inputs

- pdacR TCGA-PAAD expression matrix ($ex, log2-transformed, genes as rows by SYMBOL)
- pdacR $sampInfo sample metadata including the 'Decision' track with 'whitelist' labels
- ABSOLUTE-estimated tumor purity values keyed by sample ID
- Gene signature object from pdacR data/ (Bailey ADEX gene list by SYMBOL)

## Outputs

- Per-sample ADEX signature score (mean log2 expression across matched genes)
- Spearman correlation statistics (rho = -0.260, p = 0.0014, n = 150 on whitelisted TCGA-PAAD)
- Scatter plot of ADEX score versus tumor purity annotated with rho and p-value (Fig. 2a)

## How to apply

Install pdacR via devtools::install_github('rmoffitt/pdacR') and load the parsed TCGA-PAAD dataset ($ex expression matrix, $sampInfo metadata) plus the gene signatures object from the package data/ directory. First curate the cohort: apply the 'Decision' sample track filter, removing all labels except 'whitelist', which retains the 150 histologically confirmed PDAC tumors out of the original 182 — including the 32 non-PDAC cases would contaminate the signature-purity relationship. Match the Bailey ADEX genes to the dataset's gene rows by SYMBOL, then score each sample as the mean log2 expression across the matched ADEX genes. Merge ABSOLUTE-estimated tumor purity onto the 150-sample metadata by sample ID and compute the Spearman rank correlation between the ADEX score and purity, reporting rho, the p-value, and n=150 (the published value is rho = -0.260, p = 0.0014). Visualize as a scatter plot of ADEX score versus purity annotated with rho and p, reproducing Fig. 2a; the same gene-set scoring approach also feeds heatmapping, correlation views, dimensionality reduction, and DE coloring in pdacR.

## Related tools

- **pdacR** (R package providing the parsed TCGA-PAAD dataset, gene signatures object (Bailey ADEX), sample-track filtering, and expression-signature scoring/correlation visualization) — github.com/rmoffitt/pdacR
- **R** (Language in which the signature scoring (mean log2 expression) and Spearman correlation against purity are computed)
- **devtools** (Used to install pdacR from GitHub prior to scoring) — github.com/rmoffitt/pdacR

## Examples

```
devtools::install_github('rmoffitt/pdacR'); score <- colMeans(tcga$ex[bailey_adex_genes, wl], na.rm=TRUE); cor.test(score, purity, method='spearman')
```

## Evaluation signals

- After applying the Decision/whitelist filter, exactly n = 150 histologically confirmed PDAC tumors remain from the original 182 TCGA-PAAD samples
- Computed Spearman rho is within [-1, 1] with an associated p-value, and is consistent with the reported Fig. 2a values (rho = -0.260, p = 0.0014)
- All Bailey ADEX genes used for scoring are matched to dataset gene rows by SYMBOL before averaging
- ADEX scores are means of log2 expression values across the signature genes for each sample, not sums or raw counts
- Every scored sample has a merged ABSOLUTE purity value matched by sample ID

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so the signature-purity correlation reflects bulk composition rather than tumor-cell-intrinsic expression
- TCGA-PAAD contains 182 samples but only 150 are histologically confirmed PDAC; failing to filter skews signature scores and correlations
- Multi-platform data (bulk RNA-seq and microarrays) cannot be robustly merged, so scores are dataset-specific and not directly comparable across datasets

## Evidence

- [readme] For example, TCGA-PAAD has 182 samples, but only 150 are PDAC.: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [intro] The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [readme] Filtering rationale and whitelist: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [readme] Gene set usage in pdacR: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [discussion] Bulk data limitation: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
