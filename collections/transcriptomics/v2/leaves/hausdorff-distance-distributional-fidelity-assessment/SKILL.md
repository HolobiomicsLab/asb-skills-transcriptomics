---
name: hausdorff-distance-distributional-fidelity-assessment
description: Use when benchmarking single-cell sketching methods and you need a distribution-fidelity
  metric comparing a subsample's feature distribution (e.g. PCA embeddings in adata.obsm['X_pca'])
  against the full dataset.
when_to_use_negative:
- The goal is cell-type balance assessment rather than distributional fidelity — the
  Gini coefficient over cell-type proportions is the correct metric there, not Hausdorff
  distance.
- The input is already a reduced subsample and no full-data reference distribution
  exists, since the Hausdorff distance requires comparing sketch and full-data feature
  distributions.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: Generates the 10% sketch whose distributional fidelity is measured by Hausdorff
    distance (sketch_size=0.1, use_rep='X_pca', strategy='FB', seed=42)
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Preprocessing to obtain the log-normalized data, top 3,000 HVGs and 50 PCs
    in adata.obsm['X_pca'] used for the distance computation
- name: scikit-learn 1.5.2
  role: Numerical/scientific dependency used in the scValue implementation within
    which this metric benchmarking was executed
- name: GeoSketch
  role: Comparator subsampling method whose sketches are scored with Hausdorff distance
- name: Sphetcher
  role: Comparator subsampling method whose sketches are scored with Hausdorff distance
- name: Hopper
  role: Comparator subsampling method whose sketches are scored with Hausdorff distance
- name: scSampler
  role: Comparator subsampling method whose sketches are scored with Hausdorff distance
- name: Seurat
  role: Uniform subsampling comparator pipeline whose sketches are scored with Hausdorff
    distance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:9b0ad54339b40c7bd81b44fba3abb64d4c8eaa218fde39866a51aaa8def61ba7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - GeoSketch
  - Hopper
  - KH
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  - scikit-learn 1.5.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hausdorff-distance-distributional-fidelity-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmark metric in the scValue subsampling comparison: Hausdorff distance quantifies how faithfully a 10% cell sketch preserves the full-data feature distribution, computed for scValue and six competitor methods across 16 public scRNA-seq datasets in .h5ad format.

## When to use

Use when benchmarking single-cell sketching methods and you need a distribution-fidelity metric comparing a subsample's feature distribution (e.g. PCA embeddings in adata.obsm['X_pca']) against the full dataset.

## When NOT to use

- The goal is cell-type balance assessment rather than distributional fidelity — the Gini coefficient over cell-type proportions is the correct metric there, not Hausdorff distance.
- The input is already a reduced subsample and no full-data reference distribution exists, since the Hausdorff distance requires comparing sketch and full-data feature distributions.

## Inputs

- Preprocessed scRNA-seq AnnData (.h5ad) with log-normalized expression, top 3,000 HVGs, and 50 PCs in adata.obsm['X_pca']
- A 10% cell sketch of each dataset produced by each subsampling method (scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler)
- The corresponding full (unsketched) dataset for reference distribution computation

## Outputs

- Per-dataset Hausdorff distance between sketch and full-data feature distributions
- Averaged rank summaries (mean ± SD) of Hausdorff distance across the 16 datasets (e.g. scValue 5.3±1.1)
- Per-dataset metrics plus rank table written to method_rankings_table4.csv

## How to apply

Preprocess each .h5ad dataset with Scanpy (log-normalized data, top 3,000 highly variable genes, 50 PCs stored in adata.obsm['X_pca']), then generate a 10% sketch of the full dataset with each subsampling method (scValue, Uniform, GeoSketch, Sphetcher, Hopper, KH, scSampler). For every sketch, compute the Hausdorff distance between the sketch's and the full data's feature distributions (e.g. over the X_pca representation) to quantify distributional fidelity — lower distance means the sketch better preserves the full data's structure. Rank the seven methods per dataset on the metric, average ranks across the 16 datasets, and compare against the reported scValue Hausdorff rank of 5.3±1.1 (scValue's distributional properties were 'akin to uniform sampling'). Record per-dataset metrics plus the rank table (e.g. method_rankings_table4.csv) so the metric is interpreted alongside computation-time and Gini-coefficient ranks.

## Related tools

- **scValue** (Generates the 10% sketch whose distributional fidelity is measured by Hausdorff distance (sketch_size=0.1, use_rep='X_pca', strategy='FB', seed=42)) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing to obtain the log-normalized data, top 3,000 HVGs and 50 PCs in adata.obsm['X_pca'] used for the distance computation)
- **scikit-learn 1.5.2** (Numerical/scientific dependency used in the scValue implementation within which this metric benchmarking was executed)
- **GeoSketch** (Comparator subsampling method whose sketches are scored with Hausdorff distance)
- **Sphetcher** (Comparator subsampling method whose sketches are scored with Hausdorff distance)
- **Hopper** (Comparator subsampling method whose sketches are scored with Hausdorff distance)
- **scSampler** (Comparator subsampling method whose sketches are scored with Hausdorff distance)
- **Seurat** (Uniform subsampling comparator pipeline whose sketches are scored with Hausdorff distance)

## Examples

```
wget -O mTC.h5ad "https://datasets.cellxgene.cziscience.com/6c253303-8255-4004-8464-5e90dd5846cb.h5ad"; python -c "import scanpy as sc; adata=sc.read_h5ad('mTC.h5ad'); sc.pp.highly_variable_genes(adata,n_top_genes=3000); sc.pp.pca(adata,n_comps=50)"
```

## Evaluation signals

- Reproduced scValue Hausdorff rank across 16 datasets should approximate the reported 5.3±1.1 in Table 4.
- Hausdorff distance is finite and non-negative for every method/dataset pair and decreases toward 0 as the sketch distribution approaches the full-data distribution.
- Per-dataset Hausdorff values for all seven methods are computed on the same X_pca representation so ranks are comparable.
- Rank table including time, Gini and Hausdorff ranks is written to method_rankings_table4.csv with one row per dataset per method.

## Limitations

- scValue's Hausdorff-distance rank (5.3±1.1) is its weakest of the three benchmark metrics — the method trades some distributional fidelity for speed and cell-type balance.
- Tool versions for the comparator sketching libraries (KH, Sphetcher, Hopper, scSampler, GeoSketch) are not specified, which complicates exact reproduction of the benchmark.
- No accession numbers or repository identifiers are provided for the 16 benchmark datasets, so the benchmark depends on external dataset retrieval.

## Evidence
