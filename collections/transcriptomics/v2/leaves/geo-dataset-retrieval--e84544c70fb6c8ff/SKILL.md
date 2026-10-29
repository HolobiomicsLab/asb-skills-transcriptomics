---
name: geo-dataset-retrieval
description: Use when a task requires the original macaque retina bipolar-cell count
  matrix and sample metadata (macaque id, region id, sample id) needed to reproduce
  DESC clustering ARI benchmarks.
when_to_use_negative:
- The count matrix and cell metadata are already available locally (e.g., a .h5ad
  or CSV already loaded into AnnData) — re-downloading from GEO is redundant.
- The task targets a different dataset than the macaque retina bipolar-cell study
  (e.g., the paul15 myeloid progenitor dataset, which ships with Scanpy via sc.datasets.paul15()).
- The goal is method execution (training DESC or computing ARI) rather than data acquisition
  — a downstream skill applies instead.
tools:
- name: Scanpy
  role: Loads and preprocesses the retrieved count matrix (sc.pp.log1p, filter_genes_dispersion,
    sc.pp.scale) before DESC training
  repo: https://github.com/theislab/scanpy
- name: DESC
  role: Downstream clustering tool that consumes the retrieved GSE115746 matrix to
    reproduce the paper's ARI benchmarks
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:5517bd28de1b2b46ea7b71f71ee77414f0204ba3899948f451016026ee64827b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scanpy
  tools:
  - DESC
  - Keras
  - Scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# geo-dataset-retrieval

## Summary

Retrieves the GSE115746 macaque retina bipolar-cell scRNA-seq count matrix (21,017 foveal + 9,285 peripheral cells) and its cell metadata from GEO so that DESC clustering can be reproduced with and without batch information.

## When to use

Use when a task requires the original macaque retina bipolar-cell count matrix and sample metadata (macaque id, region id, sample id) needed to reproduce DESC clustering ARI benchmarks.

## When NOT to use

- The count matrix and cell metadata are already available locally (e.g., a .h5ad or CSV already loaded into AnnData) — re-downloading from GEO is redundant.
- The task targets a different dataset than the macaque retina bipolar-cell study (e.g., the paul15 myeloid progenitor dataset, which ships with Scanpy via sc.datasets.paul15()).
- The goal is method execution (training DESC or computing ARI) rather than data acquisition — a downstream skill applies instead.

## Inputs

- GEO accession GSE115746 (macaque retina bipolar-cell scRNA-seq study)
- Raw UMI count matrix: 21,017 foveal + 9,285 peripheral cells
- Cell metadata table with macaque id, region id, and sample id columns
- Reference bipolar-cell type labels for ARI evaluation

## Outputs

- Locally stored count matrix loadable into Scanpy/AnnData
- Cell metadata table with candidate batch definitions (macaque id, region id, sample id)
- Preprocessed AnnData (log1p-normalized, 1000 highly variable genes, scaled with max_value=6) ready for desc.train

## How to apply

Locate accession GSE115746 in GEO and download the raw UMI count matrix containing 21,017 foveal and 9,285 peripheral macaque retina bipolar cells, plus the cell metadata table carrying macaque id, region id, and sample id columns that define candidate batch labels. Verify cell and gene counts against the paper's description before preprocessing. Load the matrix into an AnnData object and preprocess with Scanpy (sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, sc.pp.scale with max_value=6) as specified in the DESC workflow, so the retrieved data feeds directly into desc.train with dims=[n_genes,64,32] and louvain_resolution=[0.8,1.0]. Correct retrieval is judged by whether downstream DESC clustering reaches ARI ~0.920 without batch information, matching the reported values.

## Related tools

- **Scanpy** (Loads and preprocesses the retrieved count matrix (sc.pp.log1p, filter_genes_dispersion, sc.pp.scale) before DESC training) — https://github.com/theislab/scanpy
- **DESC** (Downstream clustering tool that consumes the retrieved GSE115746 matrix to reproduce the paper's ARI benchmarks) — https://github.com/eleozzr/desc

## Examples

```
import scanpy as sc; sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6)
```

## Evaluation signals

- The downloaded matrix contains exactly 21,017 foveal and 9,285 peripheral bipolar cells, matching the paper's dataset description.
- Cell metadata includes all three batch-defining columns (macaque id, region id, sample id) needed for the four DESC training runs.
- After preprocessing and DESC training, ARI against reference bipolar-cell type labels falls within 0.919–0.970 with any batch definition.
- The ARI without batch information is ~0.920, matching the paper's reported value (in contrast to scVI's 0.242 on the same data).
- ARI values for all four batch configurations are written to ari_results.csv as required by the workflow.

## Limitations

- Reproducing the paper's exact DESC results requires tensorflow < 2.0 (desc 2.0.3); the README notes results may differ under tensorflow 2* (desc 2.1.1).
- No software version numbers for Keras or Scanpy are stated, so exact reproducibility of preprocessing may vary across versions.
- DESC removes batch effects only 'as long as technical differences across batches are smaller than true biological variations'.

## Evidence

- [other] GEO accession and cell counts: [evidence span withheld — anchor status: not_found]
- [other] ARI validation target: [evidence span withheld — anchor status: not_found]
- [readme] DESC version constraint for reproducibility: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper"
