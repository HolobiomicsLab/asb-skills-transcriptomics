---
name: pearson-correlation-analysis-of-sc-rna-seq-pluripotency-scores
description: Use when per-cell pluripotency scores from ORIGINS activity, LandSCENT
  SR, and CytoTRACE are already computed and you need to quantify and validate their
  pairwise agreement via Pearson correlations across datasets.
when_to_use_negative:
- Per-cell score tables have not yet been computed — run ORIGINS activity(), LandSCENT,
  and CytoTRACE scoring first
- The goal is to benchmark computational efficiency (runtime, RAM) rather than score
  agreement — ORIGINS vs LandSCENT/CytoTRACE efficiency comparisons need a different
  workflow
- The datasets are not the healthy human tissues the benchmark covers (breast, colon,
  hematopoietic, lung) and no benchmark averages are available for validation
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3300
tools:
- name: ORIGINS
  role: R package that computes the differentiation PPIN activity score per cell;
    its activity and activity-HVF outputs are two of the four scores correlated here
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Published pluripotency tool providing the SR (significant random-walk) score
    compared against ORIGINS activity
- name: CytoTRACE
  role: Published pluripotency tool providing the CytoTRACE score compared against
    ORIGINS activity and LandSCENT SR
- name: R
  role: Environment in which cor() computes pairwise Pearson correlations and dplyr/magrittr
    summarize averages across the four datasets
  repo: https://github.com/danielasenraoka/ORIGINS
- name: dplyr
  role: Imported by ORIGINS; used for summarizing mean correlation coefficients across
    datasets
  repo: https://github.com/danielasenraoka/ORIGINS
- name: magrittr
  role: Imported by ORIGINS; provides the pipe operator used in the tidy correlation
    workflow
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:258f28d589cbdad282c3b76c27f96e7ec45d7416ead906153cffe4936b4420ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - dplyr
  - magrittr
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pearson-correlation-analysis-of-sc-rna-seq-pluripotency-scores

## Summary

Reproduces the pairwise Pearson correlation matrix between pluripotency scores (ORIGINS activity, ORIGINS activity-HVF, LandSCENT SR, and CytoTRACE) across the four healthy human scRNA-seq datasets used to benchmark ORIGINS, averaging coefficients per score pair and comparing against the reported benchmark averages.

## When to use

Use when per-cell pluripotency scores from ORIGINS activity, LandSCENT SR, and CytoTRACE are already computed and you need to quantify and validate their pairwise agreement via Pearson correlations across datasets.

## When NOT to use

- Per-cell score tables have not yet been computed — run ORIGINS activity(), LandSCENT, and CytoTRACE scoring first
- The goal is to benchmark computational efficiency (runtime, RAM) rather than score agreement — ORIGINS vs LandSCENT/CytoTRACE efficiency comparisons need a different workflow
- The datasets are not the healthy human tissues the benchmark covers (breast, colon, hematopoietic, lung) and no benchmark averages are available for validation

## Inputs

- Per-cell pluripotency score tables for ORIGINS activity, ORIGINS activity-HVF, LandSCENT SR, and CytoTRACE (one table per dataset)
- scRNA-seq datasets GSE113197 (breast epithelial), GSE125970 (colon epithelial), GSE117498 (hematopoietic bone marrow), GSE161383 (lung)
- R expression matrix (rows = genes, columns = cells) plus differentiation_edges PPIN if scores need recomputation

## Outputs

- correlation_summary.csv containing per-dataset and mean Pearson correlation coefficients for each score pair
- Mean Pearson correlations per pair across the four datasets for benchmark comparison

## How to apply

Load the previously computed per-cell score tables for ORIGINS activity, activity-HVF, LandSCENT SR, and CytoTRACE in R for the breast (GSE113197), colon (GSE125970), hematopoietic (GSE117498), and lung (GSE161383) datasets. Use R's cor() function to compute pairwise Pearson coefficients for activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, and activity vs activity-HVF within each dataset. Average each score pair's coefficients across the four datasets with dplyr summarization (the ORIGINS package imports dplyr and magrittr for this style of tidy pipeline). Compare each computed mean against the reported benchmark averages — activity vs SR ~0.77, activity vs CytoTRACE ~0.44, SR vs CytoTRACE ~0.63, activity vs activity-HVF ~0.67 — and write all values to correlation_summary.csv. If ORIGINS scores must be recomputed first, pass the gene-by-cell expression matrix and the package-loaded differentiation_edges PPIN to the activity() function.

## Related tools

- **ORIGINS** (R package that computes the differentiation PPIN activity score per cell; its activity and activity-HVF outputs are two of the four scores correlated here) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Published pluripotency tool providing the SR (significant random-walk) score compared against ORIGINS activity)
- **CytoTRACE** (Published pluripotency tool providing the CytoTRACE score compared against ORIGINS activity and LandSCENT SR)
- **R** (Environment in which cor() computes pairwise Pearson correlations and dplyr/magrittr summarize averages across the four datasets) — https://github.com/danielasenraoka/ORIGINS
- **dplyr** (Imported by ORIGINS; used for summarizing mean correlation coefficients across datasets) — https://github.com/danielasenraoka/ORIGINS
- **magrittr** (Imported by ORIGINS; provides the pipe operator used in the tidy correlation workflow) — https://github.com/danielasenraoka/ORIGINS

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges); mean_cor <- scores_df %>% group_by(score_pair) %>% summarise(mean_pearson = mean(cor_val))
```

## Evaluation signals

- Mean Pearson correlation of activity vs SR across the four datasets falls near the benchmark ~0.77 within small numerical tolerance
- Mean Pearson correlation of activity vs CytoTRACE falls near ~0.44
- Mean Pearson correlation of SR vs CytoTRACE falls near ~0.63
- Mean Pearson correlation of activity vs activity-HVF falls near ~0.67
- correlation_summary.csv is produced with per-dataset and averaged coefficients for all four score pairs

## Limitations

- Validation depends on per-cell score tables computed beforehand with the same preprocessing; mismatches in score computation will shift correlations away from the benchmark values
- The benchmark averages were derived specifically from four healthy human datasets (breast, colon, hematopoietic, lung); other tissues or disease states are not covered
- Pearson correlation measures linear agreement only and is sensitive to differences in score normalization (e.g., ORIGINS2's normalization parameter controls whether activity is divided by the number of PPIN edges)

## Evidence

- [other] Average pairwise Pearson correlations across datasets (activity vs SR ~0.77, activity vs CytoTRACE ~0.44, SR vs CytoTRACE ~0.63, activity vs activity-HVF ~0.67) are the validation benchmark.: [evidence span withheld — anchor status: not_found]
- [supplementary] Score tables are computed on the four benchmark GEO datasets before correlations are calculated.: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] The ORIGINS activity() call over the expression matrix and package-loaded differentiation PPIN supplies the score that enters the correlation workflow.: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [abstract] ORIGINS and the competing tools were benchmarked on the four healthy human datasets used in this correlation analysis.: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,"
