---
name: schema-joint-cell-embedding-integration
description: Use when you have paired single-cell multimodal data (same cells assayed
  by scRNA-seq and scATAC-seq) and need unified per-cell embeddings to construct a
  pseudotime-directed cell-state DAG for downstream causal analysis.
when_to_use_negative:
- Modalities come from different cells (not jointly assayed) — Schema is designed
  for the case where all modalities are assayed for the same cells simultaneously
- Data are static with no dynamic biological process (e.g., differentiation), since
  the DAG depends on pseudotemporal context that static tissues cannot provide
- Only a gene-by-cell expression feature table exists without a matching chromatin
  modality — Schema's multimodal synthesis has nothing to integrate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: Schema
  role: Learns the joint multimodal cell representations (metric-learning quadratic
    programming) that unify RNA and ATAC modalities and underpin the cell-state kNN
    graph and pseudotime DAG
  repo: https://github.com/rs239/schema
- name: GrID-Net
  role: Consumes the Schema-derived embeddings via gridnet.schema_representations
    and gridnet.construct_dag to build the DAG on which Granger causal peak-gene inference
    is run
  repo: https://github.com/alexw16/gridnet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:a9a223f3a51c108feaba853446795c63f1235a6c6178b4f749deaf62c2863c18
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# schema-joint-cell-embedding-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Use Schema to learn joint multimodal cell representations that unify paired RNA-seq and ATAC-seq modalities (e.g., 10x Multiome), serving as the basis for the kNN graph and diffusion pseudotime DAG required by GrID-Net causal inference.

## When to use

Use when you have paired single-cell multimodal data (same cells assayed by scRNA-seq and scATAC-seq) and need unified per-cell embeddings to construct a pseudotime-directed cell-state DAG for downstream causal analysis.

## When NOT to use

- Modalities come from different cells (not jointly assayed) — Schema is designed for the case where all modalities are assayed for the same cells simultaneously
- Data are static with no dynamic biological process (e.g., differentiation), since the DAG depends on pseudotemporal context that static tissues cannot provide
- Only a gene-by-cell expression feature table exists without a matching chromatin modality — Schema's multimodal synthesis has nothing to integrate

## Inputs

- Normalized RNA-seq AnnData object (log(1+x) counts, with gene TSS annotations chr_no/txstart in adata.var)
- Normalized ATAC-seq AnnData object (log(1+x) counts, with peak start/end annotations chr_no/start/end in adata.var)
- Root cell index or root-cell marker gene for pseudotime inference

## Outputs

- X_joint: N x d matrix of joint Schema cell embeddings integrating RNA and ATAC modalities
- dag_adjacency_matrix: N x N directed acyclic adjacency matrix with edges oriented by diffusion pseudotime (via gridnet.construct_dag, n_neighbors=15, pseudotime_algo='dpt')

## How to apply

Take the log(1+x)-normalized RNA and ATAC AnnData objects (e.g., preprocessed with gridnet.preprocess_multimodal) and call gridnet.schema_representations(rna_adata, atac_adata) to obtain a joint feature embedding X_joint for every cell. Schema formulates modality synthesis as a metric-learning quadratic programming problem and requires that all modalities were assayed for the same cells simultaneously, as in 10x Multiome or SHARE-seq data. Build a k-nearest-neighbor graph (k=15) on Euclidean distances between the Schema representations, infer pseudotime using scanpy's diffusion pseudotime ('dpt'), and orient kNN edges by increasing pseudotime to form the DAG passed to gridnet.construct_dag. Choose a root cell (e.g., via a marker gene such as TOP2A for progenitors) before diffusion pseudotime computation. For very large datasets, consider sketching cells with Geosketch or Hopper before embedding, since GrID-Net models may exceed GPU memory on large data.

## Related tools

- **Schema** (Learns the joint multimodal cell representations (metric-learning quadratic programming) that unify RNA and ATAC modalities and underpin the cell-state kNN graph and pseudotime DAG) — https://github.com/rs239/schema
- **GrID-Net** (Consumes the Schema-derived embeddings via gridnet.schema_representations and gridnet.construct_dag to build the DAG on which Granger causal peak-gene inference is run) — https://github.com/alexw16/gridnet

## Examples

```
X_joint = gridnet.schema_representations(rna_adata,atac_adata); dag = gridnet.construct_dag(X_joint,iroot,n_neighbors=15,pseudotime_algo='dpt')
```

## Evaluation signals

- X_joint has one row per cell, with rows of the RNA and ATAC AnnData objects in one-to-one correspondence (paired observations)
- gridnet.construct_dag on X_joint returns a valid acyclic adjacency matrix when n_neighbors=15 and pseudotime_algo='dpt'
- Resulting DAG orients kNN edges by increasing diffusion pseudotime; no edge points from higher to lower pseudotime
- Downstream gridnet.run_gridnet completes over the candidate peak-gene pairs, indicating the DAG and embeddings were well formed
- Embeddings reproduce expected cell-type geometry (e.g., progenitor-to-neuron ordering in the 10x Multiome corticogenesis dataset consistent with the root marker TOP2A)

## Limitations

- Large single-cell datasets impose computational burden that may exceed GPU memory capacity; sketching with Geosketch or Hopper is recommended to retain biological heterogeneity
- Schema integration quality depends on the tissue undergoing a dynamic process (e.g., differentiation) to supply meaningful pseudotemporal context
- The DAG edge orientation inherits errors from pseudotime inference, which propagates into downstream Granger causal scores

## Evidence

- [discussion] We constructed a k-nearest neighbor graph (k=15) based on the Euclidean distances between these cell representations and inferred pseudotime using the scanpy91 implementation of diffusion pseudotime92: [evidence span withheld — anchor status: not_found]
- [readme] Schema is a Python library for the synthesis and integration of heterogeneous single-cell modalities. **It is designed for the case where the modalities have all been assayed for the same cells simultaneously.**: [evidence span withheld — anchor status: external_doc]
