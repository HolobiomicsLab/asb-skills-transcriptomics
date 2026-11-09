---
name: zenodo-dataset-retrieval
description: Use when a paper states its data are deposited on Zenodo (a doi:10.5281/zenodo.*
  DOI) and you must download the package, parse the deposited metadata/annotation
  tables, and verify reported dataset statistics.
when_to_use_negative:
- The dataset of interest is hosted only in SRA/GEO/other repositories rather than
  a Zenodo deposit — use a repository-specific retrieval workflow instead
- The goal is to run CellO classification on new expression data rather than to audit
  or reproduce the training-set statistics
- The deposited package lacks the per-sample metadata or Cell Ontology annotation
  tables needed to recompute the counts
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellO
  role: Source of the deposited training data package; its curated bulk RNA-seq training
    set from the SRA is the object being retrieved and audited
  repo: https://github.com/deweylab/CellO
- name: pandas
  role: Used to load the deposited sample metadata table and count unique samples,
    studies, and Cell Ontology terms
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:f2008c0c9c90b91bfae25ffa9904269b2acbabffae0663a61db3ed33ac4ac740
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
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# zenodo-dataset-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve and verify a deposited research dataset from a Zenodo record (e.g., CellO's curated bulk RNA-seq training set at doi:10.5281/zenodo.4289064) and reproduce its reported curation statistics by programmatically counting samples, studies, and annotation terms.

## When to use

Use when a paper states its data are deposited on Zenodo (a doi:10.5281/zenodo.* DOI) and you must download the package, parse the deposited metadata/annotation tables, and verify reported dataset statistics.

## When NOT to use

- The dataset of interest is hosted only in SRA/GEO/other repositories rather than a Zenodo deposit — use a repository-specific retrieval workflow instead
- The goal is to run CellO classification on new expression data rather than to audit or reproduce the training-set statistics
- The deposited package lacks the per-sample metadata or Cell Ontology annotation tables needed to recompute the counts

## Inputs

- Zenodo record DOI (e.g., 10.5281/zenodo.4289064) referencing the deposited data package
- Deposited sample metadata table (sample IDs and study accessions, readable with pandas)
- Per-sample Cell Ontology (CL term) annotation table

## Outputs

- Counts of unique samples, unique studies, total distinct cell type terms, and most-specific cell type terms
- training_set_statistics.csv (single row with the four curation statistics)

## How to apply

Download the dataset package from the cited Zenodo record (for CellO, doi:10.5281/zenodo.4289064, which holds the curated bulk RNA-seq training data drawn from the Sequence Read Archive). Load the deposited sample metadata table with pandas, identify sample identifiers and their associated study accessions, and count unique samples and unique studies. Parse the per-sample Cell Ontology annotations to count distinct CL term IDs and the subset of terms that are the most-specific annotation for at least one sample. Assemble the counts into a single-row CSV (e.g., training_set_statistics.csv) and check internal consistency — most-specific terms must be ≤ total terms — before comparing against the paper's dataset-construction description (CellO: 4,293 samples, 264 studies, 310 cell type terms, 113 most-specific).

## Related tools

- **CellO** (Source of the deposited training data package; its curated bulk RNA-seq training set from the SRA is the object being retrieved and audited) — https://github.com/deweylab/CellO
- **pandas** (Used to load the deposited sample metadata table and count unique samples, studies, and Cell Ontology terms)

## Examples

```
import pandas as pd; meta = pd.read_csv('zenodo_4289064/sample_metadata.tsv', sep='\t'); print(meta['sample_id'].nunique(), meta['study_accession'].nunique())
```

## Evaluation signals

- The four counts must be internally consistent: most-specific cell type terms (113) ≤ total cell type terms (310)
- Recomputed counts match the paper's curation statistics: 4,293 samples, 264 studies, 310 cell type terms, 113 most-specific terms
- Unique study count equals the number of distinct SRA study accessions in the metadata table
- training_set_statistics.csv contains exactly one row with the four expected numeric fields and no missing values

## Limitations

- Counts depend on the deposited Zenodo version remaining available and unchanged; the record is pinned to doi:10.5281/zenodo.4289064
- The training set covers only human primary cell bulk RNA-seq from the SRA; no curated mouse training set exists yet and future work depends on standardized mouse metadata in MetaSRA
- Determining 'most-specific' terms requires interpreting the per-sample annotation structure, which may vary if the deposited annotation format changes

## Evidence
