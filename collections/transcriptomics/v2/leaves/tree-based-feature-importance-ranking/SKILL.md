---
name: tree-based-feature-importance-ranking
description: Use when you need to rank features from two alternative feature representations
  (e.g. CKG cell matrices and gene expression matrices) with tree-based models and
  quantify overlap between their top-ranked features.
when_to_use_negative:
- When the goal is classification accuracy itself rather than interpretability of
  top features — the article's classification module already provides four classification
  methods for that purpose
- When features are not comparable via gene annotations (e.g. k-mer groups cannot
  be resolved to genes through the CKG look-up table), since the shared-gene overlap
  step depends on that mapping
- When you only have raw reads and no gene expression matrix — the overlap analysis
  requires both feature representations
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: sklearn
  role: Trains random forest and gradient boosting classifiers and provides feature_importances_
    for ranking features
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads the gene expression matrix with cell labels and the CKG cell matrix;
    writes shared_genes.csv
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Supports matrix manipulation of feature importance scores and feature matrices
  repo: github.com/digi2002/scSimClassify
- name: Bio
  role: Maps k-mers composing selected CKG features to gene annotations using the
    CKG look-up table
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:8738e316257a1351febac89b686e08166712382294be6f1eb1e47ee91d002f66
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
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tree-based-feature-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank feature importance with scikit-learn tree ensembles (Random Forest, Gradient Boosting) applied to both CKG (compressed k-mer group) cell matrices and gene expression matrices, then compare top-ranked features by intersecting their gene annotations. Used to show that reference-free CKG features capture biologically meaningful signals comparable to gene expression features.

## When to use

Use when you need to rank features from two alternative feature representations (e.g. CKG cell matrices and gene expression matrices) with tree-based models and quantify overlap between their top-ranked features.

## When NOT to use

- When the goal is classification accuracy itself rather than interpretability of top features — the article's classification module already provides four classification methods for that purpose
- When features are not comparable via gene annotations (e.g. k-mer groups cannot be resolved to genes through the CKG look-up table), since the shared-gene overlap step depends on that mapping
- When you only have raw reads and no gene expression matrix — the overlap analysis requires both feature representations

## Inputs

- Gene expression matrix with cell labels (pandas DataFrame)
- CKG cell matrix (cell features transformed from k-mer abundance via the CKG look-up table)
- Cell labels (classification targets)
- CKG look-up table mapping k-mers to compressed k-mer groups
- Gene annotation resources accessible via the Bio package

## Outputs

- Top-10 most important GE features per model (ranked by feature_importances_)
- Top-10 most important CKG features per model (ranked by feature_importances_)
- shared_genes.csv listing shared gene identifiers and the features they are assigned to

## How to apply

Load the gene expression matrix with cell labels and the CKG cell matrix for the intra-dataset experiments with pandas, then train sklearn RandomForest and GradientBoosting classifiers on each representation using the provided cell labels as targets. Rank features by each model's tree-based feature_importances_ scores and select the top-10 GE features and top-10 CKG features. Map the k-mers composing the selected CKGs to gene annotations with the Bio package, resolving k-mer membership through the CKG look-up table generated by simhashing informative k-mer vectors into fingerprints. Intersect the gene annotations of the top-10 CKG features with the top-10 GE features to determine shared genes, and write the overlap to shared_genes.csv listing each shared gene identifier and the features it is assigned to. Rationale: the article reports that CKG features lend themselves to better scRNA-seq classification accuracy than gene expression features in the majority of experimental cases, so feature-importance agreement between the two representations supports CKG biological interpretability.

## Related tools

- **sklearn** (Trains random forest and gradient boosting classifiers and provides feature_importances_ for ranking features) — github.com/digi2002/scSimClassify
- **pandas** (Loads the gene expression matrix with cell labels and the CKG cell matrix; writes shared_genes.csv) — github.com/digi2002/scSimClassify
- **numpy** (Supports matrix manipulation of feature importance scores and feature matrices) — github.com/digi2002/scSimClassify
- **Bio** (Maps k-mers composing selected CKG features to gene annotations using the CKG look-up table) — github.com/digi2002/scSimClassify

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier().fit(ckg_matrix, cell_labels); top10 = numpy.argsort(rf.feature_importances_)[::-1][:10]
```

## Evaluation signals

- shared_genes.csv exists and contains gene identifier and feature annotation fields
- Every listed shared gene appears in the annotation sets of both the top-10 CKG and top-10 GE feature lists
- Feature rankings derive from sklearn's tree-based feature_importances_ of the trained RF and GBM models, not arbitrary scores
- Each top-10 CKG feature resolves to k-mers via the CKG look-up table before gene annotation with Bio
- A large proportion of genes are shared between top-10 CKG and top-10 GE features, consistent with the article's finding of biologically comparable signals

## Limitations

- The overlap analysis is validated only in the intra-dataset experiments of the article; transferability to cross-dataset settings is not established by this evidence
- The comparison is limited to the top-10 features per representation; conclusions may not generalize to deeper feature rankings
- scSimClassify modules require modification of paths and Python 3.6 with specific packages (simhash, keras, sklearn, pandas, numpy, Bio), so environment drift can break the workflow

## Evidence
