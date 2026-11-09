---
name: geo-expression-matrix-loading
description: Use when starting from a GEO-deposited microarray dataset (tab-delimited
  probe-level matrix such as gse.txt) and the downstream analysis requires ERgene's
  DataFrame-format input for reference gene screening or normalization.
when_to_use_negative:
- Data are single-cell count matrices or RNA-seq quantifications that need alignment/quantification
  first — ERgene expects a probe-level expression matrix, and the article notes 'The
  test data are not converted by a probe' as a caveat
- Input is already a properly indexed pandas DataFrame — re-loading and re-indexing
  is unnecessary
- The analysis target is a normalization method like geNorm/Normfinder/BestKeeper,
  which use their own input formats rather than the ERgene DataFrame convention
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: ERgene
  role: Downstream library whose FindERG and normalizationdata functions consume the
    loaded DataFrame; version 1.2 requires the index to be set from the first column
  repo: https://github.com/Starlitnightly/ERgene
- name: Pandas
  role: Provides read_csv (with sep='\t') and set_index used to construct the DataFrame-format
    expression matrix from the GEO file
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-75586-5
    title: 'ERgene: Python library for screening endogenous reference genes'
  source_content_hash: sha256:d21f26bd2262828f99c35ab953ac88d0015e1958e10d26e05fafd3d0008ac6de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Starlitnightly/ERgene
  tools:
  - ERgene
  - Pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# geo-expression-matrix-loading

## Summary

Load a GEO microarray expression matrix (e.g., GSE125792 or GSE4786) into a pandas DataFrame with gene/probe IDs as the index, which is the required input format for ERgene's FindERG and normalizationdata functions.

## When to use

Use when starting from a GEO-deposited microarray dataset (tab-delimited probe-level matrix such as gse.txt) and the downstream analysis requires ERgene's DataFrame-format input for reference gene screening or normalization.

## When NOT to use

- Data are single-cell count matrices or RNA-seq quantifications that need alignment/quantification first — ERgene expects a probe-level expression matrix, and the article notes 'The test data are not converted by a probe' as a caveat
- Input is already a properly indexed pandas DataFrame — re-loading and re-indexing is unnecessary
- The analysis target is a normalization method like geNorm/Normfinder/BestKeeper, which use their own input formats rather than the ERgene DataFrame convention

## Inputs

- GEO tab-delimited expression matrix file (e.g., gse.txt from GSE125792 or GSE4786) with probe/gene IDs in the first column
- pandas DataFrame (post-loading) with probe/gene IDs as index and samples as columns

## Outputs

- Indexed pandas DataFrame ready for ERgene.FindERG and ERgene.normalizationdata

## How to apply

Download the probe-level expression matrix from GEO (e.g., GSE125792 for the McLoughlin dataset or GSE4786 for the Someya mouse cochlea data) as a tab-delimited text file. Read it with pandas.read_csv using sep='\t', then reindex with data.set_index(data.columns[0]) so the first column (probe or gene IDs) becomes the DataFrame index — this index-setting is required in ERgene version 1.2+. The resulting DataFrame, with genes/probes as rows and samples as columns, is the format ERgene.FindERG(Data, Depth) and ERgene.normalizationdata(Data, Ergname) expect; when the matrix is large, ERgene handles block calculation with every 1000 genes as one block internally.

## Related tools

- **ERgene** (Downstream library whose FindERG and normalizationdata functions consume the loaded DataFrame; version 1.2 requires the index to be set from the first column) — https://github.com/Starlitnightly/ERgene
- **Pandas** (Provides read_csv (with sep='\t') and set_index used to construct the DataFrame-format expression matrix from the GEO file)

## Examples

```
import ERgene, pandas as pd; data=pd.read_csv('gse.txt',sep='\t'); data=data.set_index(data.columns[0]); ERgene.FindERG(data,3)
```

## Evaluation signals

- The DataFrame's index equals the original first column of the GEO file (probe IDs such as ASHGV40056316 or 1439423_x_at), not a default RangeIndex
- Columns correspond one-to-one with the dataset's samples (e.g., 12 samples for the GSE125792 subset used in overlap analysis)
- Feeding the DataFrame to ERgene.FindERG runs without format errors and returns a ranked list of top-20 candidate reference genes
- Rows match the expected gene count for the dataset (e.g., 1968 genes tested in the article's benchmark run)

## Limitations

- No data availability or accession statements for the test datasets are given in the article, so the exact download path must be resolved from GEO directly
- The article's test data are probe-level; the article notes 'The test data are not converted by a probe' as an unstated mapping caveat, and probe IDs (e.g., ASHGV40056316 in GSE125792, 1439423_x_at in GSE4786) may need separate mapping to gene symbols
- ERgene version differences matter: index-setting from the first column is explicitly flagged as 'This is Ver1.2' behavior in the README

## Evidence

- [readme] readme loading call: "data=pd.read_csv('gse.txt',sep='\t')#GSE125792
data=data.set_index(data.columns[0])#This is Ver1.2"
- [discussion] DataFrame format requirement: "This function is used to screen internal reference genes. The parameter data are in DataFrame format"
- [discussion] block calculation for large matrices: "When the gene dataset is too large, block calculation is adopted. Every 1000 genes are taken as a block."
