---
name: multiomic-baseline-method-reproduction
description: Use when validating a causal peak-gene inference method on RNA+ATAC single-cell
  multimodal data and need a fair, pair-level AUROC comparison against Pearson/pseudocell
  correlation, linear Granger, GVAR, and ABC baselines.
when_to_use_negative:
- The data are bulk or single-modality only — GrID-Net and all baselines here require
  matched scRNA-seq + scATAC-seq from the same cells and a pseudotemporally dynamic
  process for DAG construction
- You need de novo locus-gene discovery rather than a benchmarked comparison — this
  skill is a controlled evaluation harness, not a discovery workflow
- The tissue is static (no differentiation/disease trajectory), since DAG edge orientation
  depends on diffusion pseudotime through a dynamic biological process
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GrID-Net
  role: 'Proposed method under benchmark: GNN-based Granger causal inference on the
    DAG of cells, scoring candidate peak-gene pairs (L=10 layers, Adam, lr 0.001,
    batch 1024)'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations used to build the kNN graph and
    diffusion pseudotime for the shared DAG used by all methods
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation framework for the GrID-Net GNN models
- name: scanpy
  role: Provides the diffusion pseudotime implementation used to orient kNN graph
    edges into a DAG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:f75e43a0f19a3bd9d96c9c08fdda586090b7149733e6aae0b945365a8cb0368d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - PyTorch
  - Schema
  - scanpy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multiomic-baseline-method-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a proposed noncoding locus-gene inference method (GrID-Net) head-to-head against correlation-based, linear Granger, GVAR, and ABC baselines on matched single-cell multimodal datasets, scoring all methods by AUROC against eQTL-supported peak-gene pairs.

## When to use

Use when validating a causal peak-gene inference method on RNA+ATAC single-cell multimodal data and need a fair, pair-level AUROC comparison against Pearson/pseudocell correlation, linear Granger, GVAR, and ABC baselines.

## When NOT to use

- The data are bulk or single-modality only — GrID-Net and all baselines here require matched scRNA-seq + scATAC-seq from the same cells and a pseudotemporally dynamic process for DAG construction
- You need de novo locus-gene discovery rather than a benchmarked comparison — this skill is a controlled evaluation harness, not a discovery workflow
- The tissue is static (no differentiation/disease trajectory), since DAG edge orientation depends on diffusion pseudotime through a dynamic biological process

## Inputs

- Unprocessed paired RNA-seq and ATAC-seq AnnData objects for 5 samples from each of 10x Multiome, SHARE-seq, SNARE-seq, and sci-CAR datasets, with chr_no/txstart annotations in adata.var for genes and chr_no/start/end for peaks
- Ground-truth peak-gene pairs overlapping significant eQTLs (P < 10^-10) from the eQTL Catalogue
- Root cell index or marker gene for pseudotime inference

## Outputs

- Per-method, per-sample AUROC values for GrID-Net and the five baselines against eQTL-supported pairs
- Consolidated benchmark table (per-method AUROC mean ± SD across 5 samples per dataset) written to eqtl_benchmark_auroc.csv
- Welch one-tailed t-test results comparing GrID-Net to the next best method per dataset

## How to apply

Preprocess each multimodal dataset (10x Multiome, SHARE-seq, SNARE-seq, sci-CAR) with gridnet.preprocess_multimodal (count normalization and log(1+x)), filtering genes expressed in <1% of cells (rna_filter_gene_percent) and peaks accessible in <0.1% of cells (atac_filter_peak_percent). Assemble candidate peak-gene pairs per sample with gridnet.identify_all_peak_gene_link_candidates using distance_thresh=1e6 (250 kb for 10x Multiome). Learn joint RNA+ATAC cell representations with gridnet.schema_representations (Schema), build the kNN graph (n_neighbors=15) and orient edges by diffusion pseudotime (dpt) via gridnet.construct_dag to form the DAG of cells. Score the identical candidate pairs and DAG with gridnet.run_gridnet (GNN with L=10 layers, Adam optimizer, learning rate 0.001, mini-batch size 1024 candidate pairs) and compute comparable pair-level scores for Pearson correlation, pseudocell correlation, Linear Granger, GVAR, and ABC on the same inputs. Evaluate each method's AUROC against ground-truth pairs overlapping significant eQTLs (P<10^-10) from the eQTL Catalogue, aggregate mean ± SD across the 5 samples per dataset, and test the proposed method vs the next best with a Welch one-tailed t-test. The rationale is fairness: all methods see the same candidate set and the same DAG so differences reflect causal-modeling power, not preprocessing.

## Related tools

- **GrID-Net** (Proposed method under benchmark: GNN-based Granger causal inference on the DAG of cells, scoring candidate peak-gene pairs (L=10 layers, Adam, lr 0.001, batch 1024)) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations used to build the kNN graph and diffusion pseudotime for the shared DAG used by all methods) — https://github.com/rs239/schema
- **PyTorch** (Implementation framework for the GrID-Net GNN models)
- **scanpy** (Provides the diffusion pseudotime implementation used to orient kNN graph edges into a DAG)

## Examples

```
candidates_df = gridnet.identify_all_peak_gene_link_candidates(rna_adata,atac_adata,distance_thresh=1e6); dag = gridnet.construct_dag(gridnet.schema_representations(rna_adata,atac_adata),iroot,n_neighbors=15,pseudotime_algo='dpt'); results_df = gridnet.run_gridnet(X,Y,X_feature_names,Y_feature_names,candidate_XY_pairs,dag)
```

## Evaluation signals

- All methods are scored on the identical candidate peak-gene pair set and the identical DAG; any AUROC difference must come from scoring, not from differing candidate lists
- GrID-Net should attain 7-22% higher AUROC than the next best method in 3 of the 4 datasets with Welch one-tailed t-test P < 0.01, with GVAR on sci-CAR the only exception
- Per-dataset AUROC reported as mean ± SD across the 5 samples; check variance is finite and n=5 per dataset
- Ground-truth pairs are restricted to those overlapping significant eQTLs (P < 10^-10) from the eQTL Catalogue; verify the overlap step before computing AUROC
- Filtering invariants: genes below 1% expression and peaks below 0.1% accessibility are excluded; 10x Multiome candidate pairs restricted to 250 kb while other datasets use 1 Mb

## Limitations

- eQTL cross-referencing only mimics perturbation-based validation; inferred associations remain causal hypotheses needing perturbation follow-up
- Large single-cell datasets may exceed GPU memory capacity (GrID-Net was trained on a single NVIDIA Tesla V100); sketching with Geosketch or Hopper is recommended to retain heterogeneity
- The method requires multiomic data from matching tissue undergoing a dynamic process to supply pseudotemporal context, and tissue matching may be imperfect

## Evidence

- [results] GrID-Net was significantly more accurate than all other methods across all datasets in retrieving eQTL-supported peak-gene pairs: [evidence span withheld — anchor status: not_found]
- [discussion] GNN training hyperparameters: [evidence span withheld — anchor status: not_found]
- [discussion] DAG construction details: [evidence span withheld — anchor status: not_found]
