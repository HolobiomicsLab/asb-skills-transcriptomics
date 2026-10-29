---
name: cell-ontology-term-counting
description: Use when you need to verify or reproduce the dataset-construction statistics
  of CellO's SRA-derived bulk RNA-seq training set (samples, studies, CL terms, most-specific
  terms) from the deposited Zenodo package.
when_to_use_negative:
- You want to run CellO classification on new scRNA-seq or bulk data — use cello_predict
  instead of recomputing training-set statistics
- Your goal is counting cell types predicted by CellO on a new dataset (the .binary.tsv
  / .most_specific.tsv outputs), not auditing the curated training corpus
- The training package is for a species other than human — no comprehensive curated
  mouse training set exists yet
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Tool whose curated bulk RNA-seq training set (SRA-derived, CL-annotated) is
    being audited; installed via pip install cello-classify
  repo: https://github.com/deweylab/CellO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:bd919c396e3de8f9aad2162fc245d00f01a0d6f9dfbc3622085f68db68a08eb2
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-ontology-term-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces CellO's curated bulk RNA-seq training-set curation statistics — 4,293 samples from 264 studies labeled with 310 Cell Ontology (CL) terms, of which 113 were most-specific cell types — by downloading the Zenodo data package and counting samples, studies, CL terms, and most-specific annotations.

## When to use

Use when you need to verify or reproduce the dataset-construction statistics of CellO's SRA-derived bulk RNA-seq training set (samples, studies, CL terms, most-specific terms) from the deposited Zenodo package.

## When NOT to use

- You want to run CellO classification on new scRNA-seq or bulk data — use cello_predict instead of recomputing training-set statistics
- Your goal is counting cell types predicted by CellO on a new dataset (the .binary.tsv / .most_specific.tsv outputs), not auditing the curated training corpus
- The training package is for a species other than human — no comprehensive curated mouse training set exists yet

## Inputs

- CellO training data package downloaded from Zenodo record doi:10.5281/zenodo.4289064
- Sample metadata table (sample identifiers and study accessions) from the Zenodo deposit
- Per-sample Cell Ontology (CL) annotation mappings from the training set

## Outputs

- training_set_statistics.csv with one row of four counts: samples (4,293), studies (264), total cell type terms (310), most-specific cell type terms (113)

## How to apply

Download the CellO training data package from Zenodo record doi:10.5281/zenodo.4289064, which holds the data used in the CellO paper. Load the deposited sample metadata table with pandas and identify sample identifiers and their associated study accessions; count unique samples and unique studies (expected: 4,293 and 264). Parse the per-sample Cell Ontology annotations to count distinct CL term IDs (expected: 310) and the subset of terms that are the most-specific annotation for at least one sample (expected: 113). The rationale is that CellO's classifiers were trained on nearly all human primary cell bulk RNA-seq data in the SRA, so the curation counts are the authoritative description of that training set; assembling them as a single CSV row (training_set_statistics.csv) makes the check reproducible.

## Related tools

- **CellO** (Tool whose curated bulk RNA-seq training set (SRA-derived, CL-annotated) is being audited; installed via pip install cello-classify) — https://github.com/deweylab/CellO

## Examples

```
import pandas as pd; meta = pd.read_csv('sample_metadata.tsv', sep='\t'); stats = (meta.shape[0], meta['study'].nunique())  # expect (4293, 264); counts of CL terms and most-specific terms parsed from per-sample annotations
```

## Evaluation signals

- Sample count equals 4,293 and unique study count equals 264 after deduplication of the metadata table
- Distinct Cell Ontology CL term count equals 310
- Most-specific term count equals 113 and satisfies the invariant most-specific terms (113) <= total terms (310)
- All four counts match the curation statistics reported in the paper's dataset-construction description
- CSV file named training_set_statistics.csv is written with the four counts as a single row

## Limitations

- The training set covers only healthy, untreated human primary bulk RNA-seq samples from the SRA; a comparable curated mouse training set does not yet exist
- Counts are only reproducible against the specific Zenodo deposit (doi:10.5281/zenodo.4289064); later updates to the deposit or Cell Ontology version would change CL term counts

## Evidence
