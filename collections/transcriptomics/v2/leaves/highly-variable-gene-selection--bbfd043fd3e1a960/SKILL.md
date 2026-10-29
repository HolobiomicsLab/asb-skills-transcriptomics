---
name: highly-variable-gene-selection
description: Use when preprocessing a raw scRNA-seq count AnnData (.h5ad, e.g. the
  pancreas dataset) prior to TFvelo model fitting, so that downstream neighbor graphs
  and moments are computed on a filtered, normalized set of the 2000 most variable
  genes.
when_to_use_negative:
- The dataset already has unspliced/spliced counts and a splicing-based velocity model
  (e.g. scVelo dynamical mode) is preferred, since TFvelo targets datasets lacking
  splicing information
- The input is not a raw count AnnData but an already normalized or feature-selected
  matrix, where re-running filter_and_normalize on top-2000 HVGs would double-filter
  and distort the counts
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ScVelo
  role: Provides pp.filter_and_normalize, pp.neighbors (n_neighbors=30) and pp.moments
    used for HVG selection and moment computation on total mRNA counts
  repo: github.com/theislab/scvelo
- name: TFvelo
  role: Hosts the preprocess() function in TFvelo_run_demo.py that chains filtering,
    normalization, graph building, and ENCODE/ChEA TF-pair restriction
  repo: github.com/xiaoyeye/TFvelo
- name: anndata
  role: Loads the raw count .h5ad file via read_h5ad and stores the preprocessed AnnData
- name: scanpy
  role: Part of the pinned environment (scanpy==1.8.2) supporting the preprocessing
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:b07ea1cd2ddf1da9716d7a9803965e311ad507b2d44e7d5c3c1537fd1a08ac63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  - anndata
  - scanpy
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# highly-variable-gene-selection

## Summary

Filter and normalize the scRNA-seq count matrix with scvelo's pp.filter_and_normalize, retaining the top-2000 highly variable genes before building the neighbor graph and moments used by TFvelo's TF-based velocity modeling.

## When to use

Use when preprocessing a raw scRNA-seq count AnnData (.h5ad, e.g. the pancreas dataset) prior to TFvelo model fitting, so that downstream neighbor graphs and moments are computed on a filtered, normalized set of the 2000 most variable genes.

## When NOT to use

- The dataset already has unspliced/spliced counts and a splicing-based velocity model (e.g. scVelo dynamical mode) is preferred, since TFvelo targets datasets lacking splicing information
- The input is not a raw count AnnData but an already normalized or feature-selected matrix, where re-running filter_and_normalize on top-2000 HVGs would double-filter and distort the counts

## Inputs

- Raw scRNA-seq count AnnData (.h5ad file, e.g. pancreas dataset)
- ENCODE TF-target database file (ENCODE.zip, must be decompressed)
- ChEA TF-target database (default with --TF_databases)

## Outputs

- Preprocessed AnnData with top-2000 highly variable genes, neighbor graph, and moments stored as 'TFvelo_'+dataset_name+'_demo' folder h5ad files
- Per-gene candidate TF annotations from ENCODE/ChEA regulatory pairs

## How to apply

Load the raw count AnnData with anndata.read_h5ad (or let TFvelo_run_demo.py --dataset_name pancreas auto-download it), then apply scvelo's pp.filter_and_normalize to filter the count matrix and normalize while retaining the top-2000 highly variable genes. Build the neighbor graph with scvelo pp.neighbors using n_neighbors=30, then compute scvelo pp.moments on the total (unspliced+spliced) mRNA counts over that graph. For each target gene, restrict candidate TFs to regulatory pairs present in the ENCODE and ChEA TF-target databases (the default --TF_databases setting), decompressing ENCODE.zip first. Save the preprocessed AnnData into the 'TFvelo_'+dataset_name+'_demo' folder so downstream TF-model fitting can consume it.

## Related tools

- **ScVelo** (Provides pp.filter_and_normalize, pp.neighbors (n_neighbors=30) and pp.moments used for HVG selection and moment computation on total mRNA counts) — github.com/theislab/scvelo
- **TFvelo** (Hosts the preprocess() function in TFvelo_run_demo.py that chains filtering, normalization, graph building, and ENCODE/ChEA TF-pair restriction) — github.com/xiaoyeye/TFvelo
- **anndata** (Loads the raw count .h5ad file via read_h5ad and stores the preprocessed AnnData)
- **scanpy** (Part of the pinned environment (scanpy==1.8.2) supporting the preprocessing pipeline)

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The preprocessed h5ad file exists in the 'TFvelo_'+dataset_name+'_demo' folder and loads with anndata
- The saved AnnData contains a neighbor graph built with n_neighbors=30 and per-gene moments computed on total (unspliced+spliced) mRNA counts
- The AnnData contains per-gene TF annotations restricted to ENCODE and ChEA regulatory pairs
- The highly variable gene set consists of the top-2000 genes after pp.filter_and_normalize
- Reproduction check: running 'python TFvelo_run_demo.py --dataset_name pancreas' stores results in 'TFvelo_pancreas/rc.h5ad'

## Limitations

- TFvelo is slower than scVelo baselines, which could be a weakness when applied to large-scale datasets; performing TFvelo with more CPUs in parallel (via --n_jobs) could reduce running time
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so filtering to top-2000 HVGs does not guarantee good dynamics fitting for all retained genes
- No numeric acceptance threshold for the preprocessing quality is reported in the paper; validation is a structural check only

## Evidence

- [methods] Apply scvelo pp.filter_and_normalize to filter and normalize the count matrix, retaining the top-2000 highly variable genes.: [evidence span withheld — anchor status: not_found]
- [methods] Build the neighbor graph with scvelo pp.neighbors (n_neighbors=30) and compute scvelo pp.moments on the total (unspliced+spliced) mRNA counts: [evidence span withheld — anchor status: not_found]
- [methods] --TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default.: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] Please decompress the ENCODE TF-target database file firstly after downloading this package.: "Please decompress the ENCODE TF-target database file firstly after downloading this package. On linux, you can run: unzip ENCODE.zip"
