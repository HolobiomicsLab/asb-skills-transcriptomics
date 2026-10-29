---
name: hierarchical-integration-ordering
description: Use when integrating many samples/studies (thousands of samples, millions
  of cells, e.g. COVID-19 CITE-seq collections) where a single flat integration is
  computationally infeasible or mixes heterogeneous studies.
when_to_use_negative:
- Input is a single small study with few batches — flat scMerge/scMerge2 without hierarchical
  ordering suffices and the concatenating approach may be cheaper than a balanced
  tree
- You need joint multi-modal integration (e.g., combining RNA and ADT/protein information)
  — the paper only performs integration within each modality
- You need an embedding-only baseline quickly and no adjusted expression matrix —
  many rapid methods (Harmony, BBKNN, scVI) return embeddings directly, though they
  lose sample-level interpretability
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMerge2
  role: Core algorithm providing hierarchical integration, pseudo-bulk building and
    pseudo-replication; scMerge2-h applies a hierarchical (data-guided) integration
    order
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Baseline integration method compared against the hierarchical scMerge2/scMerge2-h
    result on the COVID-19 CITE-seq benchmark
- name: Harmony
  role: Baseline joint-embedding integration method compared against scMerge2-h
- name: Scanorama
  role: Baseline integration method compared against scMerge2-h
- name: scVI
  role: Baseline joint-embedding integration method compared against scMerge2-h
- name: BBKNN
  role: Baseline rapid integration method run on the same COVID-19 subset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:0c70e2054779ef67c4842fb783c7e54fa450a77f57b459a6888498f3f853c148
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - BBKNN
  - Harmony
  - Scanorama
  - Seurat
  - scANVI
  - scMerge2
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hierarchical-integration-ordering

## Summary

A scMerge2 integration strategy for atlas-scale multi-sample multi-condition single-cell studies in which batches and studies are merged in a hierarchical, data-guided order (e.g., batches within one study first, or studies of similar size first), combined with pseudo-bulk building and pseudo-replication to make integration of millions of cells tractable.

## When to use

Use when integrating many samples/studies (thousands of samples, millions of cells, e.g. COVID-19 CITE-seq collections) where a single flat integration is computationally infeasible or mixes heterogeneous studies.

## When NOT to use

- Input is a single small study with few batches — flat scMerge/scMerge2 without hierarchical ordering suffices and the concatenating approach may be cheaper than a balanced tree
- You need joint multi-modal integration (e.g., combining RNA and ADT/protein information) — the paper only performs integration within each modality
- You need an embedding-only baseline quickly and no adjusted expression matrix — many rapid methods (Harmony, BBKNN, scVI) return embeddings directly, though they lose sample-level interpretability

## Inputs

- Multi-sample multi-condition single-cell count matrices (e.g., SingleCellExperiment objects; scRNA-seq, CyTOF, imaging mass cytometry, CITE-seq)
- Batch/study and condition labels per cell (e.g., 200k-cell COVID-19 subset with 6 batches; cell-type annotations for evaluation)
- Optionally, stably expressed genes (human SEG via segList$human$human_scSEG)

## Outputs

- Hierarchically integrated joint embedding over all cells
- Adjusted expression matrix for downstream sample-level analyses
- Benchmark metrics table (e.g., benchmark_metrics.csv with batch-ARI, cell-type-ARI, batch-ASW, cell-type-ASW and overall ranking per method)

## How to apply

Apply scMerge2's hierarchical integration by ordering the merges in a data-guided way: integrate batches within one study first, or merge studies of similar size, before combining larger groups. At each level, run scMerge2 using pseudo-bulk building to minimise processing demand and pseudo-replication that accounts for phenotypes/conditions, producing a joint embedding and adjusted expression matrix. On a concrete benchmark this was applied to a 200k-cell Liu and Stephenson COVID-19 CITE-seq subset (17,446 genes, 87 proteins, 6 batches), where scMerge2-h with hierarchical integration was compared against Seurat, SeuratRPCA, fastMNN, Liger, Harmony, scVI and Scanorama. Rationale: hierarchical ordering plus pseudo-bulk/pseudo-replication is what makes integration of atlas-scale collections (e.g., 20 COVID-19 studies, ~5 million cells, 1298 samples) finish within a day while better balancing batch effect removal and biological signal preservation than seven baselines across five evaluation metrics.

## Related tools

- **scMerge2** (Core algorithm providing hierarchical integration, pseudo-bulk building and pseudo-replication; scMerge2-h applies a hierarchical (data-guided) integration order) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Baseline integration method compared against the hierarchical scMerge2/scMerge2-h result on the COVID-19 CITE-seq benchmark)
- **Harmony** (Baseline joint-embedding integration method compared against scMerge2-h)
- **Scanorama** (Baseline integration method compared against scMerge2-h)
- **scVI** (Baseline joint-embedding integration method compared against scMerge2-h)
- **BBKNN** (Baseline rapid integration method run on the same COVID-19 subset)

## Evaluation signals

- The integrated representation supports clustering whose cell-type-ARI and batch-ARI can be computed per method and compared to Fig. 2a-b ordering (scMerge2/scMerge2-h outperforming Seurat, SeuratRPCA, fastMNN, Liger, Harmony, scVI, Scanorama)
- Batch-ASW and cell-type-ASW silhouette widths are computed on each integrated embedding and all five metrics appear in the benchmark table for every method
- Overall ranking aggregated from the metrics reproduces the reported scMerge2/scMerge2-h advantage over the seven baselines
- Runtime stays feasible (the paper reports integrating ~5 million COVID-19 cells within a day), indicating pseudo-bulk building is reducing processing demand
- Adjusted expression matrix (not just an embedding) is produced, allowing downstream sample-level analyses such as disease-progression signature discrimination

## Limitations

- Integration order is data-guided only; a knowledge-guided order is described as an alternative that was not used
- Integration is performed within each modality; joint use of multi-modal information is deferred to future work
- Embedding-only outputs increase interpretability challenges for downstream features such as cell–cell interactions or pathway information
- The concatenating integration approach is more computationally intensive than the balanced tree approach

## Evidence

- [discussion] Three essential innovations: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [discussion] Data-guided ordering: "Our current method is based on a data-guided order, in which we integrate batches within one study or studies with similar size first."
- [discussion] Atlas-scale scope: "Our algorithm enabled the atlas-scale integration of 20 global COVID-19 studies with around 5 million cells from 963 donors, 1298 samples."
- [readme] Package scope: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
