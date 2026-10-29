---
name: multimodal-cell-embedding-integration
description: Use when paired single-cell multimodal data (e.g., scRNA-seq + scATAC-seq
  from the same cells, such as a 10x Multiome sample) must be unified into per-cell
  joint embeddings before trajectory or DAG construction.
when_to_use_negative:
- Modalities were not assayed on the same cells simultaneously (e.g., independent
  scRNA-seq and scATAC-seq cohorts) — Schema is designed for simultaneously assayed
  multimodal data.
- Cells are not part of a dynamic process (e.g., differentiation) — the paper notes
  reliance on data from tissue undergoing a dynamic process to provide pseudotemporal
  context.
- The goal is direct peak-gene Granger testing with the wrapped pipeline; gridnet.gridnet_multimodal
  already performs this integration step internally.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: Schema
  role: Learns the joint per-cell feature embedding that unifies RNA and ATAC modalities,
    used as the coordinate space for kNN graph and pseudotime inference
  repo: https://github.com/rs239/schema
- name: GrID-Net
  role: Provides schema_representations wrapper (and preprocess_multimodal) through
    which Schema embeddings are computed before DAG construction
  repo: https://github.com/alexw16/gridnet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:638227992520021b2e556341b751155de21f47d921ece10e333f6c5d1435a817
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/rs239/schema
  tools:
  - GrID-Net
  - Schema
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# multimodal-cell-embedding-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Learn a joint cell representation that unifies paired RNA-seq and ATAC-seq modalities using Schema, so that a k-nearest-neighbor graph built from Euclidean distances between these embeddings can serve as the scaffold for downstream pseudotime-oriented cell-state DAG construction.

## When to use

Use when paired single-cell multimodal data (e.g., scRNA-seq + scATAC-seq from the same cells, such as a 10x Multiome sample) must be unified into per-cell joint embeddings before trajectory or DAG construction.

## When NOT to use

- Modalities were not assayed on the same cells simultaneously (e.g., independent scRNA-seq and scATAC-seq cohorts) — Schema is designed for simultaneously assayed multimodal data.
- Cells are not part of a dynamic process (e.g., differentiation) — the paper notes reliance on data from tissue undergoing a dynamic process to provide pseudotemporal context.
- The goal is direct peak-gene Granger testing with the wrapped pipeline; gridnet.gridnet_multimodal already performs this integration step internally.

## Inputs

- Preprocessed AnnData object of RNA-seq counts (normalized, log(1+x) transformed), with gene TSS annotations in adata.var (chr_no, txstart)
- Preprocessed AnnData object of ATAC-seq counts (normalized, log(1+x) transformed), with peak start/end annotations in adata.var (chr_no, start, end)
- Paired cells assayed simultaneously across both modalities (e.g., 5,000-cell 10x Multiome sample)

## Outputs

- X_joint: joint multimodal cell embedding matrix (N x d, one row per cell) suitable for kNN graph construction

## How to apply

First preprocess the paired RNA-seq and ATAC-seq count matrices with gridnet.preprocess_multimodal (count normalization and log(1+x) transformation). Then call gridnet.schema_representations(rna_adata, atac_adata) to learn, via Schema's metric-learning / quadratic-programming formulation, a joint embedding X_joint for each cell that unifies information from both modalities. These embeddings are the coordinate space in which a kNN graph (k=15, Euclidean distances) is built and pseudotime (e.g., scanpy diffusion pseudotime, 'dpt') is inferred, so the quality of the modality integration directly determines DAG edge orientation. Schema requires the modalities to be assayed for the same cells simultaneously, matching the 10x Multiome design. The rationale, per the paper, is that edges are oriented in accordance with pseudotime computed after multimodal integration with Schema, enabling Granger causal inference on the resulting DAG.

## Related tools

- **Schema** (Learns the joint per-cell feature embedding that unifies RNA and ATAC modalities, used as the coordinate space for kNN graph and pseudotime inference) — https://github.com/rs239/schema
- **GrID-Net** (Provides schema_representations wrapper (and preprocess_multimodal) through which Schema embeddings are computed before DAG construction) — https://github.com/alexw16/gridnet

## Examples

```
X_joint = gridnet.schema_representations(rna_adata, atac_adata); dag = gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt')
```

## Evaluation signals

- X_joint has one row per cell of the paired dataset (e.g., N=5000 rows) with a consistent joint embedding dimensionality.
- A kNN graph built from X_joint (k=15, Euclidean) can be oriented by pseudotime ('dpt') and topological-sort confirms the resulting graph is acyclic.
- Embeddings show coherent mixing/unification of RNA and ATAC modalities rather than modality-stratified clusters.
- Downstream gridnet.construct_dag accepts X_joint and the supplied root cell index iroot without error.

## Limitations

- Schema requires all modalities to be assayed for the same cells simultaneously; it is not for cross-cohort integration.
- Downstream causal inference depends on pseudotemporal context, so the tissue should be undergoing a dynamic process such as differentiation; static tissue snapshots give weak edge orientation.
- Large single-cell datasets can impose computational burden; Schema itself avoids GPU need, but the scale of cells may still stress downstream steps.

## Evidence

- [readme] Schema README scope: [evidence span withheld — anchor status: external_doc]
- [discussion] Paper discussion on kNN construction: [evidence span withheld — anchor status: not_found]
