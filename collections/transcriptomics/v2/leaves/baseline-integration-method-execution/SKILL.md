---
name: baseline-integration-method-execution
description: Use when benchmarking a new single-cell diagonal-integration method against
  established baselines on paired-modality datasets (e.g. scRNA-scATAC 10X Multiome
  or CITE-seq scRNA-surface protein PBMC data) where identical inputs and metrics
  are required for a fair comparison.
when_to_use_negative:
- The modality pair has no cross-modality feature connections or is not one of the
  benchmarked pairings (e.g. electrophysiology vs gene expression), since anchor-
  and feature-based baselines were not evaluated in this setting
- The goal is paired-multimodal integration, which is out of scope for both scConfluence
  and these diagonal-integration baselines
- The dataset contains large modality-specific rare cell populations and a paired
  method like Seurat anchoring is used, since existing methods struggle with cell
  types absent in one modality
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Seurat v4.3.0
  role: R baseline performing anchor-based integration, finding anchor pairs between
    cells from different modalities
  repo: https://github.com/cantinilab/scconfluence
- name: Liger (rliger v1.0.0)
  role: R baseline for integrative dimensionality reduction compared against scConfluence
  repo: https://github.com/cantinilab/scconfluence
- name: MultiMAP
  role: Python baseline for multi-omics manifold integration included in the five-method
    benchmark
- name: Uniport
  role: Python optimal-transport baseline for unpaired multimodal integration included
    in the benchmark
- name: scGLUE
  role: Graph-linking deep-learning baseline included in the five-method benchmark
- name: scConfluence
  role: Proposed method whose benchmark performance against the five baselines is
    the target of this skill; combined uncoupled autoencoders with regularized Inverse
    Optimal Transport
  repo: https://github.com/cantinilab/scconfluence
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:b0b73e12a93b7874e3d83961941608dfaa76dac1c20ddcac5aac05e05385c961
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - Liger (rliger v1.0.0)
  - MultiMAP
  - Seurat
  - Seurat v4.3.0
  - Uniport
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# baseline-integration-method-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the five state-of-the-art single-cell multimodal integration baselines (Seurat, Liger, MultiMAP, Uniport, scGLUE) with their reference implementations and versions on the same diagonal-integration inputs as scConfluence, so that joint embeddings can be scored head-to-head on gold-standard PBMC 10X Multiome and CITE-seq benchmarks.

## When to use

Use when benchmarking a new single-cell diagonal-integration method against established baselines on paired-modality datasets (e.g. scRNA-scATAC 10X Multiome or CITE-seq scRNA-surface protein PBMC data) where identical inputs and metrics are required for a fair comparison.

## When NOT to use

- The modality pair has no cross-modality feature connections or is not one of the benchmarked pairings (e.g. electrophysiology vs gene expression), since anchor- and feature-based baselines were not evaluated in this setting
- The goal is paired-multimodal integration, which is out of scope for both scConfluence and these diagonal-integration baselines
- The dataset contains large modality-specific rare cell populations and a paired method like Seurat anchoring is used, since existing methods struggle with cell types absent in one modality

## Inputs

- Preprocessed MuData/AnnData objects with paired scRNA-scATAC profiles from 10X Multiome PBMC and Open problems Multiome datasets
- CITE-seq scRNA-surface protein paired profiles (OP Multiome, BMCITE datasets) with CLR-normalized protein modality
- R runtime with Seurat v4.3.0 and rliger v1.0.0 installed; Python environment with MultiMAP, Uniport, and scGLUE reference implementations

## Outputs

- Joint cell embeddings (latent representations) per method per dataset
- Aggregated benchmark_metrics.csv with columns for method, dataset, and metric value (purity, transfer accuracy, connectivity, FOSCTTM, cell type FOSCTTM)
- UMAP visualizations of joint embeddings per method and dataset reproducing the layout of Fig. 3b-e

## How to apply

