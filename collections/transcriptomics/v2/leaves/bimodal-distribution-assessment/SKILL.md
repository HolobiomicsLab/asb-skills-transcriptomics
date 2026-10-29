---
name: bimodal-distribution-assessment
description: Use when you need to validate that the top edges (e.g. top 150 filtered
  edges) of a BoostDiff differential network show condition-specific correlation patterns
  in the original expression matrix, i.e. bimodal vs unimodal KDE distributions across
  target conditions.
when_to_use_negative:
- The network has not yet been filtered for target genes with a more predictive model
  for the target condition — run filter_network (p=3, n_top_edges=150) first, otherwise
  the top-edge set is undefined
- The expression matrices have fewer than ~30 samples per condition, which the BoostDiff
  README recommends as the minimum per condition for reliable inference and downstream
  checks
- The goal is differential expression analysis rather than assessing condition-specific
  regulator-target correlation patterns of differential network edges
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BoostDiff
  role: Provides the filtering_visualization postprocessing module whose get_correlations
    and plot_corr_dists functions compute per-edge Pearson correlations and plot per-target-condition
    correlation distributions
  repo: github.com/scibiome/boostdiff_inference
- name: seaborn
  role: Used by plot_corr_dists to render KDE violin plots of the correlation distributions
    with bw_adjust=0.07
- name: pandas
  role: Loads the edge lists and expression matrices and saves the per-edge correlation
    dataframes as a CSV table
- name: matplotlib
  role: Backend for saving the KDE violin-plot figure as SVG/PDF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:47bb5cc69e31dcc3e7b40ac06600a62526fb82bdc2c4f943eff09939a7448078
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoostDiff
  - Python
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# bimodal-distribution-assessment

## Summary

Validates that top-ranked edges of a BoostDiff differential network are target-condition-specific by computing Pearson correlations of the original expression data per target condition and inspecting the correlation distributions: a bimodal distribution in the target condition (strong positive = activating edges, negative = inhibitory edges) versus a unimodal distribution in the non-target condition.

## When to use

Use when you need to validate that the top edges (e.g. top 150 filtered edges) of a BoostDiff differential network show condition-specific correlation patterns in the original expression matrix, i.e. bimodal vs unimodal KDE distributions across target conditions.

## When NOT to use

- The network has not yet been filtered for target genes with a more predictive model for the target condition — run filter_network (p=3, n_top_edges=150) first, otherwise the top-edge set is undefined
- The expression matrices have fewer than ~30 samples per condition, which the BoostDiff README recommends as the minimum per condition for reliable inference and downstream checks
- The goal is differential expression analysis rather than assessing condition-specific regulator-target correlation patterns of differential network edges

## Inputs

- Filtered BoostDiff edge lists for disease-target and control-target sub-analyses (chained from filter_network, p=3, n_top_edges=150)
- Preprocessed disease expression matrix (tab-separated, rows=genes, columns=samples, first column named 'Gene', e.g. GEO GSE156063 COVID-19)
- Preprocessed control expression matrix with the same gene set (tab-separated, first column 'Gene')

## Outputs

- CSV table of per-edge Pearson correlation values for the disease-target, control-target, and random-edge sets
- KDE violin-plot figure of per-condition correlation distributions (e.g. corr_dists_covid19.svg/PDF)

## How to apply

Load the filtered BoostDiff edge lists (from filter_network with p=3, n_top_edges=150) for the disease-target and control-target sub-analyses together with the preprocessed disease and control expression matrices (e.g. GEO GSE156063 COVID-19, tab-separated with genes as rows and samples as columns). Call get_correlations with percentile=3, n_edges=150 and index='Gene' to compute Pearson correlations of the original expression data for the top edges plus randomly selected edges, returning separate dataframes for the disease-target, control-target and random-edge sets. Save these per-edge correlation dataframes as a CSV table, then plot per-condition correlation distributions as KDE violin plots with plot_corr_dists using bw_adjust=0.07. The correct outcome is a bimodal correlation distribution for edges in the target condition — strong positive values indicating target-condition-specific activating regulator-target relationships and negative values indicating inhibitory relationships — versus a unimodal distribution in the non-target condition and for the random-edge set.

## Related tools

- **BoostDiff** (Provides the filtering_visualization postprocessing module whose get_correlations and plot_corr_dists functions compute per-edge Pearson correlations and plot per-target-condition correlation distributions) — github.com/scibiome/boostdiff_inference
- **seaborn** (Used by plot_corr_dists to render KDE violin plots of the correlation distributions with bw_adjust=0.07)
- **pandas** (Loads the edge lists and expression matrices and saves the per-edge correlation dataframes as a CSV table)
- **matplotlib** (Backend for saving the KDE violin-plot figure as SVG/PDF)

## Examples

```
df_dis, df_con, df_rand = get_correlations(file_dis_expr, file_con_expr, output_folder, percentile=3, n_edges=150, index='Gene'); plot_corr_dists(df_dis, df_con, df_rand, title=title, filename='corr_dists_covid19', bw_adjust=0.07)
```

## Evaluation signals

- The correlation CSV table exists and contains per-edge correlation fields for the disease-target, control-target, and random-edge sets
- The distribution figure (e.g. corr_dists_covid19.svg/PDF) is generated and visually consistent with the bimodal-target / unimodal-non-target pattern
- Target-condition correlations separate into strong positive values (activating edges) and negative values (inhibitory edges), producing the bimodal shape
- Randomly selected edges do not show the bimodal separation, confirming the pattern is specific to the top-ranked edges

## Limitations

- The bw_adjust=0.07 KDE bandwidth is intentionally small; over-smoothing would mask the bimodal structure
- Results depend on the preceding filter_network settings (p=3, n_top_edges=150); different percentiles or edge counts change the edge set being assessed
- BoostDiff was tested on Linux, and large datasets (e.g. the Crohn's disease dataset) require a high-performance computing cluster

## Evidence

- [methods] get_correlations call computing per-edge correlations: "df_dis, df_con, df_rand = get_correlations(file_dis_expr, file_con_expr, output_folder,"
- [methods] plot_corr_dists call with bandwidth setting: "plot_corr_dists(df_dis, df_con, df_rand, title=title, filename=file_grn, bw_adjust=0.07)"
- [methods] Purpose of the correlation distribution plots: "Plot correlation distributions per target condition"
- [readme] Input matrix format required for correlation computation: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples."
