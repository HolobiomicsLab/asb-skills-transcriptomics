---
name: network-edge-filtering
description: Use when a raw BoostDiff output network (boostdiff_network_<keyword>.txt)
  must be reduced to condition-specific edges before Louvain module detection, KEGG
  enrichment, or network visualization.
when_to_use_negative:
- The input is a differential-expression table rather than a BoostDiff raw network
  — filtering by prediction-error percentile does not apply
- Only one condition's network was generated (single BoostDiff run); the differential
  filtering presumes both disease-target and control-target runs
- The goal is a full, unfiltered association network rather than a condition-specific
  differential edge set
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: Provides the filtering_visualization postprocessing module (filter_network,
    plot_histogram, plot_grn, get_correlations) used to filter and inspect the raw
    differential network
  repo: https://github.com/scibiome/boostdiff_inference
- name: python-louvain
  role: Consumes the filtered edge list for Louvain community detection on the differential
    network
  repo: https://github.com/taynaud/python-louvain
provenance:
  source_task_ids:
  - task_008
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:5b48871ded20134ed88df479cb529ac3909a63a5d1090e0ac5a379d711670416
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - conda
  - gseapy
  - matplotlib
  - networkx
  - numpy
  - pandas
  - pip
  - pygraphviz
  - python-louvain
  - seaborn
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# network-edge-filtering

## Summary

Post-processing step of the BoostDiff differential-network workflow: filter the raw boosted differential tree network down to target genes whose models are more predictive in the target condition, then retain a top-ranked, transcription-factor-filtered edge list for downstream module detection and enrichment.

## When to use

Use when a raw BoostDiff output network (boostdiff_network_<keyword>.txt) must be reduced to condition-specific edges before Louvain module detection, KEGG enrichment, or network visualization.

## When NOT to use

- The input is a differential-expression table rather than a BoostDiff raw network — filtering by prediction-error percentile does not apply
- Only one condition's network was generated (single BoostDiff run); the differential filtering presumes both disease-target and control-target runs
- The goal is a full, unfiltered association network rather than a condition-specific differential edge set

## Inputs

- Raw BoostDiff network file (boostdiff_network_<keyword>.txt) per target condition
- Per-target-condition prediction-error output from the BoostDiff run (output_folder from model.run)
- Optional disease/control expression matrices (tab-separated, first column 'Gene') for downstream get_correlations checks

## Outputs

- Filtered differential edge list per condition (e.g. brca_diff_edges.tsv, top ~1000 TF-filtered edges)
- Per-edge correlation dataframe for target/non-target/random edges (when chained with get_correlations)
- Filtered networks feeding Louvain module CSVs and KEGG enrichment tables

## How to apply

After running BoostDiff twice (disease as target, control as target with loss='exponential', n_estimators=50, n_features=2500, unit_variance normalization), load each raw network file (boostdiff_network_<keyword>.txt) and call the filtering_visualization postprocessing function filter_network with p=3 — i.e. the 3rd percentile of the largest mean differences in prediction error between conditions — so only target genes with a more predictive model in the target condition are kept. Retain roughly the top 1000 transcription-factor-filtered edges per condition as the differential edge list (e.g. brca_diff_edges.tsv). The same filter step feeds both the network visualization (plot_grn, plot_histogram of mean difference in prediction error) and downstream analyses such as Louvain community detection and KEGG over-representation. Optionally pass n_top_edges (e.g. 150) when preparing edge sets for correlation-distribution checks with get_correlations/plot_corr_dists. Rationale: the raw ensemble network contains all variable-importance edges, but only target-gene edges with lower prediction error in the target condition represent genuine condition-specific regulation.

## Related tools

- **BoostDiff** (Provides the filtering_visualization postprocessing module (filter_network, plot_histogram, plot_grn, get_correlations) used to filter and inspect the raw differential network) — https://github.com/scibiome/boostdiff_inference
- **python-louvain** (Consumes the filtered edge list for Louvain community detection on the differential network) — https://github.com/taynaud/python-louvain

## Examples

```
from boostdiff.filtering_visualization import filter_network; df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)
```

## Evaluation signals

- filter_network is called with p=3 (3rd percentile of largest mean differences in prediction error between conditions)
- The retained differential edge list contains roughly the top 1000 transcription-factor-filtered edges per target condition
- For BRCA, the filtered edge list contains the reported PPARG–PLIN4 and PPARG–PCK1 edges and the CD36 hub
- Top edges from the target condition show a bimodal Pearson correlation distribution vs a unimodal distribution in the non-target condition (plot_corr_dists, bw_adjust=0.07)
- Filtering precedes Louvain module detection and KEGG enrichment that reproduce the reported disease-relevant terms (e.g. PPAR signaling, TNF signaling)

## Limitations

- BoostDiff was tested on Linux, and large datasets (e.g. the Crohn's disease GSE126124 dataset) require a high-performance computing cluster before filtering is possible
- The raw network must be filtered before interpretation because it contains all variable-importance edges, not only target-condition-specific ones
- The p=3 percentile and top-edge cutoffs are dataset-dependent choices; validation of edge content (e.g. known regulator–target edges) is recommended per dataset

## Evidence

- [methods] Filter the raw network for target genes: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [methods] filter_network call: "df_dis = filter_network(file_net_dis, file_diff_dis, p=3, n_top_edges=150)"
- [other] Louvain on filtered edges: [evidence span withheld — anchor status: not_found]
- [readme] DVI criterion rationale: "The network is inferred by building modified AdaBoost ensembles of differential trees as base learners. BoostDiff modifies regression trees to use differential variance improvement (DVI) as the novel"