Preprocess each gold-standard dataset identically for all methods: apply the paper's QC filtering, log-normalize scRNA to 10,000 counts plus log1p, TF-IDF normalize scATAC, and apply Centered Log Ratio to protein counts. Then run each baseline with its reference implementation: Seurat v4.3.0 (R) with anchor-based integration finding anchor pairs between cells from different modalities, Liger via rliger v1.0.0 (R), and MultiMAP, Uniport, and scGLUE with their reference packages. Feed each baseline the same modality-specific matrices used for scConfluence's uncoupled autoencoders (latent dimension 16, rIOT with unbalanced OT transported mass m=0.5) so that differences reflect the methods, not the inputs. Collect the joint embeddings from each method and compute purity, transfer accuracy, connectivity, FOSCTTM, and cell type FOSCTTM on each, then aggregate into a benchmark_metrics.csv keyed by method, dataset, and metric. Confirm the outcome against the expected finding that scConfluence outperforms all five baselines across the metrics.

## Related tools

- **Seurat v4.3.0** (R baseline performing anchor-based integration, finding anchor pairs between cells from different modalities) — https://github.com/cantinilab/scconfluence
- **Liger (rliger v1.0.0)** (R baseline for integrative dimensionality reduction compared against scConfluence) — https://github.com/cantinilab/scconfluence
- **MultiMAP** (Python baseline for multi-omics manifold integration included in the five-method benchmark)
- **Uniport** (Python optimal-transport baseline for unpaired multimodal integration included in the benchmark)
- **scGLUE** (Graph-linking deep-learning baseline included in the five-method benchmark)
- **scConfluence** (Proposed method whose benchmark performance against the five baselines is the target of this skill; combined uncoupled autoencoders with regularized Inverse Optimal Transport) — https://github.com/cantinilab/scconfluence

## Examples

```
git clone git@github.com:cantinilab/scconfluence.git; cd scconfluence && pip install scconfluence  # then run Seurat (R, v4.3.0), rliger v1.0.0, MultiMAP, Uniport, scGLUE on identical MuData inputs and score with purity, transfer accuracy, connectivity, FOSCTTM
```

## Evaluation signals

- All methods receive identical QC-filtered, modality-specifically normalized inputs (log1p to 10,000 for scRNA, TF-IDF for scATAC, CLR for protein) so metric differences are attributable to method
- benchmark_metrics.csv contains one row per method x dataset x metric combination with no missing values
- FOSCTTM and cell type FOSCTTM are computable because the gold-standard datasets are paired, allowing exact cell-level correspondence checks
- UMAP plots on Scanpy-computed neighbors of the joint embeddings reproduce the qualitative layout of Fig. 3b-e
- Expected direction of results: scConfluence outperforms all five baselines on the benchmark metrics, consistent with the reported finding

## Limitations

- All methods including the baselines struggle to properly deal with rare cell populations, which are under-represented in mini-batch gradient descent
- Anchor-based and feature-connection-based baselines rely on cross-modality feature links that are not always available, e.g. when integrating electrophysiology with gene expression
- No statistical method was used to predetermine sample size in the benchmark comparison

## Evidence

- [intro] Benchmark suite: "We extensively benchmarked scConfluence against five state-of-the-art methods: Seurat (v3.0), Liger, MultiMAP, Uniport, and scGLUE"
- [discussion] Seurat implementation detail: "In practice, we used the R package Seurat v4.3.0 which finds anchor pairs between cells from different modalities"
- [discussion] Liger implementation detail: "We compare scConfluence to Liger using the R package rliger v1.0.0."
- [abstract] Benchmark outcome: "We extensively benchmark scConfluence in several single-cell integration scenarios proving that it outperforms the state-of-the-art."
- [discussion] scConfluence configuration for identical inputs: "We used a latent dimension of 16 for all datasets but adapted the number of neurons in hidden layers to the dimensionality of the datasets"
