---
name: bulk-rnaseq-training-set-curation
description: Use when you need to verify or reproduce the curated training-set composition
  of CellO's human primary cell bulk RNA-seq data (sample, study, and Cell Ontology
  term counts) from the Zenodo deposit.
when_to_use_negative:
- Your goal is to run CellO classification on new RNA-seq data rather than to audit
  the composition of its training set
- You need single-cell training data — this skill covers only the curated bulk RNA-seq
  set; no curated mouse training set exists yet
- The input dataset is not the Zenodo-deposited training package (e.g., you are working
  from raw SRA accessions without the curated Cell Ontology labels)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: The curated bulk RNA-seq training set being quantified; classifiers were trained
    on nearly all human primary cell bulk RNA-seq data in the SRA
  repo: https://github.com/deweylab/CellO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:853a3019b9477074ca3578ee2ad666aa16837f9bcc7a0c56406ad0a4ccf05d0a
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
n_claims: 10
---

# bulk-rnaseq-training-set-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the curation statistics of CellO's bulk RNA-seq training set — 4,293 samples from 264 SRA studies annotated with 310 Cell Ontology terms (113 most-specific) — by downloading the Zenodo data package (doi:10.5281/zenodo.4289064) and tabulating sample, study, and CL-term counts.

## When to use

Use when you need to verify or reproduce the curated training-set composition of CellO's human primary cell bulk RNA-seq data (sample, study, and Cell Ontology term counts) from the Zenodo deposit.

## When NOT to use

- Your goal is to run CellO classification on new RNA-seq data rather than to audit the composition of its training set
- You need single-cell training data — this skill covers only the curated bulk RNA-seq set; no curated mouse training set exists yet
- The input dataset is not the Zenodo-deposited training package (e.g., you are working from raw SRA accessions without the curated Cell Ontology labels)

## Inputs

- CellO training data package from Zenodo record doi:10.5281/zenodo.4289064
- Deposited sample metadata table (sample identifiers and study accessions)
- Per-sample Cell Ontology (CL term) annotations

## Outputs

- training_set_statistics.csv with one row containing: number of samples (4,293), number of studies (264), total cell type terms (310), most-specific cell type terms (113)

## How to apply

Download the CellO training data package from Zenodo record doi:10.5281/zenodo.4289064, which deposits the data described in the iScience paper. Load the deposited sample metadata with pandas, identify sample identifiers and their study accessions, and count unique samples and unique studies. Parse the per-sample Cell Ontology annotations to count distinct CL term IDs and the subset of terms that are the most-specific annotation for at least one sample. Assemble the four counts (samples, studies, total cell type terms, most-specific terms) as one row and write them to training_set_statistics.csv. The expected values are 4,293 samples, 264 studies, 310 cell type terms, and 113 most-specific cell types, reflecting 'nearly all of the human primary cell, bulk RNA-seq data in the Sequence Read Archive'.

## Related tools

- **CellO** (The curated bulk RNA-seq training set being quantified; classifiers were trained on nearly all human primary cell bulk RNA-seq data in the SRA) — https://github.com/deweylab/CellO

## Examples

```
import pandas as pd; md = pd.read_csv('zenodo_4289064/metadata.tsv', sep='\t'); stats = [md.shape[0], md['study'].nunique(), 310, 113]; pd.DataFrame([stats], columns=['samples','studies','terms','most_specific']).to_csv('training_set_statistics.csv', index=False)
```

## Evaluation signals

- The four counts must be internally consistent: most-specific terms (113) <= total cell type terms (310)
- Sample count must equal the number of unique sample identifiers (4,293), not rows in the metadata table
- Study count must equal the number of unique study accessions (264)
- Counts must match the curation statistics reported in the paper's dataset-construction description
- Output file must be a valid single-row CSV named training_set_statistics.csv

## Limitations

- The training set covers only human, healthy, untreated primary bulk RNA-seq samples from the SRA; comprehensive curated mouse training sets do not yet exist
- Standardized metadata availability (e.g., MetaSRA) constrains what can be curated
- The Zenodo deposit must be used as-is; re-curation from raw SRA would require reproducing the paper's metadata standardization steps

## Evidence
