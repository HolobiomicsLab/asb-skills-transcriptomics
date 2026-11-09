---
name: multi-method-integration-benchmark-comparison
description: Use when you have paired CITE-seq RNA gene counts and raw protein ADT
  counts with ground-truth cell-type labels and need to quantify whether a multimodal
  joint embedding improves clustering (ARI/NMI) over unimodal or competing integration
  methods.
when_to_use_negative:
- Input is a single-modality assay (scRNA-seq or scATAC-seq only) with no paired protein
  ADT counts — the benchmark targets dual-modality CITE-seq data
- There are no ground-truth cell-type labels available, since ARI/NMI cannot be computed
  against a reference clustering
- The goal is a single-method deployment rather than a head-to-head comparison — training
  four competing integrators over 20 replicates is unnecessary overhead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scME
  role: Dual-modality factor model producing the joint RNA+protein cell embedding
    under evaluation
  repo: https://github.com/bucky527/scME
- name: scvi-tools
  role: Provides the totalVI baseline integration method trained on the same four
    datasets
- name: scanpy
  role: Performs Leiden clustering on every joint and unimodal embedding
- name: scikit-learn
  role: Computes ARI and NMI between cluster assignments and ground-truth cell-type
    labels
- name: PyTorch
  role: Deep-learning backend used to build and train scME and the vanilla autoencoder
    baseline
- name: pyro-ppl
  role: Probabilistic programming dependency for scME's deep factor model
- name: anndata
  role: Dependency for handling single-cell data objects in the benchmark pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:702e4a3d5aa65425534fc3e78e7b7a908074e8c29979cb47f64a1ee2dd7cd101
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bucky527/scME
  tools:
  - PyTorch
  - anndata
  - numpy
  - pandas
  - pyro-ppl
  - scME
  - scanpy
  - scikit-learn
  - scvi-tools
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# multi-method-integration-benchmark-comparison

## Summary

A reproducible benchmarking protocol that compares the scME dual-modality factor model against unimodal RNA/protein embeddings and competing multimodal integrators (totalVI, Seurat V4, MOFA+, vanilla autoencoder) by Leiden-clustering the joint embeddings and scoring ARI/NMI against ground-truth cell-type labels across four CITE-seq datasets over 20 replicates.

## When to use

Use when you have paired CITE-seq RNA gene counts and raw protein ADT counts with ground-truth cell-type labels and need to quantify whether a multimodal joint embedding improves clustering (ARI/NMI) over unimodal or competing integration methods.

## When NOT to use

- Input is a single-modality assay (scRNA-seq or scATAC-seq only) with no paired protein ADT counts — the benchmark targets dual-modality CITE-seq data
- There are no ground-truth cell-type labels available, since ARI/NMI cannot be computed against a reference clustering
- The goal is a single-method deployment rather than a head-to-head comparison — training four competing integrators over 20 replicates is unnecessary overhead

## Inputs

- Paired CITE-seq RNA gene counts matrix (CSV, rows=cells, columns=genes, column 0 = cell IDs) for CBMC, BMNC, BM2, and SLN datasets from GEO accessions
- Raw protein ADT counts matrix (CSV, rows=cells, columns=ADTs, column 0 = cell IDs) paired cell-for-cell with the RNA matrix
- Ground-truth cell-type label vector per dataset for ARI/NMI computation

## Outputs

- Joint cell embeddings (zm) from scME.inference and equivalent joint embeddings from totalVI, Seurat V4, MOFA+, and the vanilla autoencoder
- Leiden cluster assignments per embedding (multimodal, RNA-only, protein-only) from scanpy
- Aggregated replicate-level ARI and NMI comparison table saved as clustering_metrics.csv (per method x dataset, over 20 replicates)

## How to apply

Format each CITE-seq dataset (e.g., CBMC, BMNC, BM2, SLN from their GEO accessions) as paired CSV matrices — cells as rows, genes/ADTs as columns, cell IDs in column 0. Train scME via build_scme (protein_dist='NB' or 'MNB', rna_latent_dim=24, protein_latent_dim=20, latent_dim=32) and train_model (max_epochs=200, batchsize=256, lr=1e-4, lr_cla=1e-4, milestones=[80]), then extract joint cell embeddings with scme.inference(rna_data, protein_data). Train the baselines on identical inputs: totalVI via scvi-tools, Seurat V4, MOFA+, and a vanilla autoencoder, plus RNA-only and protein-only unimodal embeddings. Run Leiden clustering with scanpy on every embedding, compute ARI and NMI with scikit-learn against ground-truth labels, and repeat the full embed-plus-cluster procedure over 20 replicates to control for stochasticity before aggregating per-method, per-dataset scores into clustering_metrics.csv. A run is judged correct when the aggregate table reproduces the expected ordering — e.g., CBMC ARI rising from 0.5829 (RNA) / 0.5142 (protein) to 0.8742 with scME — and all methods saw the same formatted input matrices.

## Related tools

- **scME** (Dual-modality factor model producing the joint RNA+protein cell embedding under evaluation) — https://github.com/bucky527/scME
- **scvi-tools** (Provides the totalVI baseline integration method trained on the same four datasets)
- **scanpy** (Performs Leiden clustering on every joint and unimodal embedding)
- **scikit-learn** (Computes ARI and NMI between cluster assignments and ground-truth cell-type labels)
- **PyTorch** (Deep-learning backend used to build and train scME and the vanilla autoencoder baseline)
- **pyro-ppl** (Probabilistic programming dependency for scME's deep factor model)
- **anndata** (Dependency for handling single-cell data objects in the benchmark pipeline)

## Examples

```
python scme.py --rna cbmc_rna.csv --protein cbmc_adt.csv --output-dir results/cbme_cbmc --batch-size 256
```

## Evaluation signals

- ARI/NMI values aggregated over 20 independent replicates should be reported per method per dataset, not as single-run point estimates
- CBMC benchmark expectation: scME joint embedding ARI ≈ 0.8742 versus RNA-only 0.5829 and protein-only 0.5142
- All methods must be trained on identically formatted paired CSV matrices (cells as rows, cell IDs in column 0) to make the comparison fair
- Leiden clustering must be run consistently (scanpy) across every embedding — multimodal and unimodal alike — before metric computation
- Output table clustering_metrics.csv should contain one ARI and one NMI score per method x dataset combination

## Limitations

- Benchmark restricted to four CITE-seq datasets (CBMC, BMNC, BM2, SLN); generalization to other tissues, technologies, or modality pairs (e.g., ATAC+RNA) is not demonstrated
- Results depend on replicate-level stochasticity; single-run comparisons are unreliable, and the protocol requires 20 replicates of the full embedding-plus-clustering procedure
- Most existing single-cell multiomics integration methods discard modality-specific complementary information, so baselines may underperform for structural rather than quality reasons

## Evidence

- [other] scME's multimodal embedding yielded more accurate Leiden clustering than unimodal embeddings and competing methods (e.g., CBMC ARI raised from 0.5829 RNA / 0.5142 protein to 0.8742): [evidence span withheld — anchor status: not_found]
- [abstract] the joint representation of multiple modalities yielded by scME can provide salient information to improve both single-cell clustering and cell-type classification: "the joint representation of multiple modalities yielded by scME can provide salient information to improve both single-cell clustering and cell-type classification"
- [readme] scME accepts as input the RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format: "scME accepts as input the  RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format"
- [methods] zm=scme.inference(rna_data,protein_data): "#Inference cell embedding

zm=scme.inference(rna_data,protein_data)"
