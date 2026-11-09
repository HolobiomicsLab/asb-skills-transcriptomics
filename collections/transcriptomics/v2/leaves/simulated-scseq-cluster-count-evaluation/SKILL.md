---
name: simulated-scseq-cluster-count-evaluation
description: Use when you need to benchmark single-cell clustering methods on data
  with known ground-truth cell-type labels, especially to test whether a method recovers
  the correct number of clusters without a predetermined K.
when_to_use_negative:
- Input is real data without ground-truth cell-type labels — ARI/NMI against a reference
  cannot be computed and the benchmark outcome (scAce achieving ARI = 1) is not checkable
- The analysis goal is a single method's application rather than a comparative benchmark
  across ten clustering methods
- You already know and want to fix the cluster number for scAce itself — its adaptive
  cluster merging is designed to avoid estimating the number of clusters in advance
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: Method under test; run via run_scace without pre-specifying cluster number
    to obtain cluster labels and cell embeddings
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: 'Preprocessing framework: AnnData construction, gene/cell filtering, per-cell
    normalization, log1p, and scaling'
- name: Python 3.8
  role: Implementation environment for the scAce package and the benchmark scripts
- name: Leiden
  role: Used inside scAce's de novo cluster initialization to obtain initial cluster
    labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:6dd0ca1815e4fac4f84c4540a2e0c00f9c5f3354efccbbb27bce54ca690e20a1
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# simulated-scseq-cluster-count-evaluation

## Summary

Benchmark protocol for evaluating scRNA-seq clustering methods on a simulated five-cell-type dataset, comparing ten methods (scAce plus Seurat, CIDR, scDeepCluster, DESC, scVI, scGMAAE, SCCAF, ADClust, graph-sc) by ARI, NMI, and whether the true cluster number is recovered without being pre-specified.

## When to use

Use when you need to benchmark single-cell clustering methods on data with known ground-truth cell-type labels, especially to test whether a method recovers the correct number of clusters without a predetermined K.

## When NOT to use

- Input is real data without ground-truth cell-type labels — ARI/NMI against a reference cannot be computed and the benchmark outcome (scAce achieving ARI = 1) is not checkable
- The analysis goal is a single method's application rather than a comparative benchmark across ten clustering methods
- You already know and want to fix the cluster number for scAce itself — its adaptive cluster merging is designed to avoid estimating the number of clusters in advance

## Inputs

- Simulated single-cell gene expression matrix (five cell types) loadable into a scanpy AnnData object
- Ground-truth cell-type label vector for the simulated cells
- True number of cell types (5) to pass to methods that require a predetermined cluster number

## Outputs

- Cluster labels per method (e.g. adata.obs['scace_cluster'] for scAce)
- Cell embeddings per method (e.g. adata.obsm['scace_emb'] for scAce)
- Predicted cluster number for each of the ten methods
- ARI and NMI metrics per method, assembled into ari_nmi_table.csv

## How to apply

Load the simulated gene expression matrix into a scanpy AnnData object, then preprocess exactly as scAce prescribes: filter genes with min_cells=3, filter cells with min_genes=200, store adata.raw, normalize per cell, record the scale factor as n_counts/median(n_counts), then log1p-transform and scale. Run scAce via run_scace without pre-specifying the cluster number, and run the nine comparison methods on the same preprocessed data, passing the true cell-type count as K only for methods that require it. Compute ARI and NMI against the ground-truth labels and record each method's predicted cluster number. Assemble the metrics into a single results table (ari_nmi_table.csv) and verify against the reported benchmark outcome that scAce is the only method among the ten achieving ARI = 1 together with the correct cluster number.

## Related tools

- **scAce** (Method under test; run via run_scace without pre-specifying cluster number to obtain cluster labels and cell embeddings) — https://github.com/sldyns/scAce
- **scanpy** (Preprocessing framework: AnnData construction, gene/cell filtering, per-cell normalization, log1p, and scaling)
- **Python 3.8** (Implementation environment for the scAce package and the benchmark scripts)
- **Leiden** (Used inside scAce's de novo cluster initialization to obtain initial cluster labels)

## Examples

```
import scanpy as sc; adata = sc.AnnData(data); sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200); sc.pp.normalize_per_cell(adata); sc.pp.log1p(adata); sc.pp.scale(adata); from scace import run_scace; adata = run_scace(adata)
```

## Evaluation signals

- scAce reports ARI = 1 on the simulated five-cell-type dataset — the only method among the ten to do so
- scAce recovers the correct cluster number (5) without the true count being passed to run_scace
- All ten methods are evaluated on the identical preprocessed AnnData (same min_cells=3 / min_genes=200 filtering, normalization, log1p, scaling)
- Metrics table contains ARI, NMI, and predicted cluster number for each of the ten methods and is saved as ari_nmi_table.csv
- NMI values are consistent with ARI rankings across the ten methods

## Limitations

- Comparison methods that require a predetermined cluster number must be given the true cell-type count, giving them an advantage scAce deliberately does not use
- The headline result (scAce as the only method achieving ARI = 1) is specific to the simulated five-cell-type benchmark; performance on other datasets may differ
- scAce requires Python 3.8 and its preprocessing convention (normalize_per_cell, scale-factor recorded as n_counts/median(n_counts), log1p, scale) must be followed for comparable results

## Evidence

- [abstract] scAce uses an adaptive cluster merging approach that improves clustering without estimating the number of clusters in advance: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [results] Benchmark outcome on simulated data: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing filter thresholds: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] scAce invocation and outputs: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
- [readme] De novo initialization via Leiden: "With *de novo* initialization, Leiden is used to obtain initial cluster labels"
