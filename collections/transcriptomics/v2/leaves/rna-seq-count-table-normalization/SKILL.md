---
name: rna-seq-count-table-normalization
description: Use when raw RNA-Seq count tables (e.g. DREAM 4 In Silico time-series
  or C. beijerinckii NRRL B-598 data) must be normalized with CPM, RPKM, or TPM prior
  to mutual-information GRN inference, or when benchmarking normalization effects
  on network metrics.
when_to_use_negative:
- Input count table is already normalized (e.g. the Bacillus subtilis 168 dataset
  already normalized by RPKM) — re-normalizing would distort the data
- The analysis goal is Boolean Network logic-rule assignment or database curation
  steps rather than upstream count-table processing
- Single-cell RNA-Seq data is the input, since comparable high-performance tools were
  noted to be designed for single-cell data and Augusta targets bulk standard RNA-Seq
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Provides the normalization step (CPM, RPKM, TPM via normalization_type) applied
    to raw count tables before mutual-information GRN inference
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Language of the Augusta package used to script the per-normalization pipeline
    runs and compute confusion-matrix metrics
  repo: https://github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:1dc9e4eca0f543a6b06a2b50700b2ae211a3a68b3ba4a0cdd031dc2db3e5b916
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - MEME Suite
  - Python
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rna-seq-count-table-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize raw RNA-Seq count tables using CPM, RPKM, or TPM within the Augusta pipeline before GRN inference; the choice of normalization measurably shifts sensitivity, specificity, and accuracy of the inferred network on DREAM 4.

## When to use

Use when raw RNA-Seq count tables (e.g. DREAM 4 In Silico time-series or C. beijerinckii NRRL B-598 data) must be normalized with CPM, RPKM, or TPM prior to mutual-information GRN inference, or when benchmarking normalization effects on network metrics.

## When NOT to use

- Input count table is already normalized (e.g. the Bacillus subtilis 168 dataset already normalized by RPKM) — re-normalizing would distort the data
- The analysis goal is Boolean Network logic-rule assignment or database curation steps rather than upstream count-table processing
- Single-cell RNA-Seq data is the input, since comparable high-performance tools were noted to be designed for single-cell data and Augusta targets bulk standard RNA-Seq

## Inputs

- Raw RNA-Seq count table in CSV format (e.g. DREAM 4 In Silico time-series count table, 100 genes x 21 samples, or C. beijerinckii 5442-gene, six time-point table)
- Normalization method selection string ('CPM', 'RPKM', or 'TPM') for Augusta's normalization_type parameter
- Annotated genome sequence in GenBank file format (e.g. Escherichia coli BW25113)

## Outputs

- Normalized count table used as input to mutual-information GRN inference
- Per-normalization-method benchmark metrics (sensitivity, specificity, accuracy), e.g. normalization_benchmark_metrics.csv
- Inferred initial GRN per normalization variant

## How to apply

Run Augusta's normalization step on the raw count table by setting normalization_type to 'CPM', 'RPKM', or 'TPM'; the article notes normalization of raw counts is a crucial initial step to eliminate systematic experimental bias and correct technical variations. To benchmark, rerun the full pipeline once per setting (including no normalization) with all other parameters at defaults, inferring the initial GRN via mutual information with expression values binned into D = min(n/5, 10) bins and edges oriented by the most significant difference (MSD) between adjacent time points. Convert inferred networks to the undirected typed gold standard format (drop interaction types, make negative edges positive), then compute sensitivity, specificity, and accuracy per variant. On DREAM 4 (100 genes, 21 samples), TPM preserved high specificity (0.91) and accuracy (0.89) while improving sensitivity over non-normalized data, whereas CPM and RPKM improved specificity/accuracy but reduced sensitivity. Write per-method metrics to a CSV for comparison.

## Related tools

- **Augusta** (Provides the normalization step (CPM, RPKM, TPM via normalization_type) applied to raw count tables before mutual-information GRN inference) — https://github.com/JanaMus/Augusta
- **Python** (Language of the Augusta package used to script the per-normalization pipeline runs and compute confusion-matrix metrics) — https://github.com/JanaMus/Augusta

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input = 'MyCT_file.csv', genbank_file_input = 'MyGB_file.gb', normalization_type = 'TPM')
```

## Evaluation signals

- The script completes for all four variants (none, CPM, RPKM, TPM) and emits a metrics CSV with sensitivity, specificity, and accuracy rows per method
- DREAM 4 metrics are of the same order as the paper's reported values (e.g. sensitivity ~0.06, specificity ~0.91, accuracy ~0.89 with refinement; sensitivity 0.38, specificity 0.54, accuracy 0.54 without refinement)
- TPM variant preserves high specificity and accuracy while improving sensitivity relative to non-normalized data, per the paper's finding
- Mutual-information binning uses D = min(n/5, 10) bins on the normalized values before edge orientation via MSD between adjacent time points
- Inferred networks are converted to the undirected typed gold standard (interaction types dropped, all negative edges converted to positive) before confusion-matrix computation

## Limitations

- Normalization choice trades off metrics: CPM and RPKM slightly improved specificity and accuracy but considerably reduced sensitivity on DREAM 4
- Refinement steps after normalization are computationally very demanding and require additional knowledge of a genome (GenBank annotation)
- Multiple RNA-Seq experiments from different environmental conditions cannot be processed in a single computation yet (future work)
- Only the provided methods (CPM, RPKM, TPM) are available; alternative normalizers are not supported by Augusta

## Evidence

- [methods] rationale for normalization: [evidence span withheld — anchor status: not_found]
- [results] benchmark metrics with refinement: [evidence span withheld — anchor status: not_found]
- [results] benchmark metrics without refinement: [evidence span withheld — anchor status: not_found]
- [readme] README invocation showing normalization_type: [evidence span withheld — anchor status: external_doc]
