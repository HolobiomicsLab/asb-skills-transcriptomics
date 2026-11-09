---
name: clustering-method-benchmarking
description: Use when a single-cell gene expression matrix must be clustered and compared
  across multiple clustering methods, or when reproducing scAce's ARI/NMI benchmark
  (Fig. 2A simulated, Fig. 3A real).
when_to_use_negative:
- The task is a single method's clustering of one dataset without comparison to alternatives
  — benchmarking scaffolding is unnecessary
- Ground-truth cell-type labels are absent, so ARI/NMI cannot be computed against
  annotations
- Input is not a single-cell gene expression matrix (scAce takes scRNA-seq counts
  as its input for VAE training)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: scAce
  role: 'Primary method under evaluation: VAE-based adaptive embedding and clustering
    run via run_scace'
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: AnnData construction and standardized preprocessing (filtering, normalize_per_cell,
    log1p, scale)
- name: Leiden
  role: De novo cluster initialization to obtain initial cluster labels inside scAce
- name: Python 3.8
  role: Implementation environment for scAce and the benchmark scripts
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:843b2f7d02fba169085fd0c4c3dae7f529c2929951142edd2a060b370b87a657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# clustering-method-benchmarking

## Summary

Benchmark scAce against nine state-of-the-art scRNA-seq clustering methods (Seurat, CIDR, scDeepCluster, DESC, scVI, scGMAAE, SCCAF, ADClust, graph-sc) on simulated and seven annotated real datasets, scoring accuracy with ARI and NMI.

## When to use

Use when a single-cell gene expression matrix must be clustered and compared across multiple clustering methods, or when reproducing scAce's ARI/NMI benchmark (Fig. 2A simulated, Fig. 3A real).

## When NOT to use

- The task is a single method's clustering of one dataset without comparison to alternatives — benchmarking scaffolding is unnecessary
- Ground-truth cell-type labels are absent, so ARI/NMI cannot be computed against annotations
- Input is not a single-cell gene expression matrix (scAce takes scRNA-seq counts as its input for VAE training)

## Inputs

- Single-cell gene expression count matrix as an AnnData object (simulated 5-cell-type dataset or annotated scRNA-seq datasets from Supplementary Table S4)
- Ground-truth cell-type annotations for ARI/NMI computation
- True cell-type count for methods requiring a predetermined cluster number

## Outputs

- Cluster labels per method in adata.obs['scace_cluster'] (scAce) and equivalents for competing methods
- Cell embeddings in adata.obsm['scace_emb']
- ARI and NMI values per method per dataset and mean-ARI ranking table (e.g. ari_nmi_table.csv)

## How to apply

Preprocess each AnnData with scanpy exactly as the paper specifies: filter genes with min_cells=3, filter cells with min_genes=200, store raw counts, then sc.pp.normalize_per_cell, record the scale factor as n_counts/median(n_counts), log1p, and scale. Run scAce via run_scace without pre-specifying cluster number — it pre-trains a VAE (ZINB parameters: mean, dispersion, proportion of zero), initializes clusters de novo with Leiden or via clustering enhancement, and applies adaptive cluster merging to output labels in adata.obs['scace_cluster'] and embeddings in adata.obsm['scace_emb']. Run the nine competing methods on the same preprocessed data, passing the true cell-type count (e.g. 5 for the simulated dataset) only to methods that require a preset cluster number. Compute ARI and NMI of each method's predicted labels against ground-truth cell-type annotations, record the predicted cluster number, assemble a metrics table (e.g. ari_nmi_table.csv), and rank methods by mean ARI across the seven real datasets (human pancreas GSE84133, mouse ES GSE65525, human PBMC, human kidney, mouse hypothalamus, mouse kidney, turtle brain).

## Related tools

- **scAce** (Primary method under evaluation: VAE-based adaptive embedding and clustering run via run_scace) — https://github.com/sldyns/scAce
- **scanpy** (AnnData construction and standardized preprocessing (filtering, normalize_per_cell, log1p, scale))
- **Leiden** (De novo cluster initialization to obtain initial cluster labels inside scAce)
- **Python 3.8** (Implementation environment for scAce and the benchmark scripts)

## Examples

```
from scace import run_scace; adata = run_scace(adata)
```

## Evaluation signals

- On the simulated five-cell-type dataset, scAce is the only method achieving ARI = 1 together with the correct predicted cluster number
- On the seven real datasets, the mean-ARI ranking reproduces the order scAce, DESC, graph-sc, scDeepCluster, ADClust, with scAce highest
- Each method's row is present for all seven datasets and each ARI/NMI is computed against provided cell-type annotations
- scAce outputs exist at the expected locations: adata.obs['scace_cluster'] and adata.obsm['scace_emb']
- The preprocessing chain matches the paper: filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell with scale_factor = n_counts/median(n_counts), log1p, scale

## Limitations

- Competing methods require the true cluster number to be passed, whereas scAce does not estimate the number of clusters in advance — an asymmetric setup that must be documented
- scAce depends on preprocessed count data (normalize_per_cell, log1p, scale) and records scale factors in adata.obs['scale_factor']; skipping this chain can break the ZINB-based VAE
- Cluster initialization has two modes (de novo via Leiden vs. clustering enhancement on prior results); results may differ by mode

## Evidence

- [readme] preprocessing chain: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] output locations: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
- [abstract] three-step method: "constructs a variational autoencoder to simultaneously learn cell embeddings and cluster assignments"
- [abstract] adaptive merging rationale: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [methods] de novo initialization: "With *de novo* initialization, Leiden is used to obtain initial cluster labels"
- [results] benchmark finding: "we demonstrate that scAce outperforms state-of-the-art clustering methods for scRNA-seq data, and achieves better clustering accuracy and robustness"
