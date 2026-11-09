---
name: minimum-weight-matching-on-cell-graphs
description: Use when integrating exactly two single-cell RNA-seq batches (e.g. CEL-Seq2
  vs Smart-Seq2 count matrices) where you need batch correction that maintains cell-to-cell
  (local) structure, and MNN-based tools underperform on iLISI/50-RNN.
when_to_use_negative:
- You need to integrate more than two datasets — the current implementation of BATMAN
  works only with two datasets (multi-dataset integration is possible only 'in principle')
- Your data is not single-cell expression matrices (e.g. already-integrated embeddings,
  bulk RNA-seq, or feature tables)
- You require mutually-nearest-neighbor semantics or multi-dataset harmonization pipelines
  like Harmony/Scanorama by design
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BATMAN
  role: Implements the bipartite graph construction, minimum-weight matching (exact
    blossom-based or greedy ratio-2), local shifting vectors, and batch correction;
    run via batman.py
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Pre-processing of the input count matrices to log-counts before graph construction
    and matching
- name: Scanorama
  role: Alternative MNN-based integrator used as comparison baseline in the pancreas
    benchmark (iLISI 1.22)
- name: MNNcorrect
  role: Alternative MNN-based integrator used as comparison baseline in the pancreas
    benchmark (iLISI 1.18)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:581cc8b6667e12132bcd8f2eff456fbb29e3a1570595b2e7e80aa29a3feb96c0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# minimum-weight-matching-on-cell-graphs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Core algorithmic skill of BATMAN: build a weighted bipartite graph between two scRNA-seq batches (partitions = datasets, edge weights = Euclidean distance between cells in N-dimensional gene space) and solve a minimum-weight one-to-one matching to derive local shifting vectors for batch correction. It preserves local dataset structure while using the minimal total correction necessary.

## When to use

Use when integrating exactly two single-cell RNA-seq batches (e.g. CEL-Seq2 vs Smart-Seq2 count matrices) where you need batch correction that maintains cell-to-cell (local) structure, and MNN-based tools underperform on iLISI/50-RNN.

## When NOT to use

- You need to integrate more than two datasets — the current implementation of BATMAN works only with two datasets (multi-dataset integration is possible only 'in principle')
- Your data is not single-cell expression matrices (e.g. already-integrated embeddings, bulk RNA-seq, or feature tables)
- You require mutually-nearest-neighbor semantics or multi-dataset harmonization pipelines like Harmony/Scanorama by design

## Inputs

- Two single-cell RNA-seq count matrices (one per batch/dataset), pre-processed to log-counts via Seurat V3.0 pre-processing (e.g. Muraro CEL-Seq2 2,285 cells; Segerstolpe Smart-Seq2 2,394 cells)
- Number of anchor points per dataset (-anchors) and a Pearson correlation filtering threshold between anchors across datasets (-filter-cor)

## Outputs

- Batch-corrected (integrated) expression matrix aligning the query dataset to the reference dataset
- One-to-one anchor matching with local shifting vectors used for correction (implicit intermediate artifacts)

## How to apply

First normalize each dataset with Seurat V3.0 pre-processing to obtain log-counts, then build a graph whose partitions correspond to the two datasets with edges weighted by Euclidean distance between cells in N-dimensional gene space. Solve the minimum-weight one-to-one matching either with the exact blossom-based algorithm or the greedy approximation (ratio 2, recommended for extra-large datasets via the -greedy flag). From the matched anchor pairs, determine local shifting vectors and apply batch correction to align the query dataset to the reference, modifying only the query dataset. On the Muraro CEL-Seq2 (2,285 cells) vs Segerstolpe Smart-Seq2 (2,394 cells) pancreas benchmark this yielded iLISI 1.55 / cLISI 1.08 in 9 s, versus Seurat V3.0 (1.35, 22.3 s), MNNcorrect (1.18, 228.6 s), and Scanorama (1.22, 14 s).

## Related tools

- **BATMAN** (Implements the bipartite graph construction, minimum-weight matching (exact blossom-based or greedy ratio-2), local shifting vectors, and batch correction; run via batman.py) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Pre-processing of the input count matrices to log-counts before graph construction and matching)
- **Scanorama** (Alternative MNN-based integrator used as comparison baseline in the pancreas benchmark (iLISI 1.22))
- **MNNcorrect** (Alternative MNN-based integrator used as comparison baseline in the pancreas benchmark (iLISI 1.18))

## Examples

```
python3.7 batman.py -data muraro_celseq2_logcounts.csv,segerstolpe_smartseq2_logcounts.csv -output corrected.csv -anchors 500 -filter-cor 0.5
```

## Evaluation signals

- Mean iLISI of the integrated representation should exceed competing MNN-based methods (BATMAN 1.55 vs Seurat V3.0 1.35, MNNcorrect 1.18, Scanorama 1.22 on the full pancreas all-cell-type benchmark), while cLISI stays near 1.05–1.08
- Mean 50-RNN should remain high (e.g. 0.92 for BATMAN in the LB scenario), indicating retained cell-to-cell local structure
- Wall-clock runtime should be far below MNNcorrect (BATMAN ~9 s vs 228.6 s on the pancreas pair)
- Matching output must be one-to-one (a valid matching in the bipartite graph), not many-to-one
- Corrected query dataset should show reduced cross-batch discrepancy relative to the uncorrected baseline (no-correction iLISI 1.07 / cLISI 1.05)

## Limitations

- The current implementation of BATMAN works only with two datasets; integrating multiple datasets is possible only 'in principle'
- The greedy matching option trades exactness for speed with an approximation ratio of 2
- Anchors are filtered by a Pearson correlation threshold across datasets; too stringent a -filter-cor or too few -anchors can degrade matching quality
- Correction usually modifies only the query dataset to align it to the reference

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points: [evidence span withheld — anchor status: external_doc]
