---
name: random-forest-gradient-boosting-classification
description: Use when classifying single-cell RNA-seq cells from CKG or gene expression
  feature matrices with supervised tree-based models, or when RF/GBM feature_importances_
  rankings are needed to compare feature representations.
when_to_use_negative:
- Input is raw reads or jellyfish k-mer files (BAM/FASTA) — the k-mer matrix, CKG
  generation, and cell matrix generation modules must run upstream before classification.
- The goal is CKG generation or k-mer compression itself (simhash fingerprinting);
  this skill starts from existing CKG/GE feature matrices with labels.
- No cell type labels are available — RF and GBM here are supervised classifiers that
  require the provided cell labels as targets.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn (sklearn)
  role: Supplies the RandomForest and GradientBoosting classifiers fitted on CKG and
    GE matrices; feature_importances_ provides the top-k feature rankings
- name: pandas
  role: Loads the gene expression matrix with cell labels and the CKG cell matrix;
    writes shared_genes.csv overlap output
- name: numpy
  role: Array operations for sorting feature_importances_ and slicing the top-k features
- name: Bio (Biopython)
  role: Maps the k-mers composing selected CKGs to gene annotations via the CKG look-up
    table
- name: scSimClassify
  role: Hosts the classification module (four methods including RF and GBM) and the
    upstream k-mer/CKG/cell matrix generation whose outputs feed classification
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:adcf015934f8ab4aec8afe96a1a16d29b2e6a7e5b66a92d977621d1cf4204c46
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - Bio
  - Bio (Biopython)
  - numpy
  - pandas
  - scSimClassify
  - scikit-learn (sklearn)
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# random-forest-gradient-boosting-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains scikit-learn random forest (RF) and gradient boosting (GBM) classifiers on single-cell feature representations — CKG compressed k-mer group cell matrices and gene expression (GE) matrices — to classify cell types, then ranks features via feature_importances_ to test whether CKG features capture signal comparable to gene expression features.

## When to use

Use when classifying single-cell RNA-seq cells from CKG or gene expression feature matrices with supervised tree-based models, or when RF/GBM feature_importances_ rankings are needed to compare feature representations.

## When NOT to use

- Input is raw reads or jellyfish k-mer files (BAM/FASTA) — the k-mer matrix, CKG generation, and cell matrix generation modules must run upstream before classification.
- The goal is CKG generation or k-mer compression itself (simhash fingerprinting); this skill starts from existing CKG/GE feature matrices with labels.
- No cell type labels are available — RF and GBM here are supervised classifiers that require the provided cell labels as targets.

## Inputs

- Gene expression (GE) feature matrix with cell labels (cell x gene, pandas-readable)
- CKG cell matrix (cell x compressed k-mer group) from the scSimClassify cell matrix generation module
- CKG look-up table resolving which k-mers compose each CKG (for Bio-based gene annotation mapping)
- Cell type labels used as classification targets

## Outputs

- Trained RF and GBM classifiers with cell type predictions and accuracy per feature representation (CKG vs GE)
- Per-model feature_importances_ rankings and top-10 CKG / top-10 GE feature lists
- shared_genes.csv listing shared gene identifiers and the CKG/GE features they are assigned to

## How to apply

Load the gene expression matrix with its cell labels and the CKG cell matrix (built by the scSimClassify cell matrix generation module, where cell features are transformed from k-mer abundance to CKG expression) with pandas, separating feature matrices X from the label vector y. Fit sklearn's RandomForestClassifier and GradientBoostingClassifier on each of the two representations (e.g., the intra-dataset experiments); the classification module of scSimClassify provides four classification methods, of which RF and GBM are the tree-based choices that expose feature_importances_. Rank each model's features by feature_importances_ and select the top-10 CKG and top-10 GE features. For gene-level interpretation, use the Bio package with the CKG look-up table (generated by selecting informative k-mers and simhashing them into fingerprints) to map the k-mers composing each selected CKG to gene annotations, then intersect the CKG gene annotation sets with the top-10 GE genes to find shared genes. Compare classification accuracy between the CKG and GE representations — the paper reports CKG features outperform gene expression features in the majority of experimental cases. Persist the intersection to shared_genes.csv with pandas, listing shared gene identifiers and the features assigning them.

## Related tools

- **scikit-learn (sklearn)** (Supplies the RandomForest and GradientBoosting classifiers fitted on CKG and GE matrices; feature_importances_ provides the top-k feature rankings)
- **pandas** (Loads the gene expression matrix with cell labels and the CKG cell matrix; writes shared_genes.csv overlap output)
- **numpy** (Array operations for sorting feature_importances_ and slicing the top-k features)
- **Bio (Biopython)** (Maps the k-mers composing selected CKGs to gene annotations via the CKG look-up table)
- **scSimClassify** (Hosts the classification module (four methods including RF and GBM) and the upstream k-mer/CKG/cell matrix generation whose outputs feed classification) — github.com/digi2002/scSimClassify

## Examples

```
from sklearn.ensemble import RandomForestClassifier; import numpy as np; top10_ckg = np.argsort(RandomForestClassifier().fit(X_ckg, y_labels).feature_importances_)[::-1][:10]
```

## Evaluation signals

- shared_genes.csv exists and contains gene identifier and feature annotation fields.
- Every shared gene in shared_genes.csv appears in the annotation sets of both the top-10 CKG and top-10 GE feature lists.
- Each model's feature_importances_ vector has one entry per input feature column and its values sum to 1.
- The top-10 CKG and top-10 GE lists contain exactly 10 features ordered by decreasing importance.
- Accuracy comparison reproduces the paper's direction: CKG features perform at least as well as gene expression features in the majority of cases.

## Limitations

- CKG features are reference-free compressed k-mer groups derived without alignment to a reference genome; gene-level interpretation requires the CKG look-up table and Bio-based k-mer-to-gene mapping, so overlap results depend on annotation coverage.
- The repository targets python 3.6 with packages simhash, keras, sklearn, pandas, numpy, Bio, and instructs users to modify paths to run the modules — not turnkey.
- The CKG-over-GE accuracy advantage is empirical ('majority of experimental cases') and not guaranteed for every dataset.

## Evidence
