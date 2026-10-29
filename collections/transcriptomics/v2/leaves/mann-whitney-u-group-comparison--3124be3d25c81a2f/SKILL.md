---
name: mann-whitney-u-group-comparison
description: Use when testing whether a SenePy sp_score (or similar cell-level score)
  differs between two independent sample groups — e.g., COVID-19 decedents vs controls
  — after aggregating per-cell scores to per-sample means with small, non-normal samples.
when_to_use_negative:
- Design is paired or matched per-donor (e.g., pre/post within the same sample) —
  the Mann-Whitney U test assumes independent samples.
- Comparing per-cell sp_scores directly without aggregating to per-sample means —
  per-cell scores violate independence and inflate significance.
- Expression matrix is already scaled — score_hub scoring must be run on raw or normalized-but-unscaled
  data.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SenePy
  role: Generates the per-cell senescence sp_score (via score_hub with binarize=True
    and hub-network importance weighting) that is aggregated per sample before the
    Mann-Whitney test
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Holds the single-cell expression matrix and stores scores in adata.obs['sp_score']
    prior to per-sample aggregation
- name: scanpy
  role: AnnData-integrated preprocessing (normalize_total/log1p optional under binarized
    default scoring) of the scRNA-seq input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:191f0915a2cc3ea70192833ce32ebe10f3300f0405c48aa4e6f35521efe95246
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# mann-whitney-u-group-comparison

## Summary

Two-tailed nonparametric Mann-Whitney U comparison of per-sample senescence scores between independent groups, as used to show increased senescent lung epithelial cells (AT1, AT2, airway) in COVID-19 decedents vs age-matched controls (p = 0.004) with SenePy-scored data.

## When to use

Use when testing whether a SenePy sp_score (or similar cell-level score) differs between two independent sample groups — e.g., COVID-19 decedents vs controls — after aggregating per-cell scores to per-sample means with small, non-normal samples.

## When NOT to use

- Design is paired or matched per-donor (e.g., pre/post within the same sample) — the Mann-Whitney U test assumes independent samples.
- Comparing per-cell sp_scores directly without aggregating to per-sample means — per-cell scores violate independence and inflate significance.
- Expression matrix is already scaled — score_hub scoring must be run on raw or normalized-but-unscaled data.

## Inputs

- Per-cell AnnData scRNA-seq expression matrix (raw or normalized, not scaled), stored in .h5ad via AnnData/scanpy
- Human cell-type senescence hubs from sp.load_hubs(species='Human') with hubs.metadata (tissue, cell, hub_num)
- Sample-level metadata labeling group membership (COVID-19 decedent vs age-matched control)

## Outputs

- Per-sample mean epithelial senescence scores with outlier annotations (e.g., covid_epithelial_senescence_scores.csv)
- Mann-Whitney U statistic and two-tailed p-value comparing the two groups

## How to apply

First produce per-cell scores with sp.score_hub(adata, hubs.hubs[(tissue, cell, hub_num)]) keeping defaults (binarize=True and importance weighting by hub-network gene connections), and ensure the AnnData expression is raw or normalized but NOT scaled. Aggregate per-cell scores to per-sample mean epithelial senescence scores (epithelial hubs span AT1, AT2, and airway populations) and flag per-sample outliers deviating from the cohort distribution. Run a two-sided Mann-Whitney U test on the per-sample mean scores comparing COVID-19 decedents vs controls. Rationale: per-cell scores are not independent, so the test must operate at the sample level; the two-tailed form detects senescence shifts in either direction. Record the U statistic and p-value alongside per-sample scores and outlier annotations.

## Related tools

- **SenePy** (Generates the per-cell senescence sp_score (via score_hub with binarize=True and hub-network importance weighting) that is aggregated per sample before the Mann-Whitney test) — https://github.com/jaleesr/senepy
- **AnnData** (Holds the single-cell expression matrix and stores scores in adata.obs['sp_score'] prior to per-sample aggregation)
- **scanpy** (AnnData-integrated preprocessing (normalize_total/log1p optional under binarized default scoring) of the scRNA-seq input)

## Examples

```
from scipy.stats import mannwhitneyu; u, p = mannwhitneyu(covid_sample_scores, control_sample_scores, alternative='two-sided')
```

## Evaluation signals

- Two-sided Mann-Whitney U test on per-sample mean epithelial senescence scores reproduces the reported increase in senescent AT1/AT2/airway cells in COVID-19 decedents at approximately p = 0.004
- sp_score was computed with score_hub defaults (binarize=True, importance weighting by hub-network gene connections), not the scanpy.score_genes-mimicking mode (binarize=False, importance=False)
- Expression values fed to scoring are raw or normalized but not scaled
- Per-sample outlier samples deviating from the cohort distribution are flagged and their effect on the p-value is documented
- Output CSV contains per-sample scores, outlier annotations, and the recorded Mann-Whitney test result

## Limitations

- Default score_hub binarizes the data, making normalization/transformation optional — p-values are only comparable if the scoring mode is fixed (binarized+importance vs scanpy-mimicking mode give different scores)
- Per-sample outliers deviating from the cohort distribution can drive the group comparison and should be flagged before testing
- SenePy signatures are cell-type-specific; a merged/universal hub must be created via merge_hubs (e.g., overlap_threshold or calculate_thresh) if a pan-epithelial or universal score is desired, and merged hubs lack per-gene importance weights unless taken from merge_hubs

## Evidence

- [methods] score_hub scoring defaults: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] score_hub importance weighting: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network."
- [readme] unscaled input requirement: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
- [results] COVID-19 epithelial senescence result: [evidence span withheld — anchor status: not_found]
- [readme] score stored in obs: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
