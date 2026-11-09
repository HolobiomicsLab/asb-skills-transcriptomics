---
name: size-factor-log-normalization
description: Use when a raw scRNA-seq count matrix (h5) must be library-size corrected
  and reduced to ~2048 highly variable genes before imputation, clustering, or graph-based
  modeling as in scGCL.
when_to_use_negative:
- Input data are not raw scRNA-seq counts (e.g. already normalized, imputed, or bulk
  RNA-seq matrices)
- The downstream method does not require HVG-restricted input and would discard genes
  the >1% non-zero filter plus 2048-HVG cutoff would remove
- Quality control requires stricter cell/gene thresholds than scGCL's >1% non-zero
  criterion (e.g. minimum UMI counts or mitochondrial-gene filters)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGCL
  role: Pipeline whose Section 2.1 pre-processing defines the >1% non-zero filter,
    the size-factor log normalization N(X_i,j), and the t=2048 HVG selection feeding
    graph construction
  repo: https://github.com/zehaoxiong123/scGCL
- name: scanpy
  role: Used to perform highly-variable-gene selection of the top t=2048 genes from
    the normalized matrix
- name: Python
  role: Execution environment for the scGCL pre-processing and command-line entry
    point
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:331d3ee30f0ef78e9a6ac39fc9660503d4a58faeed9f205e7a0226b8a849246d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - Python
  - scGCL
  - scanpy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Size-factor log normalization of scRNA-seq counts

## Summary

Pre-processing step from scGCL's Section 2.1 pipeline that applies quality-control filtering, then size-factor log normalization N(X_i,j) to raw scRNA-seq counts, and restricts the matrix to the top t=2048 highly variable genes via scanpy before graph construction.

## When to use

Use when a raw scRNA-seq count matrix (h5) must be library-size corrected and reduced to ~2048 highly variable genes before imputation, clustering, or graph-based modeling as in scGCL.

## When NOT to use

- Input data are not raw scRNA-seq counts (e.g. already normalized, imputed, or bulk RNA-seq matrices)
- The downstream method does not require HVG-restricted input and would discard genes the >1% non-zero filter plus 2048-HVG cutoff would remove
- Quality control requires stricter cell/gene thresholds than scGCL's >1% non-zero criterion (e.g. minimum UMI counts or mitochondrial-gene filters)

## Inputs

- Raw scRNA-seq count matrix stored in an h5 file with expression values, true labels, and other information (e.g. the adam Alzheimer's disease dataset)
- HVG count parameter t (default t=2048 per scGCL)

## Outputs

- Filtered, size-factor log-normalized expression matrix restricted to the top 2048 highly variable genes, saved as an h5 file

## How to apply

Load the h5 scRNA-seq file containing the expression matrix and true labels (e.g. --dataset adam). First apply the Section 2.1 quality-control filter, retaining only genes with non-zero expression in more than 1% of cells and cells with non-zero expression in more than 1% of genes, following the scGNN reference strategy. Then apply size-factor log normalization N(X_i,j) to the filtered count matrix to correct library-size differences between cells. Next use scanpy's highly-variable-gene selection to keep the top t=2048 HVGs from the normalized matrix. Save the normalized, HVG-restricted matrix as an h5 file for downstream graph construction and ZINB autoencoder imputation.

## Related tools

- **scGCL** (Pipeline whose Section 2.1 pre-processing defines the >1% non-zero filter, the size-factor log normalization N(X_i,j), and the t=2048 HVG selection feeding graph construction) — https://github.com/zehaoxiong123/scGCL
- **scanpy** (Used to perform highly-variable-gene selection of the top t=2048 genes from the normalized matrix)
- **Python** (Execution environment for the scGCL pre-processing and command-line entry point)

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Output h5 expression matrix contains exactly the 2048 selected highly variable genes
- Every retained gene has non-zero expression in more than 1% of cells; every retained cell has non-zero expression in more than 1% of genes
- Normalized values are log-transformed size-factor corrected counts (values shifted to a log scale relative to raw counts, with library-size differences between cells removed)
- The output file exists and can be loaded as an h5 dataset with expression matrix plus associated labels for downstream graph construction
- Running the pipeline end-to-end (e.g. python main.py --dataset adam --epochs 300) completes without errors on the pre-processed file

## Limitations

- The paper reports no numeric acceptance threshold for this pre-processing step; validation is a qualitative structural check on the output file
- The >1% non-zero filter follows the scGNN reference strategy, so datasets needing different QC criteria may be over- or under-filtered
- Restricting to t=2048 HVGs discards lowly expressed genes outside the HVG set, which may matter for analyses targeting specific genes

## Evidence

- [other] size-factor log normalization step: [evidence span withheld — anchor status: not_found]
- [other] HVG selection step: [evidence span withheld — anchor status: not_found]
- [other] QC filter definition: [evidence span withheld — anchor status: not_found]
- [readme] input file format: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
