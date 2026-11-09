---
name: scrna-seq-feature-interpretation
description: 'Use when interpreting which genes drive scRNA-seq classification: after
  training tree-based models on CKG and gene-expression matrices, to test whether
  top-10 CKG features map to the same genes as top-10 GE features.'
when_to_use_negative:
- Input is raw reads (BAM/FASTA) that still need k-mer matrix generation or CKG generation
  — run the upstream scSimClassify modules first; this skill starts from existing
  feature matrices.
- The goal is benchmarking classification accuracy of CKG vs gene-expression features
  rather than biological interpretation of feature importance — use the classification
  module comparison instead.
- Only one feature representation exists — the gene-overlap analysis requires both
  top-10 CKG and top-10 GE feature lists to intersect.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: sklearn
  role: Trains random forest and gradient boosting classifiers on CKG and GE matrices;
    feature_importances_ ranks features to select the top-10 per representation
- name: pandas
  role: Loads the gene expression matrix with cell labels and the CKG cell matrix;
    writes shared_genes.csv with shared gene identifiers and feature annotations
- name: numpy
  role: Array operations on feature matrices and feature_importances_ rankings for
    top-10 selection
- name: Bio
  role: Maps the k-mers composing each selected CKG to gene annotations via the CKG
    look-up table
- name: scSimClassify
  role: Source of the CKG look-up table, cell matrices, and cell labels used for feature
    interpretation
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:516f048c1155f9f136d549be11b98b391f776364fe0a7bb248f91a7dc13731f2
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
  - scSimClassify
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scrna-seq-feature-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks tree-based model importances over CKG and gene-expression (GE) feature matrices from scRNA-seq classification and maps compressed k-mer groups (CKGs) to gene annotations to test whether top CKG features capture biologically meaningful signals comparable to gene-expression features.

## When to use

Use when interpreting which genes drive scRNA-seq classification: after training tree-based models on CKG and gene-expression matrices, to test whether top-10 CKG features map to the same genes as top-10 GE features.

## When NOT to use

- Input is raw reads (BAM/FASTA) that still need k-mer matrix generation or CKG generation — run the upstream scSimClassify modules first; this skill starts from existing feature matrices.
- The goal is benchmarking classification accuracy of CKG vs gene-expression features rather than biological interpretation of feature importance — use the classification module comparison instead.
- Only one feature representation exists — the gene-overlap analysis requires both top-10 CKG and top-10 GE feature lists to intersect.

## Inputs

- Gene expression (GE) feature matrix with cell labels for the intra-dataset experiments
- CKG cell matrix (cells x CKG expression) transformed from k-mer abundances via the CKG look-up table
- CKG look-up table resolving k-mer membership within each compressed k-mer group
- Provided cell labels used as classification targets

## Outputs

- shared_genes.csv listing shared gene identifiers and the top-10 CKG/GE features they are assigned to
- Top-10 ranked feature lists per representation derived from tree-based feature_importances_ scores

## How to apply

Load the gene expression matrix with cell labels and the CKG cell matrix from the intra-dataset experiments with pandas. Train sklearn's random forest and gradient boosting classifiers on each of the two feature representations using the provided cell labels as targets. Rank features by each model's tree-based feature_importances_ scores and select the top-10 GE and top-10 CKG features. Map the k-mers composing each selected CKG to gene annotations with the Bio package, resolving k-mer membership through the CKG look-up table. Intersect the gene annotations of the top-10 CKG features with the top-10 GE features to determine the shared genes. Write the overlap to shared_genes.csv with pandas, listing each shared gene identifier and the CKG/GE features it is assigned to.

## Related tools

- **sklearn** (Trains random forest and gradient boosting classifiers on CKG and GE matrices; feature_importances_ ranks features to select the top-10 per representation)
- **pandas** (Loads the gene expression matrix with cell labels and the CKG cell matrix; writes shared_genes.csv with shared gene identifiers and feature annotations)
- **numpy** (Array operations on feature matrices and feature_importances_ rankings for top-10 selection)
- **Bio** (Maps the k-mers composing each selected CKG to gene annotations via the CKG look-up table)
- **scSimClassify** (Source of the CKG look-up table, cell matrices, and cell labels used for feature interpretation) — github.com/digi2002/scSimClassify

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier().fit(ckg_matrix, cell_labels); top10 = rf.feature_importances_.argsort()[::-1][:10]
```

## Evaluation signals

- shared_genes.csv exists and contains gene identifier and feature annotation fields
- Every listed shared gene appears in the annotation sets of both the top-10 CKG and the top-10 GE feature lists
- Top-10 selections reproduce exactly from the feature_importances_ rankings of the RF and GBM models on the intra-dataset matrices
- Each k-mer in a selected CKG resolves through the CKG look-up table to a gene annotation before intersection

## Limitations

- CKGs are derived from raw reads without alignment to a reference genome, so gene-level interpretation depends on k-mer-to-gene annotation coverage and can be weak where the reference genome is incomplete or insufficient.
- Overlap is quantified only on the top-10 features of intra-dataset RF/GBM models; conclusions do not extend to lower-ranked features or cross-dataset settings without rerunning.
- The README notes 'Please modify the path to use the modules' — no turnkey CLI, so paths to matrices and the look-up table must be adapted manually.

## Evidence
