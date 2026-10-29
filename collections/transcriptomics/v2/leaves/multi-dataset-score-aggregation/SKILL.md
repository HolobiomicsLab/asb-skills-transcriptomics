---
name: multi-dataset-score-aggregation
description: Use when per-cell scores from multiple stemness methods (ORIGINS activity,
  LandSCENT SR, CytoTRACE) have already been computed on several datasets and you
  need to quantify their agreement via averaged Pearson correlations.
when_to_use_negative:
- Per-cell pluripotency scores (ORIGINS activity, LandSCENT SR, CytoTRACE) have not
  yet been computed for the datasets — run the scoring workflows first
- The goal is to compute a single-method score from raw count matrices rather than
  to compare already-computed scores across methods and datasets
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: ORIGINS
  role: Provides the differentiation PPIN activity score per cell (activity and activity-HVF)
    that is correlated against other stemness scores
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Source of the SR (single-cell entropy-based pluripotency) score used as a
    correlation partner
- name: CytoTRACE
  role: Source of the CytoTRACE pluripotency score used as a correlation partner
- name: dplyr
  role: Used to summarize/average correlation coefficients across the four datasets
- name: magrittr
  role: Pipe operator for the dplyr summarization workflow
- name: R
  role: Implementation language; cor() computes the Pearson correlations between score
    pairs
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:55faf6b99fa6dac3865c827243590081f222e3c2294d2c9c00f69d66d8b1efdc
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-dataset-score-aggregation

## Summary

Aggregates per-cell pluripotency scores (ORIGINS activity, LandSCENT SR, CytoTRACE, activity-HVF) across multiple healthy human scRNA-seq datasets by computing pairwise Pearson correlations per dataset and averaging them with dplyr, enabling cross-dataset comparison of stemness methods.

## When to use

Use when per-cell scores from multiple stemness methods (ORIGINS activity, LandSCENT SR, CytoTRACE) have already been computed on several datasets and you need to quantify their agreement via averaged Pearson correlations.

## When NOT to use

- Per-cell pluripotency scores (ORIGINS activity, LandSCENT SR, CytoTRACE) have not yet been computed for the datasets — run the scoring workflows first
- The goal is to compute a single-method score from raw count matrices rather than to compare already-computed scores across methods and datasets

## Inputs

- Per-cell score table for ORIGINS activity per dataset (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383)
- Per-cell score table for ORIGINS activity-HVF per dataset
- Per-cell score table for LandSCENT SR per dataset
- Per-cell score table for CytoTRACE per dataset

## Outputs

- correlation_summary.csv containing computed and benchmark mean Pearson correlations for each score pair
- Per-dataset pairwise Pearson correlation coefficients (activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, activity vs activity-HVF)
- dplyr-averaged mean correlations per score pair across the four datasets

## How to apply

Load the per-cell score tables for ORIGINS activity, activity-HVF, LandSCENT SR, and CytoTRACE previously computed on each dataset (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383) in R. For each dataset, use R's cor() function to compute pairwise Pearson correlation coefficients for the pairs activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, and activity vs activity-HVF. Average the correlation coefficients for each score pair across the four datasets using dplyr summarization with magrittr pipes. Compare each computed mean against the reported benchmark averages (activity vs SR ~0.77, activity vs CytoTRACE ~0.44, SR vs CytoTRACE ~0.63, activity vs activity-HVF ~0.67) within small numerical tolerance, and write all values to correlation_summary.csv.

## Related tools

- **ORIGINS** (Provides the differentiation PPIN activity score per cell (activity and activity-HVF) that is correlated against other stemness scores) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Source of the SR (single-cell entropy-based pluripotency) score used as a correlation partner)
- **CytoTRACE** (Source of the CytoTRACE pluripotency score used as a correlation partner)
- **dplyr** (Used to summarize/average correlation coefficients across the four datasets)
- **magrittr** (Pipe operator for the dplyr summarization workflow)
- **R** (Implementation language; cor() computes the Pearson correlations between score pairs) — https://github.com/danielasenraoka/ORIGINS

## Examples

```
library(dplyr); cor_summary <- scores %>% group_by(pair) %>% summarise(mean_r = mean(pearson_r)); write.csv(cor_summary, "correlation_summary.csv")
```

## Evaluation signals

- Mean Pearson correlation for activity vs SR across the four datasets is ~0.77
- Mean Pearson correlation for activity vs CytoTRACE is ~0.44
- Mean Pearson correlation for SR vs CytoTRACE is ~0.63
- Mean Pearson correlation for activity vs activity-HVF is ~0.67
- All four score pairs are computed for each of the four datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383) and results are written to correlation_summary.csv

## Limitations

- Correlations were benchmarked only on four healthy human datasets (breast, colon, hematopoietic, lung); behavior on diseased or other tissue datasets is not covered by the reported averages
- The benchmark averages are approximate (~0.77, ~0.44, ~0.63, ~0.67), so validation must allow small numerical tolerance rather than exact equality

## Evidence

- [other] Across the four datasets, the average correlation coefficient was around 0.77 between activity (ORIGINS) and SR (LandSCENT), 0.44 between activity (ORIGINS) and CytoTRACE, and 0.63 between SR (LandSCENT) and CytoTRACE.: [evidence span withheld — anchor status: not_found]
- [other] Compute the pairwise Pearson correlation coefficients between activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, and activity vs activity-HVF using the cor() function in R for each of the four datasets.: [evidence span withheld — anchor status: not_found]
- [other] Average the correlation coefficients for each score pair across the four datasets with dplyr summarization.: [evidence span withheld — anchor status: not_found]
- [abstract] We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets"
- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
