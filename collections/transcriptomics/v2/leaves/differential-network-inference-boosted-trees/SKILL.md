---
name: differential-network-inference-boosted-trees
description: Use when you have two tab-separated gene expression matrices (same genes,
  rows=genes, columns=samples) from two biological conditions and need a condition-specific
  differential network for downstream pathway enrichment.
when_to_use_negative:
- The two input matrices do not share the same gene/feature set or the first column
  is not named 'Gene' — BoostDiff requires identical features across conditions
- Either condition has fewer than ~15 samples per condition (min_samples default)
  or well below the recommended ~30 samples per condition
- The goal is a static (single-condition) regulatory network rather than a differential
  network between two conditions
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: BoostDiff
  role: 'Core method: builds the modified AdaBoost ensemble of differential trees
    and infers the differential network from the two expression matrices'
  repo: https://github.com/scibiome/boostdiff_inference
- name: GEOquery
  role: Downloads the GSE126124 dataset from GEO in R and extracts sample metadata
    (tissue:ch1, disease type:ch1) for grouping
- name: biomaRt
  role: Maps Affy probe IDs (affy_hugene_1_0_st_v1) to HGNC gene symbols before network
    inference
- name: dplyr
  role: Aggregates probes per gene by mean and writes the disease/control expression
    tables
- name: python-louvain
  role: Louvain community detection to partition the filtered differential network
    into modules for enrichment analysis
  repo: https://github.com/taynaud/python-louvain
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:dd6e0bb22a5890e24a9186b090073cff8f5cfb218abc582926adc739d4bb32bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - biomaRt
  - conda
  - dplyr
  - pip
  - python-louvain
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# differential-network-inference-boosted-trees

## Summary

Infers a differential gene regulatory network between two conditions (e.g., Crohn's disease vs. control colon biopsies) using BoostDiff, a modified AdaBoost ensemble of differential regression trees with differential variance improvement (DVI) as the splitting criterion.

## When to use

Use when you have two tab-separated gene expression matrices (same genes, rows=genes, columns=samples) from two biological conditions and need a condition-specific differential network for downstream pathway enrichment.

## When NOT to use

- The two input matrices do not share the same gene/feature set or the first column is not named 'Gene' — BoostDiff requires identical features across conditions
- Either condition has fewer than ~15 samples per condition (min_samples default) or well below the recommended ~30 samples per condition
- The goal is a static (single-condition) regulatory network rather than a differential network between two conditions

## Inputs

- Disease-condition expression matrix as a tab-separated file (rows=genes, columns=samples, first column named 'Gene', e.g. df_crohns.txt)
- Control-condition expression matrix as a tab-separated file with the same feature set (e.g. df_ctrl_crohns.txt)

## Outputs

- differences_<keyword>.txt per run (difference in prediction error per target gene)
- boostdiff_network_<keyword>.txt per run (raw differential network edge list written into 'disease' and 'control' output subfolders)
- Filtered differential network edge list (filter_network at p=3 percentile)
- Louvain module assignments (CSV) and KEGG enrichment tables (CSV)

## How to apply

Preprocess the expression data (e.g., download GSE126124 via GEOquery, filter to 'tissue:ch1' == 'colon biopsy', split by 'disease type:ch1', map Affy probe IDs to HGNC symbols with biomaRt, aggregate probes per gene by mean with dplyr) and write df_crohns.txt and df_ctrl_crohns.txt as tab-separated files with a 'Gene' first column. Install BoostDiff in a conda environment ('conda env create --name bdenv --file environment.yml; pip install .') and run it twice — once with the disease condition as target and once with control as target — setting n_estimators=50 (recommended to avoid overfitting on real transcriptomics data), regulators='all', and normalize='unit_variance'. Filter each raw network (boostdiff_network_<keyword>.txt) for target genes whose model is more predictive in the target condition, starting at the p=3 percentile of the largest differences in prediction error (filter_network(..., p=3, n_top_edges=150)). Partition the filtered differential network into modules with Louvain community detection (python-louvain, imported as 'community') and perform KEGG over-representation analysis per module to recover disease-relevant pathways such as TNF signaling, NOD-like receptor signaling, and Th17 cell differentiation.

## Related tools

- **BoostDiff** (Core method: builds the modified AdaBoost ensemble of differential trees and infers the differential network from the two expression matrices) — https://github.com/scibiome/boostdiff_inference
- **GEOquery** (Downloads the GSE126124 dataset from GEO in R and extracts sample metadata (tissue:ch1, disease type:ch1) for grouping)
- **biomaRt** (Maps Affy probe IDs (affy_hugene_1_0_st_v1) to HGNC gene symbols before network inference)
- **dplyr** (Aggregates probes per gene by mean and writes the disease/control expression tables)
- **python-louvain** (Louvain community detection to partition the filtered differential network into modules for enrichment analysis) — https://github.com/taynaud/python-louvain

## Examples

```
model.run(file_disease='df_crohns.txt', file_control='df_ctrl_crohns.txt', output_folder='../outputs_crohns', n_estimators=50, n_features=2500, n_subsamples=20, keyword='crohnsvsctrl', normalize=True, n_processes=4)
```

## Evaluation signals

- The KEGG enrichment tables contain significant terms matching disease-relevant immune signaling pathways (TNF signaling, NOD-like receptor signaling, Th17 cell differentiation, IL-17 signaling, NF-κB signaling, inflammatory bowel disease)
- Each boostdiff_network_<keyword>.txt is a non-empty tab-separated table and the filtered edge list is populated after the p=3 percentile filter
- Output files differences_<keyword>.txt and boostdiff_network_<keyword>.txt are written into both the 'disease' and 'control' output subfolders (two-sided run)
- Input matrices are tab-separated with a 'Gene' first column and identical feature sets across disease and control files
- The Crohn's disease network (37 CD vs 19 healthy controls, colon biopsies) yields modules with significant KEGG over-representation

## Limitations

- The Crohn's disease dataset is too large for the demo and requires a high-performance computing cluster to run
- BoostDiff was tested on Linux only
- For real transcriptomics datasets, a relatively low number of base learners (n_estimators=50) is recommended to avoid overfitting
- Raw networks must be filtered for target genes where BoostDiff found a more predictive model for the target condition (p=3 percentile) before interpretation

## Evidence

- [methods] BoostDiff is a tree-based method for inferring differential networks from large-scale transcriptomics data: "BoostDiff is a tree-based method for inferring differential networks from large-scale transcriptomics data"
- [abstract] boostdiff abstract: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition. To build the differential trees, we propose differential variance improvement as a"
- [readme] readme installation: "conda env create --name bdenv --file environment.yml
conda activate bdenv

pip install ."
- [readme] readme parameter recommendation: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting"
- [readme] readme input format: "Each input is a tab-separated file. The rows correspond to genes, while the columns correspond to samples. Note that the two inputs should have the same set of features, where the first column should"
- [readme] readme HPC limitation: "the Crohn's disease dataset is too large and will need a high-performance computing cluster to run"
