---
name: correlation-distribution-visualization
description: 'Use when validating filtered BoostDiff differential networks: you have
  top edges (e.g. percentile=3, n_edges=150) plus disease/control expression matrices
  and need to check whether edge correlations are target-condition-specific (bimodal
  vs unimodal distributions).'
when_to_use_negative:
- You have not yet run BoostDiff and filtered the raw network for target genes — there
  are no top edges to correlate
- Expression matrices for disease and control lack a shared gene feature set or the
  'Gene' first column required by BoostDiff inputs
- The goal is testing edge significance rather than post-hoc validation of correlation
  patterns in the original data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: BoostDiff
  role: Source of the differential network edges and the filtering_visualization module
    providing get_correlations and plot_corr_dists
  repo: https://github.com/scibiome/boostdiff_inference
- name: pandas
  role: Loading filtered edge lists and expression matrices; saving the per-edge correlation
    dataframes as a CSV table
- name: seaborn
  role: KDE violin-plot rendering of per-condition correlation distributions via plot_corr_dists
    (bw_adjust=0.07)
- name: matplotlib
  role: Plot backend for saving the correlation distribution figures as SVG/PDF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:9fb9cd75a0e0388b346f6fd79b6b046b80f282e00ac8018121ee60c23a6ab3f4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# correlation-distribution-visualization

## Summary

Computes Pearson correlations of the original expression data for top-ranked BoostDiff differential-network edges per target condition and visualizes their distributions as KDE violin plots, to validate that filtered edges reflect condition-specific regulator-target relationships.

## When to use

Use when validating filtered BoostDiff differential networks: you have top edges (e.g. percentile=3, n_edges=150) plus disease/control expression matrices and need to check whether edge correlations are target-condition-specific (bimodal vs unimodal distributions).

## When NOT to use

- You have not yet run BoostDiff and filtered the raw network for target genes — there are no top edges to correlate
- Expression matrices for disease and control lack a shared gene feature set or the 'Gene' first column required by BoostDiff inputs
- The goal is testing edge significance rather than post-hoc validation of correlation patterns in the original data

## Inputs

- Filtered BoostDiff edge lists for the disease-target and control-target sub-analyses (from filter_network with p=3, n_top_edges=150)
- Preprocessed disease expression matrix (tab-separated, rows=genes, columns=samples, first column named 'Gene'), e.g. COVID-19 GSE156063 disease data
- Preprocessed control expression matrix (same format and same gene features), e.g. GSE156063 control data

## Outputs

- CSV table of per-edge Pearson correlations for disease-target, control-target, and random-edge sets
- KDE violin plot of correlation distributions per target condition (e.g. corr_dists_covid19.svg/PDF)

## How to apply

After filtering the raw BoostDiff networks (e.g. filter_network with p=3, n_top_edges=150), load the disease-target and control-target edge lists plus the preprocessed disease and control expression matrices (rows=genes, columns=samples, first column 'Gene'). Call get_correlations from the filtering_visualization postprocessing module with percentile=3, n_edges=150, index='Gene' to compute Pearson correlations on the original expression data for the top edges plus a randomly selected edge set, yielding df_dis, df_con, and df_rand. Save these per-edge correlation dataframes as a CSV table, then call plot_corr_dists (seaborn, bw_adjust=0.07) to produce KDE violin plots of the per-condition correlation distributions. The expected pattern is a bimodal distribution in the target condition — strong positive correlations indicating activating regulator-target relationships, negative values indicating inhibitory ones — versus a unimodal distribution in the non-target condition and for random edges.

## Related tools

- **BoostDiff** (Source of the differential network edges and the filtering_visualization module providing get_correlations and plot_corr_dists) — https://github.com/scibiome/boostdiff_inference
- **pandas** (Loading filtered edge lists and expression matrices; saving the per-edge correlation dataframes as a CSV table)
- **seaborn** (KDE violin-plot rendering of per-condition correlation distributions via plot_corr_dists (bw_adjust=0.07))
- **matplotlib** (Plot backend for saving the correlation distribution figures as SVG/PDF)

## Examples

```
df_dis, df_con, df_rand = get_correlations(file_dis_expr, file_con_expr, output_folder, percentile=3, n_edges=150, index='Gene'); plot_corr_dists(df_dis, df_con, df_rand, bw_adjust=0.07)
```

## Evaluation signals

- The correlation table CSV exists and contains per-edge Pearson correlation fields for df_dis, df_con, and the random-edge set
- The distribution figure (corr_dists_covid19.svg/PDF) is generated and visually consistent with a bimodal target-condition distribution
- The non-target (control) condition shows a unimodal correlation distribution, contrasting with the target condition
- Strong positive correlations correspond to target-condition-specific activating regulator-target edges and negative values to inhibitory edges
- Random-edge correlations serve as a baseline and should not show the bimodal target-condition pattern

## Limitations

- Requires at least ~15 samples per condition (BoostDiff min_samples default) and ideally ≥30 per condition for reliable Pearson correlations
- The pattern was demonstrated on the COVID-19 dataset (GSE156063, ~15,000 genes, ~100 samples per condition); other datasets may need adaptation
- Bw_adjust=0.07 makes the KDE sensitive to sampling noise; correlation distributions are computed on the original expression data, not on the tree model itself

## Evidence

- [methods] get_correlations and plot_corr_dists in methods: "df_dis, df_con, df_rand = get_correlations(file_dis_expr, file_con_expr, output_folder,"
- [methods] plot_corr_dists parameters: "plot_corr_dists(df_dis, df_con, df_rand, title=title, filename=file_grn, bw_adjust=0.07)"
- [methods] filter_network parameters: "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
- [readme] input format from README: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples."
- [methods] dataset availability: "The COVID-19 and Crohn's disease datasets are available in the Gene Expression Omnibus (GEO) database with accession numbers GSE156063 and GSE126124, respectively."
