---
name: differential-network-inference-from-transcriptomics
description: Use when two tab-separated gene expression matrices (genes as rows, first
  column named 'Gene') from two conditions are available and the goal is a condition-specific
  differential regulatory network with enriched disease-relevant pathways.
when_to_use_negative:
- The inputs are not matched two-condition expression matrices with an identical gene
  set — BoostDiff simultaneously requires a disease and a control matrix with the
  same features
- The goal is single-condition network inference or differential expression alone;
  BoostDiff targets differential networks and is meant to complement standard differential
  expression analyses
- Sample size per condition is below the package minimum (min_samples=15 per condition);
  the README recommends at least 30 samples per condition as ideal input
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: 'Core method: builds the modified AdaBoost ensemble of differential trees
    and writes the raw differential network and differences files'
  repo: github.com/gihannagalindez/boostdiff_inference
- name: GEOquery
  role: R package used to download GEO datasets (e.g., GSE156063, GSE126124) and extract
    expression matrices via getGEO
- name: biomaRt
  role: Maps Affy probe IDs (affy_hugene_1_0_st_v1) to HGNC gene symbols before writing
    the expression matrices
- name: dplyr
  role: Aggregates multiple probes per gene by mean and cleans missing gene-symbol
    rows in the R preprocessing script
- name: python-louvain
  role: Louvain community detection on the filtered differential network; imported
    as 'community' to export module assignments
  repo: taynaud/python-louvain
- name: gseapy
  role: KEGG pathway enrichment (Enrichr) of detected network modules to validate
    disease-relevant pathway recovery
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:c1cfbeba69d845a748443cf51fc7224dcc318fd9edc691fca2d8255bc2de6a28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - biomaRt
  - conda
  - dplyr
  - gseapy
  - numpy
  - pandas
  - pip
  - python-louvain
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.68
n_claims: 25
---

# differential-network-inference-from-transcriptomics

## Summary

Infer a differential gene regulatory network discriminating two biological conditions (e.g., disease vs. control) by building modified AdaBoost ensembles of differential regression trees with differential variance improvement (DVI) as the splitting criterion, as implemented in BoostDiff.

## When to use

Use when two tab-separated gene expression matrices (genes as rows, first column named 'Gene') from two conditions are available and the goal is a condition-specific differential regulatory network with enriched disease-relevant pathways.

## When NOT to use

- The inputs are not matched two-condition expression matrices with an identical gene set — BoostDiff simultaneously requires a disease and a control matrix with the same features
- The goal is single-condition network inference or differential expression alone; BoostDiff targets differential networks and is meant to complement standard differential expression analyses
- Sample size per condition is below the package minimum (min_samples=15 per condition); the README recommends at least 30 samples per condition as ideal input

## Inputs

- Disease-condition gene expression matrix (tab-separated; rows = genes, columns = samples, first column named 'Gene')
- Control-condition gene expression matrix (same feature set as the disease matrix)
- Run parameters: n_estimators=50, n_features=2500, n_subsamples=20, min_samples_leaf=2, min_samples_split=6, max_depth=2, learning_rate=1.0, loss='exponential', regulators='all', unit_variance normalization, n_processes

## Outputs

- boostdiff_network_<keyword>.txt raw edge list written per target-condition run into 'disease' and 'control' subfolders
- differences_<keyword>.txt file of prediction-error differences between conditions
- Filtered differential edge list after filter_network at the p=3 percentile (top ~1000 TF-filtered edges per condition)
- Louvain module assignment CSV and KEGG pathway enrichment CSV per module/network

## How to apply

Clone the boostdiff_inference repo, create the conda environment (bdenv) from environment.yml, and pip install the package. Prepare disease and control expression matrices as tab-separated files with genes as rows, columns as samples, identical feature sets, and the first column named 'Gene' (use GEOquery/biomaRt/dplyr for GEO data, e.g. mapping Affy probes to HGNC symbols and aggregating probes per gene by mean). Run BoostDiff twice via main_boostdiff.BoostDiff — Run 1 with disease as target and Run 2 with control as target — writing outputs into 'disease' and 'control' subfolders with a shared keyword; for real transcriptomics data use n_estimators=50 to avoid overfitting, n_features=2500, regulators='all', unit_variance normalization, and loss='exponential'. Afterwards, filter each raw network (boostdiff_network_<keyword>.txt) for target genes where BoostDiff found a more predictive model for the target condition (e.g., filter_network with p=3, the 3rd percentile of the largest mean differences in prediction error), then detect modules with Louvain (python-louvain) and run KEGG enrichment (gseapy Enrichr) to validate disease relevance.

## Related tools

- **BoostDiff** (Core method: builds the modified AdaBoost ensemble of differential trees and writes the raw differential network and differences files) — github.com/gihannagalindez/boostdiff_inference
- **GEOquery** (R package used to download GEO datasets (e.g., GSE156063, GSE126124) and extract expression matrices via getGEO)
- **biomaRt** (Maps Affy probe IDs (affy_hugene_1_0_st_v1) to HGNC gene symbols before writing the expression matrices)
- **dplyr** (Aggregates multiple probes per gene by mean and cleans missing gene-symbol rows in the R preprocessing script)
- **python-louvain** (Louvain community detection on the filtered differential network; imported as 'community' to export module assignments) — taynaud/python-louvain
- **gseapy** (KEGG pathway enrichment (Enrichr) of detected network modules to validate disease-relevant pathway recovery)

## Examples

```
from boostdiff.main_boostdiff import BoostDiff; model.run(file_etha, file_ctrl, output_folder, n_estimators, n_features, n_subsamples, keyword=keyword, n_processes=n_processes)
```

## Evaluation signals

- BoostDiff completes both target-condition runs and writes differences_<keyword>.txt and boostdiff_network_<keyword>.txt into each of the 'disease' and 'control' subfolders
- Each run builds 50 boosted differential trees (n_estimators=50), the recommended ensemble size for real transcriptomics data to avoid overfitting
- The filtered edge list is a non-empty tab-separated table filtered at the p=3 percentile of the largest differences in prediction error
- KEGG enrichment tables contain the reported disease-relevant terms (e.g., 'Coronavirus disease', 'TNF signaling pathway', 'Th17 cell differentiation', 'NF-kappa B', 'NOD-like receptor' signaling for COVID-19)
- Both input matrices share the same feature set with a first column named 'Gene' and genes as rows

## Limitations

- Large datasets (e.g., the Crohn's disease dataset GSE126124) may require a high-performance computing cluster to run
- BoostDiff was tested on Linux only
- For real transcriptomics datasets, the ensemble size must stay relatively low (n_estimators=50) to avoid overfitting
- At least 15 samples per condition are required (min_samples=15); at least 30 per condition is ideal

## Evidence

- [abstract] BoostDiff abstract claim: "We develop a new ensemble method called BoostDiff (boosted differential regression trees) to infer a differential network discriminating between two conditions."
- [abstract] AdaBoost ensemble w.r.t. target condition: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition."
- [readme] Input format requirement: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples."
- [readme] Recommended ensemble size: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting."
- [methods] Network filtering rationale: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [methods] Default normalization: "default is "unit_variance" normalization"
