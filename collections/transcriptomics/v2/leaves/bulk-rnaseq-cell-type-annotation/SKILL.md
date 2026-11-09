---
name: bulk-rnaseq-cell-type-annotation
description: Use when annotating human primary-cell RNA-seq samples (bulk or single-cell)
  with Cell Ontology terms, or when auditing whether independent one-vs-rest classifiers
  produce probabilities inconsistent with the ontology's 'is a' edges.
when_to_use_negative:
- 'Input is non-human data (e.g. mouse): the paper notes no comprehensive curated
  mouse training set exists yet and future work is required'
- Input gene set does not match the classifier's genes and you do not intend to train
  a new model with the '-t' option — CellO requires input genes to match the trained
  classifier's genes
- The goal is calibrated binary yes/no decisions rather than hierarchical probabilities
  — the discussion notes performance with F1-score lags average precision and better
  calibration is needed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellO
  role: Python package providing the per-term binary classifiers ('IR' isotonic regression
    default, 'CLR' cascaded logistic regression) used to produce ontology-structured
    predictions on RNA-seq data
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring the linear models' gene-wise coefficients across
    the Cell Ontology to interpret cell type expression signals
  repo: https://github.com/uwgraphics/CellOViewer
- name: cell-type-classification-paper
  role: Repository holding all code for performing the paper's experiments, including
    the leave-study-out inconsistency analysis
  repo: https://github.com/deweylab/cell-type-classification-paper
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:bc036367464468db991ceb55c9dd0bcf8f432044b3fd866416494e59e1bc66b3
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
  - CellO Viewer
  - cell-type-classification-paper
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bulk-rnaseq-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate human RNA-seq data against the graph-structured Cell Ontology using CellO's per-term binary classifiers (isotonic regression by default), and quantify logical inconsistencies that arise when classifiers are trained independently of the ontology's 'is a' hierarchy.

## When to use

Use when annotating human primary-cell RNA-seq samples (bulk or single-cell) with Cell Ontology terms, or when auditing whether independent one-vs-rest classifiers produce probabilities inconsistent with the ontology's 'is a' edges.

## When NOT to use

- Input is non-human data (e.g. mouse): the paper notes no comprehensive curated mouse training set exists yet and future work is required
- Input gene set does not match the classifier's genes and you do not intend to train a new model with the '-t' option — CellO requires input genes to match the trained classifier's genes
- The goal is calibrated binary yes/no decisions rather than hierarchical probabilities — the discussion notes performance with F1-score lags average precision and better calibration is needed

## Inputs

- Human primary bulk RNA-seq expression matrix from SRA samples (TSV/CSV/HDF5/10x format), genes matching CellO's training gene set
- Cell Ontology graph with parent-child 'is a' edges
- Leave-study-out cross-validation fold assignments (samples grouped by study)

## Outputs

- N x M cell-by-cell-type probability matrix (cf. <output_prefix>.probability.tsv)
- inconsistency_rates.csv with fraction of inconsistent ontology edges, fraction of samples with >=1 inconsistent edge, and fraction with >=1 severely inconsistent edge

## How to apply

Load the human primary bulk RNA-seq training set CellO was trained on (deposited at Zenodo doi=10.5281/zenodo.4289064) and split samples by study to build leave-study-out cross-validation folds. For each Cell Ontology term, train an independent binary classifier using the 'IR' (isotonic regression) algorithm that is CellO's default, then predict on each held-out study to obtain an N x M probability matrix over cells and cell types (the same structure CellO writes to <output_prefix>.probability.tsv). For every parent-child 'is a' edge in the Cell Ontology, flag an inconsistency when the child's probability exceeds its parent's, and a severe inconsistency when the child exceeds the parent by at least 0.25. Aggregate flagged edges and samples to compute the fraction of inconsistent edges, fraction of samples with at least one inconsistent edge, and fraction with a severely inconsistent edge, writing these to inconsistency_rates.csv. The rationale is that flat/independent classifiers ignore the hierarchical relationships encoded in the Cell Ontology, which CellO's hierarchical algorithms ('IR' or 'CLR' cascaded logistic regression) are designed to respect.

## Related tools

- **CellO** (Python package providing the per-term binary classifiers ('IR' isotonic regression default, 'CLR' cascaded logistic regression) used to produce ontology-structured predictions on RNA-seq data) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring the linear models' gene-wise coefficients across the Cell Ontology to interpret cell type expression signals) — https://github.com/uwgraphics/CellOViewer
- **cell-type-classification-paper** (Repository holding all code for performing the paper's experiments, including the leave-study-out inconsistency analysis) — https://github.com/deweylab/cell-type-classification-paper

## Examples

```
pip install cello-classify
```

## Evaluation signals

- Reproduced independent-classifier statistics match the paper's reported rates: about 12.1% of examined ontology edges inconsistent, more than 99% of samples with at least one inconsistent edge, and about 34% of samples with a severely inconsistent edge (child exceeding parent by >=0.25)
- Output probability table has shape N cells x M cell types with entries in [0,1], matching the <output_prefix>.probability.tsv schema
- After hierarchical ('IR') classification, no child term's probability exceeds its parent's along 'is a' edges (consistency invariant)
- Cross-validation folds are grouped by study, so each held-out study is fully excluded from training
- Classifier gene set matches the input matrix's genes (or a new model is trained via '-t' and genes_absent_from_training_set.tsv is inspected)

## Limitations

- Independent classifiers can be logically inconsistent for cell types that share a parent via 'is a' but have divergent evolutionary origins, and classifier calibration is expected to improve only as more training data per cell type become available
- Calibration gap: CellO's performance with F1-score lags its average precision, so binary yes-no decisions are less reliable than ranked probabilities
- Determining the optimal clustering in scRNA-seq data (which CellO uses before classification) is an open problem affecting cluster-based accuracy
- Confounding by sorting/preparation procedures (e.g. FACS) on gene expression versus cell type is unclear

## Evidence

- [methods] CellO makes hierarchical predictions against the Cell Ontology.: "CellO makes hierarchical predictions against the [Cell Ontology](http://www.obofoundry.org/ontology/cl.html)."
- [readme] README: CellO classifiers trained on human primary bulk RNA-seq data from SRA.: "These classifiers were trained on nearly all of the human primary cell, bulk RNA-seq data in the [Sequence Read Archive](https://www.ncbi.nlm.nih.gov/sra)."
- [discussion] Discussion: inconsistencies arise from independent classifiers and the 'is a' relationship.: [evidence span withheld — anchor status: not_found]
- [readme] Methods: default hierarchical algorithm is isotonic regression ('IR').: "-a ALGO, --algo=ALGO  Hierarchical classification algorithm to apply
                        (default='IR'). Must be one of: 'IR' - Isotonic
                        regression, 'CLR' - cascaded"
