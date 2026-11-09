---
name: most-specific-cell-type-annotation-analysis
description: Use when reproducing or validating CellO's bulk RNA-seq training set
  curation counts (samples, studies, CL terms, most-specific terms) or when analyzing
  most-specific cell type annotations against the Cell Ontology hierarchy.
when_to_use_negative:
- 'Input samples are non-human (e.g., mouse): CellO''s training set covers only human
  primary cell bulk RNA-seq data from the SRA, and no curated mouse training set exists
  yet'
- 'The goal is de novo clustering or marker-gene-based annotation: this skill audits
  ontology-based hierarchical annotation statistics, not marker gene or flat-classifier
  workflows'
- The input already contains curated cell type labels and the task is not ontology-based
  re-annotation or training-set statistics reproduction
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: CellO
  role: Source of the curated bulk RNA-seq training set and the hierarchical Cell
    Ontology classifier whose most-specific annotations are analyzed; installable
    via 'pip install cello-classify'
  repo: https://github.com/deweylab/CellO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:24bafc7fd13b31b5e74e0a2c30b02a70ccd3ebead5188e302679aebc24b38221
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# most-specific-cell-type-annotation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces CellO's curated bulk RNA-seq training-set curation statistics (4,293 samples, 264 studies, 310 Cell Ontology terms, 113 most-specific cell types) by auditing the deposited sample metadata and per-sample CL annotations, grounding the most-specific cell type annotation task CellO performs against the Cell Ontology.

## When to use

Use when reproducing or validating CellO's bulk RNA-seq training set curation counts (samples, studies, CL terms, most-specific terms) or when analyzing most-specific cell type annotations against the Cell Ontology hierarchy.

## When NOT to use

- Input samples are non-human (e.g., mouse): CellO's training set covers only human primary cell bulk RNA-seq data from the SRA, and no curated mouse training set exists yet
- The goal is de novo clustering or marker-gene-based annotation: this skill audits ontology-based hierarchical annotation statistics, not marker gene or flat-classifier workflows
- The input already contains curated cell type labels and the task is not ontology-based re-annotation or training-set statistics reproduction

## Inputs

- CellO training data package from Zenodo record doi:10.5281/zenodo.4289064
- Sample metadata table (sample IDs and SRA study accessions) readable with pandas
- Per-sample Cell Ontology (CL) term annotations
- Gene expression matrix in TSV, CSV, HDF5, or 10x format (matrix.mtx, genes.tsv, barcodes.tsv) if annotating cells with cello_predict

## Outputs

- training_set_statistics.csv with one row of four counts: samples, studies, total cell type terms, most-specific cell type terms
- <output_prefix>.most_specific.tsv mapping each cell to its most-specific predicted cell type (if running cello_predict)
- <output_prefix>.probability.tsv and <output_prefix>.binary.tsv classification tables (if running cello_predict)

## How to apply

Download the CellO training data package from the Zenodo record doi:10.5281/zenodo.4289064 and load the deposited sample metadata table with pandas to extract sample identifiers and their associated SRA study accessions. Count unique samples (expected 4,293) and unique studies (expected 264), then parse the per-sample Cell Ontology annotations to count distinct CL term IDs (expected 310) and the subset of terms that are the most-specific annotation for at least one sample (expected 113). Assemble the four counts into a single row and write training_set_statistics.csv. The decision point is the most-specific definition: a CL term is most-specific only if it is the deepest annotation for at least one sample, mirroring how CellO's hierarchy lets 'the algorithm to place a cell internally within the graph rather than at a leaf node'. Validate that most-specific terms <= total terms and that all four counts match the paper's dataset-construction description.

## Related tools

- **CellO** (Source of the curated bulk RNA-seq training set and the hierarchical Cell Ontology classifier whose most-specific annotations are analyzed; installable via 'pip install cello-classify') — https://github.com/deweylab/CellO

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- The four reproduced counts must exactly match the paper's curation statistics: 4,293 samples, 264 studies, 310 cell type terms, and 113 most-specific cell types
- Internal consistency invariant: the count of most-specific cell type terms (113) must be <= the count of total cell type terms (310)
- Each of the 113 most-specific CL terms must be the deepest annotation for at least one of the 4,293 samples in the metadata table
- If running cello_predict, every input cell must appear in <output_prefix>.most_specific.tsv, and the most-specific prediction must be logically consistent with the binary decision matrix (all ancestors of a predicted term are also predicted)
- Unique study counts must be computed over distinct SRA study accessions in the metadata, not per-sample rows

## Limitations

- CellO's training set covers only human, healthy, untreated primary samples; no comprehensive curated mouse training set exists and future curation depends on standardized mouse metadata in MetaSRA
- Classifier calibration lags behind average-precision performance when evaluated by F1-score, affecting binary yes-no cell type decisions
- Determining the optimal clustering resolution for scRNA-seq annotation is an unsolved problem that affects CellO's cluster-based accuracy
- Input gene sets must match the classifier's training genes, otherwise a new model must be trained with the -t option, and genes absent from the training set are logged in genes_absent_from_training_set.tsv

## Evidence
