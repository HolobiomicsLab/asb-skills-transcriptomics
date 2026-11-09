---
name: covid19-transcriptomics-differential-network-analysis
description: Use when you have BoostDiff edge lists from both target-condition sub-analyses
  (e.
when_to_use_negative:
- You only have one BoostDiff run (single target condition) — the bimodal-vs-unimodal
  comparison requires both disease-target and control-target sub-analyses plus the
  random-edge baseline
- You have raw unfiltered BoostDiff network output — correlations are meant for the
  top-ranked edges after filter_network (p=3, n_top_edges=150), not the full raw differential
  network
- You want to infer the differential network itself or tune BoostDiff parameters (n_estimators,
  n_features, n_subsamples) — this skill is a postprocessing/validation step operating
  on already inferred networks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BoostDiff
  role: Source of the differential-network edge lists whose top edges are analyzed;
    its filtering_visualization postprocessing module provides get_correlations and
    plot_corr_dists
  repo: https://github.com/scibiome/boostdiff_inference
- name: Python
  role: Execution environment for the postprocessing script (boostdiff.main_boostdiff
    import, pandas/numpy I/O)
- name: pandas
  role: Loads filtered edge lists and preprocessed expression matrices; writes the
    per-edge correlation CSV
- name: seaborn
  role: Renders the KDE violin correlation-distribution plots via plot_corr_dists
    with bw_adjust=0.07
- name: matplotlib
  role: Plotting backend for saving the correlation-distribution figures as SVG/PDF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:fc888d8961b46837c177b4d11f344c51fdeee0e0348948c53589485a2f6125e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - Python
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# covid19-transcriptomics-differential-network-analysis

## Summary

Computes Pearson correlations on the original disease/control expression data for top-ranked BoostDiff differential-network edges per target condition (disease-target vs control-target) and compares their per-condition correlation distributions. It validates that top edges show condition-specific regulatory behavior, e.g. a bimodal correlation distribution in the target condition vs a unimodal one in the non-target condition.

## When to use

Use when you have BoostDiff edge lists from both target-condition sub-analyses (e.g. COVID-19 GSE156063 disease-target and control-target runs) and preprocessed expression matrices, and you want to characterize whether top edges encode target-condition-specific activating vs inhibitory regulator-target relationships.

## When NOT to use

- You only have one BoostDiff run (single target condition) — the bimodal-vs-unimodal comparison requires both disease-target and control-target sub-analyses plus the random-edge baseline
- You have raw unfiltered BoostDiff network output — correlations are meant for the top-ranked edges after filter_network (p=3, n_top_edges=150), not the full raw differential network
- You want to infer the differential network itself or tune BoostDiff parameters (n_estimators, n_features, n_subsamples) — this skill is a postprocessing/validation step operating on already inferred networks

## Inputs

- Filtered BoostDiff edge list for the disease-target sub-analysis (top ~150 edges, percentile=3 filter)
- Filtered BoostDiff edge list for the control-target sub-analysis
- Preprocessed disease expression matrix (tab-separated, rows=genes, first column 'Gene') from GEO GSE156063
- Preprocessed control expression matrix (same format, same feature set as disease matrix)

## Outputs

- CSV table of per-edge Pearson correlation values for disease-target, control-target, and random-edge sets
- KDE violin plot of per-condition correlation distributions (corr_dists_covid19.svg/PDF, bw_adjust=0.07)

## How to apply

Load the filtered COVID-19 BoostDiff edge lists from the disease-target and control-target sub-analyses (chained from the p=3, n_top_edges=150 filter_network step) together with the preprocessed GSE156063 disease/control expression matrices (tab-separated, rows=genes, first column named 'Gene') using pandas. Call get_correlations with percentile=3, n_edges=150, index='Gene' on the original (not normalized) expression data to compute Pearson correlations for the top edges plus a randomly selected edge set, returning three dataframes (df_dis, df_con, df_rand). Save the per-edge correlation dataframes as a CSV table, then render the per-condition correlation distributions as KDE violin plots with plot_corr_dists (seaborn, bw_adjust=0.07, i.e. very narrow bandwidth so bimodality is not smoothed away), saving as corr_dists_covid19.svg/PDF. Verify the expected pattern: strong positive correlations in the target condition indicate activating regulator-target relationships, negative values indicate target-condition-specific inhibitory relationships, and the non-target condition shows a unimodal distribution.

## Related tools

- **BoostDiff** (Source of the differential-network edge lists whose top edges are analyzed; its filtering_visualization postprocessing module provides get_correlations and plot_corr_dists) — https://github.com/scibiome/boostdiff_inference
- **Python** (Execution environment for the postprocessing script (boostdiff.main_boostdiff import, pandas/numpy I/O))
- **pandas** (Loads filtered edge lists and preprocessed expression matrices; writes the per-edge correlation CSV)
- **seaborn** (Renders the KDE violin correlation-distribution plots via plot_corr_dists with bw_adjust=0.07)
- **matplotlib** (Plotting backend for saving the correlation-distribution figures as SVG/PDF)

## Examples

```
df_dis, df_con, df_rand = get_correlations(file_dis_expr, file_con_expr, output_folder, percentile=3, n_edges=150, index='Gene'); plot_corr_dists(df_dis, df_con, df_rand, bw_adjust=0.07)
```

## Evaluation signals

- The correlation table CSV exists and contains per-edge Pearson correlation fields for the disease-target, control-target, and random-edge sets
- The KDE violin plot figure (corr_dists_covid19.svg/PDF) is generated and visually consistent with a bimodal distribution in the target condition
- The non-target condition shows a unimodal correlation distribution, as reported for the COVID-19 dataset
- Strong positive correlations among target-condition top edges mark activating regulator-target relationships, negative values mark inhibitory ones
- Parameters match the protocol: percentile=3, n_edges=150, index='Gene' for get_correlations; bw_adjust=0.07 for plot_corr_dists

## Limitations

- The KDE shape is highly sensitive to bandwidth — the very narrow bw_adjust=0.07 is needed to preserve bimodality; a default bandwidth could wash out the reported pattern
- BoostDiff ideally receives at least 30 samples per condition; datasets like the Crohn's disease one (GSE126124) are too large for a typical machine and need an HPC cluster
- BoostDiff was tested on linux only, per the repository README

## Evidence

- [methods] get_correlations on filtered edge lists: "df_dis, df_con, df_rand = get_correlations(file_dis_expr, file_con_expr, output_folder,"
- [methods] plot_corr_dists bandwidth parameter: "plot_corr_dists(df_dis, df_con, df_rand, title=title, filename=file_grn, bw_adjust=0.07)"
- [methods] filter parameters for top edges: "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
- [readme] input format requirement for expression matrices: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples."
- [readme] recommended sample count per condition: "The ideal number of samples per condition as input to BoostDiff would be at least 30 samples each."
