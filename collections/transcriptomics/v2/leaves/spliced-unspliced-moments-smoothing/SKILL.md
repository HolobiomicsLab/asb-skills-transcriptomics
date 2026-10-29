---
name: spliced-unspliced-moments-smoothing
description: Use when a raw scRNA-seq count matrix (h5ad) must be filtered, normalized,
  graph-smoothed, and paired with candidate TF-target sets (ENCODE/ChEA) prior to
  RNA velocity or TF-based dynamic model fitting.
when_to_use_negative:
- The input already contains spliced/unspliced counts suitable for classical splicing-based
  RNA velocity models (e.g., scVelo dynamical modeling), where TFvelo's TF-based smoothing
  is unnecessary
- The dataset lacks TF-target regulatory information compatible with ENCODE/ChEA,
  so candidate TF restriction cannot be performed
- A large-scale dataset where runtime matters — TFvelo is slower than scVelo in the
  same framework
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Pipeline owner; TFvelo_run_demo.py wraps preprocessing and runs the TF model
    on the moments-smoothed data
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Provides pp.filter_and_normalize (top-2000 HVGs), pp.neighbors (n_neighbors=30),
    and pp.moments for graph-based smoothing; TFvelo is built on the scvelo framework
  repo: theislab/scvelo
- name: anndata
  role: Loads and saves the raw and preprocessed count matrices as h5ad AnnData objects
- name: scanpy
  role: Supports filtering and normalization operations in the preprocessing environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:8742bab8c411e7fa8de55548dcc85b10f35fb7789d75ae7c025c73a4016221de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  - anndata
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# spliced-unspliced-moments-smoothing

## Summary

Preprocessing skill from the TFvelo pipeline: filter and normalize a raw scRNA-seq count AnnData with scvelo, build a k-NN neighbor graph, and smooth expression by computing moments over mRNA counts before transcription-factor-based velocity model fitting.

## When to use

Use when a raw scRNA-seq count matrix (h5ad) must be filtered, normalized, graph-smoothed, and paired with candidate TF-target sets (ENCODE/ChEA) prior to RNA velocity or TF-based dynamic model fitting.

## When NOT to use

- The input already contains spliced/unspliced counts suitable for classical splicing-based RNA velocity models (e.g., scVelo dynamical modeling), where TFvelo's TF-based smoothing is unnecessary
- The dataset lacks TF-target regulatory information compatible with ENCODE/ChEA, so candidate TF restriction cannot be performed
- A large-scale dataset where runtime matters — TFvelo is slower than scVelo in the same framework

## Inputs

- Raw scRNA-seq count AnnData (.h5ad file)
- Pancreas dataset (auto-downloadable via TFvelo_run_demo.py --dataset_name pancreas)
- ENCODE TF-target database file (ENCODE.zip, must be unzipped)
- ChEA TF-target database (default alongside ENCODE)

## Outputs

- Preprocessed AnnData (h5ad) containing filtered/normalized counts, neighbor graph, moments-smoothed layers, and per-gene TF annotations
- h5ad files stored in the folder 'TFvelo_'+dataset_name+'_demo'

## How to apply

Load the raw AnnData with anndata read_h5ad (or auto-download the pancreas dataset via TFvelo_run_demo.py). Apply scvelo pp.filter_and_normalize to retain the top-2000 highly variable genes, then build the neighbor graph with scvelo pp.neighbors using n_neighbors=30. Compute scvelo pp.moments on the mRNA counts over that graph to obtain smoothed values for downstream TF-model fitting. Restrict candidate TFs per target gene to regulatory pairs from the ENCODE and ChEA TF-target databases (the default --TF_databases; the ENCODE.zip file must be decompressed first). Save the preprocessed AnnData to the 'TFvelo_'+dataset_name+'_demo' folder so the TF model fitting step can consume it.

## Related tools

- **TFvelo** (Pipeline owner; TFvelo_run_demo.py wraps preprocessing and runs the TF model on the moments-smoothed data) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Provides pp.filter_and_normalize (top-2000 HVGs), pp.neighbors (n_neighbors=30), and pp.moments for graph-based smoothing; TFvelo is built on the scvelo framework) — theislab/scvelo
- **anndata** (Loads and saves the raw and preprocessed count matrices as h5ad AnnData objects)
- **scanpy** (Supports filtering and normalization operations in the preprocessing environment)

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The preprocessed h5ad file exists in the 'TFvelo_'+dataset_name+'_demo' folder and loads with anndata
- The saved AnnData contains the neighbor graph (n_neighbors=30) and per-gene TF annotations; no numeric acceptance threshold is reported, so this structural check is the qualitative criterion
- The normalized matrix retains the top-2000 highly variable genes after scvelo pp.filter_and_normalize
- ENCODE.zip has been decompressed (unzip ENCODE.zip) so ENCODE/ChEA TF pairs are available during preprocessing
- For the pancreas demo, the TF model output is stored at 'TFvelo_pancreas/rc.h5ad'

## Limitations

- TFvelo is slower than scVelo, which could be a weakness on large-scale datasets; running with more CPUs in parallel (--n_jobs) can reduce runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14)
- No numeric acceptance threshold is defined for the preprocessing step — validation is structural (file exists, loads, contains graph and TF annotations)

## Evidence

- [methods] Apply scvelo pp.filter_and_normalize to filter and normalize the count matrix, retaining the top-2000 highly variable genes.: [evidence span withheld — anchor status: not_found]
- [methods] Build the neighbor graph with scvelo pp.neighbors (n_neighbors=30) and compute scvelo pp.moments on the total (unspliced+spliced) mRNA counts over that graph.: [evidence span withheld — anchor status: not_found]
- [readme] Please decompress the ENCODE TF-target database file firstly after downloading this package. On linux, you can run: unzip ENCODE.zip: "Please decompress the ENCODE TF-target database file firstly after downloading this package."
- [readme] This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'.: "This will automatically download, preprocess and run TF model on pancrease dataset."
- [readme] --TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default.: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
