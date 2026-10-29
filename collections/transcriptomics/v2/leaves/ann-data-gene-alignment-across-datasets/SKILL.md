---
name: ann-data-gene-alignment-across-datasets
description: Use when a query AnnData with different or unordered var_names must be
  passed to TOSICA.pre() alongside a trained reference model, since the reference
  and query gene sets must match exactly.
when_to_use_negative:
- The query data is already subset to the reference gene space and var_names order
  has been verified — re-subsetting is redundant.
- The query dataset uses a different genome annotation with mostly non-overlapping
  genes, so subsetting would discard too many features to support meaningful annotation.
- Input is not scanpy AnnData format (e.g. raw matrix files), which would require
  a different ingestion pipeline before alignment.
tools:
- name: TOSICA
  role: Performs the reference-to-query label transfer whose gene-space requirement
    motivates the alignment step; TOSICA.pre() consumes the aligned query AnnData
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads the query AnnData object and provides the var_names-based column subsetting
    used for gene alignment
- name: torch
  role: Backend framework for the TOSICA Transformer model (tested with torch 1.7.1)
    whose fixed input layer requires consistent gene ordering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:cc2077201b58be6be0dfb5b2e60cb4eb612389b8236fd012b4b35bc5e7c45a81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# AnnData gene alignment across datasets

## Summary

Aligns the gene space (var_names) of a query AnnData object to a reference AnnData before label-transfer prediction with TOSICA, ensuring the trained Transformer model receives genes in the identical, same-ordered feature space it was trained on.

## When to use

Use when a query AnnData with different or unordered var_names must be passed to TOSICA.pre() alongside a trained reference model, since the reference and query gene sets must match exactly.

## When NOT to use

- The query data is already subset to the reference gene space and var_names order has been verified — re-subsetting is redundant.
- The query dataset uses a different genome annotation with mostly non-overlapping genes, so subsetting would discard too many features to support meaningful annotation.
- Input is not scanpy AnnData format (e.g. raw matrix files), which would require a different ingestion pipeline before alignment.

## Inputs

- query AnnData object (.h5ad) with raw single-cell expression counts
- reference AnnData object whose var_names define the canonical gene order
- trained model weight file (model-*.pth) from TOSICA.train()
- project folder containing pathway.csv (gene set list) and gene2token_weights.csv

## Outputs

- gene-aligned query AnnData with var_names identical to ref_adata.var_names
- new_adata with obs['Prediction'], obs['Probability'], and attention matrix in X
- tosica_predictions.csv assembling predictions, probabilities, and attention matrix

## How to apply

Load the query dataset with scanpy and subset it to the reference's gene order using query_adata = query_adata[:, ref_adata.var_names]; the README explicitly warns that var_names of ref_adata and query_adata must be consistent and in the same order. This matters because TOSICA's masked linear transformation embeds expression of n genes into k pathway tokens via a mask matrix built from the gmt file during training — any gene-order mismatch would silently misalign the mask. After subsetting, call TOSICA.pre(query_adata, model_weight_path=<model-*.pth>, project=<my_project>), then extract new_adata.obs['Prediction'], obs['Probability'], and the attention matrix in new_adata.X. The alignment is the prerequisite step that makes interpretability (attention traceable to original input genes via gene2token_weights.csv) valid.

## Related tools

- **TOSICA** (Performs the reference-to-query label transfer whose gene-space requirement motivates the alignment step; TOSICA.pre() consumes the aligned query AnnData) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads the query AnnData object and provides the var_names-based column subsetting used for gene alignment)
- **torch** (Backend framework for the TOSICA Transformer model (tested with torch 1.7.1) whose fixed input layer requires consistent gene ordering)

## Examples

```
import scanpy as sc; query_adata = sc.read_h5ad('query.h5ad'); query_adata = query_adata[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, model_weight_path='./my_project/model-6.pth', project='my_project')
```

## Evaluation signals

- After subsetting, query_adata.var_names equals ref_adata.var_names element-wise (same genes, same order)
- new_adata.obs contains a non-null Prediction and Probability for every query cell, with all probabilities in [0, 1]
- Column count of the attention matrix (new_adata.X) equals the number of gene sets listed in pathway.csv
- gene2token_weights.csv row count corresponds to the aligned gene set, allowing attention tokens to be mapped back to genes
- tosica_predictions.csv exists and Prediction labels draw only from entries in the project's label_dictionary.csv

## Limitations

- The README warns that inconsistent or reordered var_names between ref_adata and query_adata breaks prediction; the fix (subsetting) drops any query genes absent from the reference, losing information.
- The Transformer framework avoids dimensionality reduction to stay traceable, so it cannot tolerate feature-space drift the way latent-space methods might absorb it.
- TOSICA accuracy depends on attention tokens masked by high-level biologically relevant pathways or regulons, so alignment must respect the gene-set mask built at training time from the gmt file.

## Evidence

- [readme] var_names warning: "Warning: the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [readme] alignment code snippet: "query_adata = query_adata[:,ref_adata.var_names]"
- [discussion] mask construction rationale: "we map the input genes to selected gene sets (gmt files)"
- [intro] traceability rationale: "the Transformer framework does not involve dimensionality reduction"
