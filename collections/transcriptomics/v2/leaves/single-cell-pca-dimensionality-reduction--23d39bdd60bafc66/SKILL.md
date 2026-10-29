---
name: single-cell-pca-dimensionality-reduction
description: Use when two (or more) scRNA-seq count matrices (e.g. 10X PBMC 8k and
  Pan T cells) need a common low-dimensional embedding before batch-correction or
  integration comparison, and cells must be represented as points in N-dimensional
  space for distance-weighted matching.
when_to_use_negative:
- When the integration method itself requires operating in the high-dimensional gene
  space rather than a reduced embedding — BATMAN's headline results operate in the
  high-dimensional gene space, so a PCA-space projection changes the regime being
  tested
- When integrating more than two datasets — the BATMAN implementation used here supports
  only two datasets, and PCA-space comparisons in the source were restricted to pairs
  (10X PBMC 8k vs Pan T cells)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Consumes the shared PCA embeddings to build a Euclidean-distance-weighted
    bipartite graph for minimum-weight matching and batch correction
  repo: https://github.com/mandricigor/batman
- name: Harmony
  role: Runs on the same PCA embeddings of the two datasets as the PCA-space comparison
    baseline
- name: Scanorama
  role: Alternative integration method benchmarked in the same comparison framework
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:80ffa6460907e5965aba7c4080a55808ff8789cab598ab1009f36cdfae333c41
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
  - Harmony
  - Scanorama
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-pca-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Project single-cell RNA-seq datasets into a shared N-dimensional PCA space after Seurat V3.0-style preprocessing, so each cell is represented as a point in a common low-dimensional embedding suitable for downstream batch-integration methods such as BATMAN and Harmony.

## When to use

Use when two (or more) scRNA-seq count matrices (e.g. 10X PBMC 8k and Pan T cells) need a common low-dimensional embedding before batch-correction or integration comparison, and cells must be represented as points in N-dimensional space for distance-weighted matching.

## When NOT to use

- When the integration method itself requires operating in the high-dimensional gene space rather than a reduced embedding — BATMAN's headline results operate in the high-dimensional gene space, so a PCA-space projection changes the regime being tested
- When integrating more than two datasets — the BATMAN implementation used here supports only two datasets, and PCA-space comparisons in the source were restricted to pairs (10X PBMC 8k vs Pan T cells)

## Inputs

- Raw count matrices for two scRNA-seq datasets (e.g. 10X Genomics PBMC 8k dataset with 8,381 cells and Pan T cells dataset with 3,555 T cells)
- Seurat V3.0 pre-processing configuration to obtain log-counts per dataset

## Outputs

- Shared PCA embeddings (cells as points in N-dimensional space) for each dataset
- PCA coordinates ready to feed into BATMAN (Euclidean-distance-weighted graph construction) or Harmony

## How to apply

First apply Seurat V3.0 pre-processing steps to the raw count matrices of each dataset to obtain log-counts, as the article states was done in all experiments. Then project both log-count datasets into a shared PCA space so that each cell is a point in N-dimensional space; this PCA embedding is the coordinate system in which downstream tools operate. For BATMAN, edges of the bipartite graph between datasets are weighted by the Euclidean distance between cells in this N-dimensional PCA space, and Harmony likewise consumes the same PCA embeddings as its input. The choice of PCA-space operation matters for benchmarking: in the 10X PBMC comparison, running both BATMAN and Harmony on the same PCA embeddings allowed a fair head-to-head evaluation via mean iLISI and 10-RNN metrics with ~14 s runtimes per tool. Keep the PCA projection consistent across the tools being compared so that metric differences reflect the integration method, not the embedding.

## Related tools

- **BATMAN** (Consumes the shared PCA embeddings to build a Euclidean-distance-weighted bipartite graph for minimum-weight matching and batch correction) — https://github.com/mandricigor/batman
- **Harmony** (Runs on the same PCA embeddings of the two datasets as the PCA-space comparison baseline)
- **Scanorama** (Alternative integration method benchmarked in the same comparison framework)

## Examples

```
python3.7 batman.py -data pbmc8k_logcounts.csv,panTcells_logcounts.csv -output integrated_pbmc.csv
```

## Evaluation signals

- Both datasets must end up in a common N-dimensional PCA space where per-cell Euclidean distances are well defined (required for BATMAN's edge weighting)
- The same PCA embeddings should be passed to every tool being compared (BATMAN and Harmony) so integration metrics reflect the method, not the embedding
- On the 10X PBMC datasets in PCA space, expect Harmony to show slightly better iLISI and 10-RNN than BATMAN with both tools completing in ~14 s — deviations flag an inconsistent embedding or wrong preprocessing
- Seurat V3.0-style pre-processing (log-counts) must precede the PCA projection, matching the paper's stated workflow
- Compute mean iLISI and 10-RNN on the integrated output and record wall-clock runtimes for the comparison table

## Limitations

- The current BATMAN implementation works only with two datasets; PCA-space integration of more than two datasets is not supported (only possible 'in principle')
- In PCA space on the 10X PBMC datasets, BATMAN loses its headline advantage: Harmony showed slightly better iLISI and 10-RNN, so PCA-space operation is not where BATMAN's reported up-to-80% metric gains are realized
- PCA is a linear dimensionality reduction; the paper notes Gaussian-based approaches like ComBat and limma assume Gaussian expression values, and ZIFA addresses zero inflation — PCA-space comparisons inherit such distributional caveats

## Evidence

- [readme] README confirms the distance-weighted graph construction used on the embedding: [evidence span withheld — anchor status: external_doc]
