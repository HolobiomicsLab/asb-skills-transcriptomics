---
name: time-lag-edge-direction-sign-estimation
description: Use when a time-series RNA-Seq count table has been discretized and pairwise
  mutual information computed for candidate edges, and undirected edges must be oriented
  TF->TG with a regulation sign.
when_to_use_negative:
- Input expression data are not time series (no adjacent time-point pairs exist, so
  MSD-based time lag cannot be estimated)
- You only need undirected co-regulation scores — plain mutual information tools such
  as ARACNE or minet suffice without the lag/orientation step
- You need a fully refined network with TF binding motifs and curated database interactions
  — this skill only produces the initial, MI-based GRN before refinement
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Augusta
  role: Python package implementing MI-based initial GRN inference with MSD time-lag
    edge orientation and sign assignment
  repo: github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:9ec5976b1cc086d0b89c6a4ad218086e7d797ea9a718745d0ae003c4b3ee6220
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  tools:
  - Augusta
  - Python
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Time-lag edge direction and sign estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Orients candidate TF-target edges in an initial GRN inferred by mutual information (as in Augusta) and assigns positive/negative regulation signs using the transcriptional time lag, identified via the most significant difference (MSD) among adjacent pairs of time points in the expression time series.

## When to use

Use when a time-series RNA-Seq count table has been discretized and pairwise mutual information computed for candidate edges, and undirected edges must be oriented TF->TG with a regulation sign.

## When NOT to use

- Input expression data are not time series (no adjacent time-point pairs exist, so MSD-based time lag cannot be estimated)
- You only need undirected co-regulation scores — plain mutual information tools such as ARACNE or minet suffice without the lag/orientation step
- You need a fully refined network with TF binding motifs and curated database interactions — this skill only produces the initial, MI-based GRN before refinement

## Inputs

- Discretized time-series RNA-Seq count table (genes x time points, normalized e.g. by RPKM/TPM/CPM, binned with D = min(n/5, 10) bins)
- Candidate edge list from pairwise mutual information computation (every gene pair)

## Outputs

- Oriented and signed initial GRN adjacency matrix (.csv, e.g. initial_grn_adjacency_matrix.csv) with TF->TG direction and positive/negative sign per edge

## How to apply

Starting from the MI-based candidate edges over the discretized count table (number of bins D = min(n/5, 10) for n genes, natural logarithm in nats), scan all adjacent pairs of time points in the measured series to locate the most significant difference (MSD) for each candidate gene pair (Augusta Eq. 5). The position of the MSD gives the transcriptional time lag: the gene whose expression shift precedes the other's is assigned as the regulator (TF) and the lagging gene as the target (TG), thereby orienting the edge. The sign of the expression change across the lag (co-increase vs. opposite direction) determines whether the regulation is annotated as positive or negative. Export the oriented, signed network as an adjacency matrix (e.g., initial_grn_adjacency_matrix.csv) that feeds Augusta's subsequent refinement and Boolean Network steps.

## Related tools

- **Augusta** (Python package implementing MI-based initial GRN inference with MSD time-lag edge orientation and sign assignment) — github.com/JanaMus/Augusta

## Examples

```
import Augusta; Augusta.RNASeq_to_GRN(count_table_input='count_table.csv', promoter_length=100, genbank_file_input='genome.gb', normalization_type='RPKM', motifs_max_time=60)
```

## Evaluation signals

- Every candidate edge in the output adjacency matrix is directed (TF->TG) and carries an explicit positive or negative sign — no undirected or unsigned entries remain
- Orientation matches the MSD-derived transcriptional time lag: the regulator's expression shift occurs at an earlier adjacent time-point pair than the target's
- Benchmarked runtime on progressively scaled inputs fits the claimed O(m*n^2*D^2) complexity (m = time points, n = genes, D = bins)
- MI-only (no refinement) results on DREAM 4 are in the reported ballpark: sensitivity ~0.38, specificity ~0.54, accuracy ~0.54 in ~83.07 s

## Limitations

- Only m on the order of units to tens of time points is expected in the input count table, limiting lag estimation resolution
- Inferring autoregulations remains a significant challenge
- Inferred networks are approximations, not perfectly accurate representations of biological system regulation; multiple RNA-Seq experiments under different conditions cannot yet be processed in a single computation

## Evidence

- [readme] Augusta invocation for GRN inference from RNA-Seq: [evidence span withheld — anchor status: external_doc]
