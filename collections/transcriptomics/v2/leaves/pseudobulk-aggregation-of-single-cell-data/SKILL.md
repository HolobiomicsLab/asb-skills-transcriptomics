---
name: pseudobulk-aggregation-of-single-cell-data
description: Use when you have a cell-level PAS matrix from scPAFA ('fast_Ucell' or
  'fast_score_genes') plus cell-level metadata and need sample-level input for MOFA-based
  multicellular pathway module discovery.
when_to_use_negative:
- The study has fewer than 15 samples (not cells) — scPAFA requires at least 15 samples
  to use the MOFA model for identifying multicellular pathway modules.
- The analysis requires single-cell resolution — pseudobulk integration of the PAS
  matrix leads to information loss since the PAS of multiple cells is aggregated.
- The input is already a sample-level feature table (aggregation would be redundant;
  proceed directly to MOFA training).
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPAFA
  role: Python library whose Step-2 pseudobulk processing reformats the single-cell
    PAS matrix into the mofapy2 input dataframe
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: mofapy2
  role: Consumer of the long-table-like pseudobulk dataframe; defines the required
    input format and trains the MOFA+ model (multi-group if batch effects are known)
  repo: https://github.com/bioFAM/mofapy2
- name: MOFA
  role: Framework whose multi-group mode groups technical batch for correction when
    known, consuming the pseudobulk PAS data
  repo: https://github.com/bioFAM/mofapy2
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:1f5ffdb6860a74a78f5dbc17a435374d93fff22cea34aa83d48804b809dbedff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - MOFA
  - Python
  - UCell
  - mofapy2
  - scPAFA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pseudobulk-aggregation-of-single-cell-data

## Summary

Step 2 of the scPAFA workflow: reformat the cell-level pathway activity score (PAS) matrix into a long-table-like pandas dataframe via pseudobulk aggregation per sample/donor and cell type, with sample/donor, cell type, and technical batch metadata, producing the input required by mofapy2 for MOFA model training.

## When to use

Use when you have a cell-level PAS matrix from scPAFA ('fast_Ucell' or 'fast_score_genes') plus cell-level metadata and need sample-level input for MOFA-based multicellular pathway module discovery.

## When NOT to use

- The study has fewer than 15 samples (not cells) — scPAFA requires at least 15 samples to use the MOFA model for identifying multicellular pathway modules.
- The analysis requires single-cell resolution — pseudobulk integration of the PAS matrix leads to information loss since the PAS of multiple cells is aggregated.
- The input is already a sample-level feature table (aggregation would be redundant; proceed directly to MOFA training).

## Inputs

- cell-level PAS matrix computed with scPAFA 'fast_Ucell' (or 'fast_score_genes') over pathway dictionary (e.g., 1636 NCATS BioPlanet pathways)
- cell-level metadata: sample/donor ID, cell type annotation, technical batch
- single-cell gene expression matrix with metadata (e.g., GSE164522, 163,347 cells; lupus PBMC atlas)

## Outputs

- long-table-like pandas dataframe of pseudobulk-aggregated PAS per sample/donor x cell type, in mofapy2 input format
- sample count summary (e.g., 65 pseudobulk CRC tumor epithelial samples; 40 CRC-CRLM samples; 175 lupus training pseudobulk samples)
- pseudobulk PAS matrix ready for MOFA model training (Step 3)

## How to apply

Take the single-cell PAS matrix computed with scPAFA's 'fast_Ucell' and reformat it into the long-table-like pandas dataframe format expected by mofapy2 (the getting_started_python input format). Aggregate PAS per sample/donor and cell type (e.g., 108,497 CRC tumor epithelial cells -> 65 pseudobulk samples; 163,347-cell CRC-CRLM dataset -> 40 samples from 10 patients; lupus atlas of 941,542 cells -> 175 training pseudobulk samples), attaching cell-level metadata columns for sample/donor, cell type, and technical batch. Decide on the MOFA grouping scheme at this stage: with clearly known batch effects, structure the data for the multi-group MOFA+ framework (batch as group) for correction; otherwise use the single-group framework. The rationale is that MOFA captures variance in PAS among different samples, so per-sample pseudobulk aggregation is required before MOFA model training in Step 3. Note that aggregation trades single-cell resolution for sample-level factor interpretability.

## Related tools

- **scPAFA** (Python library whose Step-2 pseudobulk processing reformats the single-cell PAS matrix into the mofapy2 input dataframe) — https://github.com/ZhuoliHuang/scPAFA
- **mofapy2** (Consumer of the long-table-like pseudobulk dataframe; defines the required input format and trains the MOFA+ model (multi-group if batch effects are known)) — https://github.com/bioFAM/mofapy2
- **MOFA** (Framework whose multi-group mode groups technical batch for correction when known, consuming the pseudobulk PAS data) — https://github.com/bioFAM/mofapy2

## Examples

```
jupyter notebook tutorial/steps2%263_Pseudobulk_processing_and_MOFA_model_training.ipynb
```

## Evaluation signals

- The reformatted dataframe matches the long-table-like pandas input format of the mofapy2 getting_started_python notebook, with sample/donor, cell type, and batch columns present.
- Number of pseudobulk samples equals the number of unique donors/samples after aggregation (e.g., 65 for CRC tumor epithelial cells, 40 for CRC-CRLM, 175 for the lupus training set).
- PAS values per row are aggregated across cells within each sample x cell type combination, not single-cell values.
- The downstream MOFA model trains without format errors and captures variance in PAS among the aggregated samples.
- For known batch effects, the group structure of the multi-group MOFA+ model corresponds to the technical batch column attached during reformatting.

## Limitations

- Pseudobulk integration of the PAS matrix would lead to information loss since the PAS of multiple cells is aggregated.
- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor.
- At least 15 samples (not cells) are required to use the MOFA model for identifying multicellular pathway modules.

## Evidence

- [readme] Step 2 pseudobulk reformatting into mofapy2 input: "In step 2, the single-cell PAS matrix is reformatted into a suitable input ([a long-table-like pandas dataframe])"
- [readme] Metadata attached during reformatting: "along with cell-level metadata including sample/donor, cell type, and technical batch information"
- [readme] Multi-group recommendation for batch effects: "In the presence of clearly known batch effects, we recommend using multi-group MOFA+ framework for correction"
- [methods] Purpose of the subsequent MOFA training: "MOFA model is trained to capture variance in PAS among different samples"
- [discussion] Information-loss caveat of pseudobulk aggregation: "pseudobulk integration of the PAS matrix would lead to information loss since the PAS of multiple cells is aggregated"
