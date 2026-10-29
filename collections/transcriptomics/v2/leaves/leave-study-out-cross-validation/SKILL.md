---
name: leave-study-out-cross-validation
description: Use when evaluating whether hierarchical cell-type classification probabilities
  are logically consistent with the Cell Ontology DAG, by reproducing inconsistency
  rates under study-level held-out validation.
when_to_use_negative:
- Input is single-cell data being annotated for downstream use rather than benchmarking
  — this skill is a validation experiment, not an annotation run
- You need CellO's final hierarchical predictions (e.g. the .binary.tsv or .most_specific.tsv
  outputs) for a new sample; this skill specifically measures the independent-classifier
  inconsistency failure mode
- The data are non-human, treated, or non-primary samples — CellO's training set is
  human, healthy, untreated primary bulk RNA-seq from the SRA
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Python package providing the 'IR' (isotonic regression) hierarchical classification
    algorithm and the one-vs-rest classifiers trained on SRA bulk RNA-seq data; used
    to train per-term binary classifiers and produce probability matrices
  repo: https://github.com/deweylab/CellO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:ce54e6b84adf44d98d1c0d09d0b2cade70cb6ac73b7407b3d47ba91c85fa2b0b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# leave-study-out-cross-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol that trains independent one-vs-rest isotonic-regression (IR) classifiers on CellO's human primary bulk RNA-seq SRA training set, holds out entire studies as cross-validation folds, and quantifies how often classifier outputs violate the Cell Ontology 'is a' parent-child constraints.

## When to use

Use when evaluating whether hierarchical cell-type classification probabilities are logically consistent with the Cell Ontology DAG, by reproducing inconsistency rates under study-level held-out validation.

## When NOT to use

- Input is single-cell data being annotated for downstream use rather than benchmarking — this skill is a validation experiment, not an annotation run
- You need CellO's final hierarchical predictions (e.g. the .binary.tsv or .most_specific.tsv outputs) for a new sample; this skill specifically measures the independent-classifier inconsistency failure mode
- The data are non-human, treated, or non-primary samples — CellO's training set is human, healthy, untreated primary bulk RNA-seq from the SRA

## Inputs

- Human primary bulk RNA-seq training samples from the Sequence Read Archive used to train CellO's classifiers (Zenodo deposit 10.5281/zenodo.4289064)
- Study/source metadata per sample for constructing leave-study-out folds
- Cell Ontology graph with parent-child 'is a' edges
- Gene expression matrix compatible with CellO's trained classifier gene set

## Outputs

- N x M cell-by-cell-type probability matrix per held-out study (CellO .probability.tsv format)
- inconsistency_rates.csv with three statistics: fraction of inconsistent ontology edges, fraction of samples with at least one inconsistent edge, fraction of samples with at least one severely inconsistent edge (child exceeds parent by >= 0.25)

## How to apply

Load CellO's human primary bulk RNA-seq training data (deposited on Zenodo, doi 10.5281/zenodo.4289064) and split samples by source study to build leave-study-out folds, so no sample from a held-out study contributes to training. For each Cell Ontology term, train an independent binary classifier using the 'IR' (isotonic regression) algorithm, CellO's default, on the training folds of each split. Predict on each held-out study to obtain an N x M probability matrix over cells and cell types (CellO's <output_prefix>.probability.tsv format). For every parent-child 'is a' edge in the Cell Ontology, flag an inconsistency when a child's probability exceeds its parent's, and a severe inconsistency when the child exceeds the parent by at least 0.25. Aggregate over folds to compute the fraction of inconsistent edges, the fraction of samples with at least one inconsistent edge, and the fraction with at least one severely inconsistent edge, writing the results to inconsistency_rates.csv.

## Related tools

- **CellO** (Python package providing the 'IR' (isotonic regression) hierarchical classification algorithm and the one-vs-rest classifiers trained on SRA bulk RNA-seq data; used to train per-term binary classifiers and produce probability matrices) — https://github.com/deweylab/CellO

## Examples

```
cello_predict -a IR -d TSV -u COUNTS -s 3_PRIME -t -o test example_input/LX653_tumor.tsv
```

## Evaluation signals

- Fraction of inconsistent ontology edges matches the paper's independent-classifier rate of about 12.1%
- More than 99% of held-out samples have at least one inconsistent ontology edge
- About 34% of held-out samples have at least one severely inconsistent edge (child probability exceeding parent by >= 0.25)
- inconsistency_rates.csv contains exactly the three aggregated statistics with no per-cell predictions leaking across study folds
- Every flagged edge corresponds to an actual parent-child 'is a' edge in the Cell Ontology

## Limitations

- Independent classifiers can be inconsistent when cell types share a parent via the 'is a' relationship but have divergent evolutionary origins
- Classifier calibration is expected to improve only as more training data per cell type become available
- CellO's classifiers were trained on nearly all human primary cell bulk RNA-seq data in the SRA; results depend on that training corpus and do not extend to mouse or other species

## Evidence

- [discussion] inconsistencies in the independent classifiers approach arise for cell types sharing a parent via 'is a' with divergent origins: [evidence span withheld — anchor status: not_found]
