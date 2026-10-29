---
name: ontology-edge-consistency-analysis
description: Use when you need to measure logical inconsistency of independent binary
  classifiers against a hierarchy — e.g. comparing one-vs-rest vs hierarchical CellO
  outputs, or validating a probability matrix against ontology DAG constraints.
when_to_use_negative:
- You are annotating new single-cell RNA-seq data with CellO itself (use cello_predict
  classification instead — this skill is a diagnostic on training-set classifier outputs)
- Your ontology lacks 'is a' parent-child edges or you have only a flat label set
  with no DAG structure to check
- You are working with non-human data (e.g. mouse), since no comprehensive curated
  mouse training set from the SRA exists yet
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0089
tools:
- name: CellO
  role: Provides the independent one-vs-rest binary classifiers ('IR' isotonic regression,
    CellO's default) trained per Cell Ontology term, and the N x M probability output
    used for edge-consistency checks
  repo: https://github.com/deweylab/CellO
- name: cell-type-classification-paper
  role: Repository of all code for performing the paper's experiments, including the
    leave-study-out cross-validation and inconsistency analysis
  repo: https://github.com/deweylab/cell-type-classification-paper
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:5d3e61ff1fb97c34d00403175c1ac0e77d2e81e14078258417efb98ff00034cf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deweylab/CellO
  repo_url: https://github.com/deweylab/CellO
  tools:
  - CellO
  - cell-type-classification-paper
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ontology-edge-consistency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A diagnostic skill that quantifies how frequently independently trained one-vs-rest CellO classifiers (the 'IR' isotonic-regression algorithm) produce probabilities that logically violate Cell Ontology 'is a' parent-child edges, justifying hierarchical classification.

## When to use

Use when you need to measure logical inconsistency of independent binary classifiers against a hierarchy — e.g. comparing one-vs-rest vs hierarchical CellO outputs, or validating a probability matrix against ontology DAG constraints.

## When NOT to use

- You are annotating new single-cell RNA-seq data with CellO itself (use cello_predict classification instead — this skill is a diagnostic on training-set classifier outputs)
- Your ontology lacks 'is a' parent-child edges or you have only a flat label set with no DAG structure to check
- You are working with non-human data (e.g. mouse), since no comprehensive curated mouse training set from the SRA exists yet

## Inputs

- Human primary bulk RNA-seq training set from the SRA that CellO's classifiers were trained on (Zenodo doi 10.5281/zenodo.4289064)
- Cell Ontology DAG with parent-child 'is a' edges between cell type terms
- Study-of-origin metadata per sample for constructing leave-study-out cross-validation folds
- N x M cell-by-cell-type probability matrix from independent (non-hierarchy-corrected) classifier predictions

## Outputs

- inconsistency_rates.csv with three statistics: fraction of inconsistent ontology edges, fraction of samples with at least one inconsistent edge, fraction of samples with at least one severely inconsistent edge (child exceeds parent by >= 0.25)

## How to apply

Load the human primary bulk RNA-seq training set from the SRA (data deposited as 10.5281/zenodo.4289064) and split samples by study to form leave-study-out cross-validation folds. For each Cell Ontology term, train an independent binary classifier using CellO's default 'IR' (isotonic regression) algorithm on the training folds of each split. Predict on each held-out study to obtain an N x M probability matrix over cells and cell types (the same layout as CellO's <output_prefix>.probability.tsv output). For every parent-child 'is a' edge in the Cell Ontology, flag an inconsistency when the child's probability exceeds its parent's, and a severe inconsistency when the child exceeds the parent by at least 0.25. Aggregate flagged edges and samples into three statistics written to inconsistency_rates.csv: fraction of inconsistent edges, fraction of samples with at least one inconsistent edge, and fraction of samples with at least one severely inconsistent edge.

## Related tools

- **CellO** (Provides the independent one-vs-rest binary classifiers ('IR' isotonic regression, CellO's default) trained per Cell Ontology term, and the N x M probability output used for edge-consistency checks) — https://github.com/deweylab/CellO
- **cell-type-classification-paper** (Repository of all code for performing the paper's experiments, including the leave-study-out cross-validation and inconsistency analysis) — https://github.com/deweylab/cell-type-classification-paper

## Examples

```
cello_predict -a IR -d TSV -u LOG1_TPM -s FULL_LENGTH training_set.tsv -o probs
```

## Evaluation signals

- Fraction of inconsistent ontology edges under leave-study-out cross-validation should be about 12.1%, matching the paper's reported independent-classifier rate
- More than 99% of samples should have at least one inconsistent edge
- About 34% of samples should have at least one severely inconsistent edge, where the child probability exceeds the parent's by >= 0.25
- The probability matrix must be N x M over N cells and M cell types, and an inconsistency is flagged only when child probability > parent probability
- Statistics must be aggregated over all parent-child 'is a' edges in the Cell Ontology, not just leaf terms

## Limitations

- The analysis only checks 'is a' edges; the Cell Ontology does not encode lineage-based or evolutionary relationships that could also be used to improve accuracy
- Inconsistencies arise when cell types share a parent via the currently encoded 'is a' relationship but are purported to have divergent evolutionary origins
- Classifier calibration is expected to improve only as more training data per cell type become available, so inconsistency rates depend on training-data volume

## Evidence

- [discussion] Source of inconsistencies: [evidence span withheld — anchor status: not_found]
