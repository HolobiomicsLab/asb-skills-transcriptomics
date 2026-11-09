---
name: fdr-thresholded-differential-expression-calling
description: Use when you have per-gene FDR result tables from RNA-seq mixed model
  fits (e.g. kimma lme/lmerel models) and need to define DEG sets at FDR < 0.05, or
  compare DEG detection between two model formulations.
when_to_use_negative:
- Input is raw count data or expression matrices that have not yet been fit by a mixed
  model (kimma kmFit) — FDR values must already exist per gene
- The goal is to fit or compare models themselves (e.g. selecting between kimma_lme
  and kimma_lmerel by AIC) rather than thresholding and comparing existing FDR results
- Analysis calls DEGs from an unpaired or non-related-cohort design where kinship
  covariance does not apply — a standard limma-style DEG call without model-set comparison
  is more appropriate
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: Source of the per-gene FDR result tables via kmFit (kimma_lme / kimma_lmerel)
    and of the summarise_kmFit function whose fdr.cutoff argument applies the FDR
    < 0.05 threshold
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Execution environment in which the FDR thresholding and set comparison are
    performed
  repo: https://github.com/BIGslu/kimma
- name: dplyr
  role: Used to compute set comparisons between the kinship and non-kinship DEG sets
- name: data.table
  role: Used for data manipulation when computing DEG set comparisons
- name: tibble
  role: Used to structure the comparison results before writing the CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:b117f4a4cd9d9fe61c19c3c89493ce87f920252acc67edc92914cf9ba598632c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - data.table
  - dplyr
  - kimma
  - tibble
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fdr-thresholded-differential-expression-calling

## Summary

Calls differentially expressed genes (DEGs) by thresholding per-gene FDR values from linear mixed model fits (e.g. kimma kmFit) at FDR < 0.05, and compares DEG sets across model specifications such as paired models with versus without a kinship covariance random effect.

## When to use

Use when you have per-gene FDR result tables from RNA-seq mixed model fits (e.g. kimma lme/lmerel models) and need to define DEG sets at FDR < 0.05, or compare DEG detection between two model formulations.

## When NOT to use

- Input is raw count data or expression matrices that have not yet been fit by a mixed model (kimma kmFit) — FDR values must already exist per gene
- The goal is to fit or compare models themselves (e.g. selecting between kimma_lme and kimma_lmerel by AIC) rather than thresholding and comparing existing FDR results
- Analysis calls DEGs from an unpaired or non-related-cohort design where kinship covariance does not apply — a standard limma-style DEG call without model-set comparison is more appropriate

## Inputs

- Per-gene FDR result tables from kimma_lmerel (kinship) model fit on the example cohort
- Per-gene FDR result tables from kimma_lme (non-kinship) model fit on the example cohort
- FDR cutoff value (fdr.cutoff = 0.05, the fdr.cutoff argument of summarise_kmFit)

## Outputs

- Kinship-model DEG set (genes with FDR < 0.05)
- Non-kinship-model DEG set (genes with FDR < 0.05)
- Set comparison counts: DEGs newly detected with kinship, DEGs only without kinship, and percentage agreement between models
- deg_kinship_comparison.csv containing comparison counts and agreement percentage

## How to apply

Load the per-gene FDR result tables produced by the upstream kimma model fits (kimma_lmerel for the kinship model, kimma_lme for the non-kinship paired model). Threshold each gene at FDR < 0.05 — the fdr.cutoff argument of summarise_kmFit — to define each model's DEG set in R. Compute set comparisons with dplyr/data.table/tibble: DEGs newly detected with kinship, DEGs detected only without kinship, and the percentage agreement of calls between the two models. In the source task, this comparison revealed 25 new DEGs detected in the kinship model at FDR < 0.05, demonstrating that kinship covariance impacts DEG detection in the paired Mtb infected versus media model of related individuals. Write the comparison counts and agreement percentage to a CSV (e.g. deg_kinship_comparison.csv) for downstream inspection.

## Related tools

- **kimma** (Source of the per-gene FDR result tables via kmFit (kimma_lme / kimma_lmerel) and of the summarise_kmFit function whose fdr.cutoff argument applies the FDR < 0.05 threshold) — https://github.com/BIGslu/kimma
- **R** (Execution environment in which the FDR thresholding and set comparison are performed) — https://github.com/BIGslu/kimma
- **dplyr** (Used to compute set comparisons between the kinship and non-kinship DEG sets)
- **data.table** (Used for data manipulation when computing DEG set comparisons)
- **tibble** (Used to structure the comparison results before writing the CSV)

## Examples

```
library(kimma); degs_kin <- subset(kmFit_lmerel_results, FDR < 0.05); degs_nokin <- subset(kmFit_lme_results, FDR < 0.05); write.csv(data.frame(new_with_kinship = setdiff(degs_kin$gene, degs_nokin$gene)), "deg_kinship_comparison.csv")
```

## Evaluation signals

- Each gene in each model's result table is classified as DEG or non-DEG exactly once, with the classification determined solely by FDR < 0.05 (the fdr.cutoff of summarise_kmFit)
- The union of 'newly detected with kinship', 'only without kinship', and 'shared' DEGs equals the union of the two model DEG sets (set-comparison invariant)
- The percentage agreement of calls between the two models is a valid proportion between 0 and 100%
- Output file deg_kinship_comparison.csv contains both comparison counts and the agreement percentage
- In the source cohort, 25 new DEGs were detected in the kinship model at FDR < 0.05 — a nonzero set difference between models is expected when kinship impacts DEG detection

## Limitations

- Thresholding is sensitive to the chosen cutoff: FDR < 0.05 is the cutoff used here, but genes near the boundary may flip between models, inflating apparent set differences
- Set differences reflect model specification (kinship covariance in the paired Mtb model of related individuals), not biological validation — newly detected DEGs in the kinship model are not independently confirmed
- kimma's kinship-based models require a kinship matrix (example.kin) and related individuals; without genetically related samples, the kinship versus non-kinship comparison is not applicable

## Evidence

- [other] Threshold each gene at FDR < 0.05 (the fdr.cutoff argument of summarise_kmFit) to define the kinship-model DEG set and the non-kinship-model DEG set in R: [evidence span withheld — anchor status: not_found]
- [other] Adding a kinship covariance random effect changed DEG detection in the paired Mtb model of related individuals, with 25 new DEGs detected in the kinship model at FDR < 0.05: [evidence span withheld — anchor status: not_found]
- [abstract] kimma revealed that kinship impacts model fit and DEG detection in a related cohort: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [abstract] kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models: "kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models"
- [methods] summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff): "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
