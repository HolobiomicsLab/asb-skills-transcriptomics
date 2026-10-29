---
name: differential-gene-regulatory-network-inference
description: Use when you have two gene expression matrices (tab-separated, rows =
  genes, columns = samples, first column named 'Gene') from two conditions and want
  a context-specific differential network beyond plain differential expression.
when_to_use_negative:
- The Crohn's disease dataset (GSE126124) is too large and needs a high-performance
  computing cluster — avoid running on very large cohorts without HPC resources
- Either condition has fewer than min_samples (default 15) samples per class — BoostDiff
  requires at least the minimum samples per condition
- You only need standard differential expression, not condition-specific network structure
  — BoostDiff is complementary to, not a replacement for, DE analysis
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: BoostDiff
  role: 'Core method: builds the modified AdaBoost ensemble of differential trees
    and outputs the raw differential network'
  repo: https://github.com/scibiome/boostdiff_inference
- name: GEOquery
  role: Downloads expression datasets (e.g. GSE27219, GSE126124, GSE156063) from GEO
    in R
- name: biomaRt
  role: Maps platform probe IDs to desired gene IDs/symbols before network inference
- name: dplyr
  role: Aggregates replicate probes by mapped gene (summarise_all('mean')) and prepares
    the input matrices
- name: pySMOTE
  role: Oversamples the minority condition (e.g. to 48 samples per condition) to balance
    classes before running BoostDiff
  repo: https://github.com/akazs/pySMOTE
- name: python-louvain
  role: Louvain community detection on the filtered differential edge list to identify
    network modules for enrichment analysis
  repo: https://github.com/taynaud/python-louvain
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:590be1930c59b773df4730050a28b76b3c6120c583ef9dc5946d27d7341877ec
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
  - matplotlib
  - networkx
  - numpy
  - pandas
  - pip
  - pySMOTE
  - pygraphviz
  - python-louvain
  - seaborn
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5666666666666667
n_claims: 30
---

# differential-gene-regulatory-network-inference

## Summary

Infer a differential gene regulatory network discriminating two conditions (e.g. disease vs. control) with BoostDiff, an AdaBoost ensemble of differential regression trees using differential variance improvement (DVI) as splitting criterion, then filter the raw network for target-condition-specific edges.

## When to use

Use when you have two gene expression matrices (tab-separated, rows = genes, columns = samples, first column named 'Gene') from two conditions and want a context-specific differential network beyond plain differential expression.

## When NOT to use

- The Crohn's disease dataset (GSE126124) is too large and needs a high-performance computing cluster — avoid running on very large cohorts without HPC resources
- Either condition has fewer than min_samples (default 15) samples per class — BoostDiff requires at least the minimum samples per condition
- You only need standard differential expression, not condition-specific network structure — BoostDiff is complementary to, not a replacement for, DE analysis

## Inputs

- Tab-separated disease-condition expression matrix (rows = genes, columns = samples, first column named 'Gene')
- Tab-separated control-condition expression matrix with the same feature set
- Optional TF/regulator gene list (or the string 'all')
- Optional: minimum 30 samples per condition recommended; SMOTE oversampling for balanced classes

## Outputs

- boostdiff_network_<keyword>.txt raw differential network per run (disease-target and control-target)
- Filtered differential edge list (e.g. brca_diff_edges.tsv) after filter_network with p=3
- Hub gene tables per condition (CSV)
- Louvain module assignments per gene (e.g. brca_modules.csv) and KEGG over-representation enrichment table (e.g. brca_kegg_enrichment.csv)

## How to apply

Preprocess data (GEO via GEOquery, map IDs with biomaRt, aggregate replicates with dplyr) into two tab-separated matrices with 'Gene' as first column and identical feature sets; optionally filter by variance cutoff (e.g. 0.4) and oversample imbalanced classes with SMOTE to ≥30 samples per condition. Run model.run twice per context — once with disease as target and once with control as target — using n_estimators=50 to avoid overfitting, n_features=2500, n_subsamples=20, loss='exponential', regulators='all', and default 'unit_variance' normalization. Filter each raw network (boostdiff_network_<keyword>.txt) with filter_network(p=3, n_top_edges=150 per sub-analysis), keeping target genes for which BoostDiff found a more predictive model for the target condition. Compute hub genes or detect Louvain modules on the filtered edges, then validate biologically (e.g. SigB regulon membership for B. subtilis stress, or KEGG enrichment of PPAR signaling / insulin resistance / AMPK / adipocytokine pathways in breast cancer modules).

## Related tools

- **BoostDiff** (Core method: builds the modified AdaBoost ensemble of differential trees and outputs the raw differential network) — https://github.com/scibiome/boostdiff_inference
- **GEOquery** (Downloads expression datasets (e.g. GSE27219, GSE126124, GSE156063) from GEO in R)
- **biomaRt** (Maps platform probe IDs to desired gene IDs/symbols before network inference)
- **dplyr** (Aggregates replicate probes by mapped gene (summarise_all('mean')) and prepares the input matrices)
- **pySMOTE** (Oversamples the minority condition (e.g. to 48 samples per condition) to balance classes before running BoostDiff) — https://github.com/akazs/pySMOTE
- **python-louvain** (Louvain community detection on the filtered differential edge list to identify network modules for enrichment analysis) — https://github.com/taynaud/python-louvain

## Examples

```
model.run(file_etha, file_ctrl, output_folder, n_estimators=50, n_features=2500, n_subsamples=20, keyword='saltvssmm', loss='exponential', regulators='all', normalize='unit_variance')
```

## Evaluation signals

- Each input file is tab-separated with rows = genes, columns = samples, and the first column is named 'Gene'; both matrices share the same feature set
- Runs are performed twice per context (disease-target and control-target), and each raw network is filtered with filter_network(p=3, n_top_edges=150) keeping only target genes with a more predictive model for the target condition
- For GSE27219 stress contexts, the top-ten hub genes (e.g. csbD, ohrB, yuzA, yycD for salt/heat) are members of the SigB regulon
- For TCGA BRCA, the enrichment table reproduces 'PPAR signaling pathway', 'Insulin resistance', 'AMPK signaling pathway', and 'Adipocytokine signaling pathway' terms, and the edge list contains PPARG–PLIN4, PPARG–PCK1 edges and the CD36 hub
- n_estimators is kept relatively low (50) to avoid overfitting, and each condition has at least 30 samples (or ≥min_samples=15) after preprocessing

## Limitations

- BoostDiff was tested on Linux only
- Large datasets like the Crohn's disease cohort (GSE126124) require a high-performance computing cluster
- Ideal sample size is at least 30 per condition; below min_samples (default 15) per condition, trees cannot be built reliably
- n_estimators should stay relatively low (e.g. 50) on real transcriptomics data to avoid overfitting

## Evidence

- [abstract] BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition. To build the differential trees, we propose differential variance improvement as a: "BoostDiff builds an adaptively boosted (AdaBoost) ensemble of differential trees with respect to a target condition. To build the differential trees, we propose differential variance improvement as a"
- [methods] the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition: "the raw network should be filtered for target genes in which BoostDiff found a more predictive model for the target condition"
- [readme] for real transcriptomics datasets, we recommend to set a relatively low number of base learners, e.g. set n_estimators=50: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting."
- [abstract] BoostDiff identifies context-specific networks that are enriched with genes of known disease-relevant pathways and complements standard differential expression analyses.: "BoostDiff identifies context-specific networks that are enriched with genes of known disease-relevant pathways and complements standard differential expression analyses."
