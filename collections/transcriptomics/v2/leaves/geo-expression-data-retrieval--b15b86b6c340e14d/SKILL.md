---
name: geo-expression-data-retrieval
description: Use when a workflow needs subtype-labeled bulk expression profiles as
  raw input for marker-gene detection or deconvolution benchmarking, and the dataset
  is identified by a GEO accession number such as GSE28490.
when_to_use_negative:
- The data is a precomputed marker-gene list or already-aggregated subtype means rather
  than raw subtype expression profiles
- Only a handful of genes are present — p-value estimation may not work ('too few
  genes may not work for predicting p-values'); see the Example_GSE28490 vignette
- The expression data is not batch-corrected, since 'the input of COT should be batch-corrected'
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: COT
  role: Consumes the retrieved expression DataFrame as df_raw and detects marker genes
    among the subtypes
  repo: https://github.com/MintaYLu/COT
- name: pandas
  role: Loads the downloaded GEO expression matrix into the df_raw DataFrame format
    COT expects
- name: Python
  role: Runtime for the retrieval and loading script (COT requires python >= 3.7.4)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:8621b0cb6270b4552d9c6fc4b4155e4895e3596726f91a8506aa792b3dd3cb71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - SciPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# geo-expression-data-retrieval

## Summary

Retrieve subtype-resolved gene expression profiles (e.g., the GSE28490 benchmark) from NCBI GEO via its accession number and load them as batch-corrected DataFrames to serve as raw input for COT marker-gene detection and deconvolution benchmarks.

## When to use

Use when a workflow needs subtype-labeled bulk expression profiles as raw input for marker-gene detection or deconvolution benchmarking, and the dataset is identified by a GEO accession number such as GSE28490.

## When NOT to use

- The data is a precomputed marker-gene list or already-aggregated subtype means rather than raw subtype expression profiles
- Only a handful of genes are present — p-value estimation may not work ('too few genes may not work for predicting p-values'); see the Example_GSE28490 vignette
- The expression data is not batch-corrected, since 'the input of COT should be batch-corrected'

## Inputs

- GEO accession number (e.g., GSE28490) identifying the benchmark subtype expression dataset
- Subtype/sample label mapping for the downloaded expression matrix

## Outputs

- Batch-corrected raw expression DataFrame (df_raw: genes x subtype samples) loadable by COT(df_raw=df_raw, normalization=False)
- Subtype label vector aligned with sample columns (e.g., five GSE28490 cell types)

## How to apply

Look up the accession number (e.g., GSE28490) on https://www.ncbi.nlm.nih.gov/geo/ and download the expression profiles of the five benchmark cell types. Load the expression matrix into a pandas DataFrame (df_raw) with genes as rows and subtype samples as columns, since COT is 'a Python module for machine learning built on top of NumPy and Pandas'. Ensure the input is batch-corrected before COT ingestion, because the COT input should be batch-corrected. Confirm the subtype label vector (e.g., subtype_label = ['A','A','B','B'] style) aligns column-wise with the samples so that generate_subtype_means can map samples to subtypes; p-value estimation may fail if the gene count is too small (see the Example_GSE28490 vignette for p-value computation).

## Related tools

- **COT** (Consumes the retrieved expression DataFrame as df_raw and detects marker genes among the subtypes) — https://github.com/MintaYLu/COT
- **pandas** (Loads the downloaded GEO expression matrix into the df_raw DataFrame format COT expects)
- **Python** (Runtime for the retrieval and loading script (COT requires python >= 3.7.4))

## Examples

```
import pandas as pd; df_raw = pd.read_csv('GSE28490_expression.tsv', index_col=0); from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False)
```

## Evaluation signals

- The loaded df_raw matches the COT input schema: a gene-indexed DataFrame with subtype sample columns, as shown in the README input table
- Data is batch-corrected before COT ingestion, per the README note on input
- Column count of df_raw matches the length of subtype_label so generate_subtype_means correctly maps samples to subtypes
- Downstream generate_subtype_means produces cot.subtypes mapping each label to its sample names and a df_mean of subtype mean values
- For GSE28490-scale gene sets, p-values compute without NaN issues; otherwise remove NaNs before estimate_p_values()

## Limitations

- COT cannot calculate p-values when the gene count is too limited; the paper points to the Example_GSE28490 for p-values computation
- Input must be batch-corrected; uncorrected batch effects will be carried into marker-gene detection
- GSE28490 must be obtained manually from GEO (https://www.ncbi.nlm.nih.gov/geo/) under accession GSE28490; no direct download command is provided in the README

## Evidence

- [supplementary] Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490.: "Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490."
- [readme] Note that the input of COT should be batch-corrected.: "Note that the input of COT should be batch-corrected."
- [methods] Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation.: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
- [readme] COT is a Python module for machine learning built on top of NumPy and Pandas, and is distributed under the MIT license.: "COT is a Python module for machine learning built on top of NumPy and Pandas, and is distributed under the MIT license."
