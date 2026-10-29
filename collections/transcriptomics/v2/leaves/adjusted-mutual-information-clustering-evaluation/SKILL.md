---
name: adjusted-mutual-information-clustering-evaluation
description: Use when comparing clustering solutions (e.g., eeSNV- vs gene-expression-derived
  partitions of single cells) against known subpopulation labels and you need a chance-corrected
  agreement score over multiple bootstrap runs.
when_to_use_negative:
- No ground-truth subpopulation labels exist for the dataset — AMI requires reference
  labels and cannot be computed
- Input is a single clustering result without repeated runs or known labels, where
  internal validation indices (e.g., silhouette) would be more appropriate
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Provides the clustering algorithms (K-means, agglomerative WARD, PCA, factor
    analysis) and the adjusted mutual information metric used to score partitions
    against known labels
  repo: http://scikit-learn.org/
- name: SSrGE
  role: Produces the eeSNV-reduced feature matrices (ssrge.fit / ssrge.transform /
    ssrge.rank_eeSNVs) that are clustered and then scored by AMI against GE baselines
  repo: https://github.com/lanagarmire/SSrGE.git
- name: SIMLR
  role: One of the five clustering strategies whose outputs are evaluated by AMI across
    the 30 bootstrap runs
- name: Python 2.7
  role: Runtime environment (with Numpy, Scipy, scikit-learn 0.18, tabulate) in which
    the clustering and AMI aggregation pipeline executes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:0999b42a439627723a98a929c090c8e73ce683538e7d9780aee2282eac6ecf7d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Numpy
  - Python
  - Python 2.7
  - SIMLR
  - SSrGE
  - Scipy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# adjusted-mutual-information-clustering-evaluation

## Summary

Evaluates clustering quality by computing adjusted mutual information (AMI) between inferred cell cluster labels and known subpopulation labels, repeated over bootstrap runs (30 per dataset/feature type) to obtain mean and variability. Used in the eeSNV vs GE comparison to score subpopulation retrieval across scRNA-seq datasets.

## When to use

Use when comparing clustering solutions (e.g., eeSNV- vs gene-expression-derived partitions of single cells) against known subpopulation labels and you need a chance-corrected agreement score over multiple bootstrap runs.

## When NOT to use

- No ground-truth subpopulation labels exist for the dataset — AMI requires reference labels and cannot be computed
- Input is a single clustering result without repeated runs or known labels, where internal validation indices (e.g., silhouette) would be more appropriate

## Inputs

- Known subpopulation labels for each cell in a scRNA-seq dataset
- Predicted cluster assignments from one of five clustering strategies (PCA/FA + K-means, PCA/FA + agglomerative WARD, SIMLR)
- eeSNV-reduced feature matrices from ssrge.transform(X) and full gene expression matrices (n_cells x n_Genes)
- Bootstrap resamples of the cell-by-feature matrices (30 runs per dataset per feature type)

## Outputs

- Per-run AMI scores against known subpopulation labels, bounded in [0, 1]
- Aggregated mean and variability of AMI per dataset, feature type (eeSNV vs GE), and clustering strategy
- Full comparison table written to ami_comparison_results.csv

## How to apply

For each dataset, transform cells into eeSNV feature space via ssrge.transform(X) or use the full gene expression (GE) matrix, then run one of five clustering strategies (PCA or factor-analysis followed by K-means or agglomerative WARD linkage, or SIMLR). Repeat each clustering over 30 bootstrap resampling runs per dataset per feature type, computing adjusted mutual information against the known subpopulation labels at every run. AMI scores are bounded in [0, 1]; aggregate the mean and variability of AMI per dataset, feature type, and clustering strategy into a comparison table (e.g., ami_comparison_results.csv). Correct application reproduces the expected pattern in which eeSNV features achieve higher accuracy than GE for identifying subpopulations in all datasets tested, independent of the clustering method.

## Related tools

- **scikit-learn** (Provides the clustering algorithms (K-means, agglomerative WARD, PCA, factor analysis) and the adjusted mutual information metric used to score partitions against known labels) — http://scikit-learn.org/
- **SSrGE** (Produces the eeSNV-reduced feature matrices (ssrge.fit / ssrge.transform / ssrge.rank_eeSNVs) that are clustered and then scored by AMI against GE baselines) — https://github.com/lanagarmire/SSrGE.git
- **SIMLR** (One of the five clustering strategies whose outputs are evaluated by AMI across the 30 bootstrap runs)
- **Python 2.7** (Runtime environment (with Numpy, Scipy, scikit-learn 0.18, tabulate) in which the clustering and AMI aggregation pipeline executes)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(labels_true, labels_pred)
```

## Evaluation signals

- All AMI values fall within the valid range [0, 1]
- Aggregated AMI reproduces the Fig. 2 pattern: eeSNV features achieve higher accuracy than GE in all five datasets (GSE73121, GSE51372, GSE67980, GSE57872, GSE75688), independent of clustering method
- Each dataset/feature-type/strategy cell of the comparison table contains 30 bootstrap runs (bootstrap variability reported)
- The full comparison table is emitted to ami_comparison_results.csv with rows keyed by dataset, feature type, and clustering strategy

## Limitations

- AMI against known labels only measures agreement with existing subpopulation annotations; it cannot validate clusters when reference labels are noisy or unavailable
- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads, which can depress eeSNV-based AMI
- LASSO may arbitrarily select one of highly correlated SNV features corresponding to GE, affecting the eeSNV features being scored

## Evidence

- [abstract] eeSNV superiority over GE across clustering methods: "In all the datasets tested, eeSNVs achieve better accuracies than gene expression for identifying subpopulations."
- [methods] SSrGE usage as Python package producing features to cluster: "SSrGE should be used as a python package, below are usage example."
- [readme] SSrGE pipeline components and library basis: "This procedure can be used as  a dimension reduction/feature selection procedure or as a feature ranking. It is based on the Scikit-Learn library"
- [abstract] Noise motivation for subpopulation characterization: "characterization of subpopulations with transcript abundance is subject to a significant amount of noise"
