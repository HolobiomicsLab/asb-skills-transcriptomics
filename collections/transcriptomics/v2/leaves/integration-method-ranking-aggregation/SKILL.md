---
name: integration-method-ranking-aggregation
description: Use when comparing multiple single-cell integration methods on the same
  dataset and you need a reproducible multi-metric ranking (batch/cell-type ARI and
  ASW) summarised into a single benchmark table.
when_to_use_negative:
- You only need a single integration result for downstream analysis rather than a
  comparative benchmark across methods — running scMerge2 alone suffices
- Your data are already integrated or you lack per-cell batch and cell-type labels,
  since ARI/ASW computation requires both batch labels and cell-type annotations as
  references
tools:
- name: scMerge2
  role: Method under test; produces corrected representation via pseudo-bulk building
    and pseudo-replication within conditions
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Baseline integration method providing a corrected embedding for metric computation
- name: Scanorama
  role: Baseline integration method providing a corrected embedding for metric computation
- name: Harmony
  role: Baseline integration method providing a joint embedding for metric computation
- name: BBKNN
  role: Baseline rapid integration method providing a joint embedding for metric computation
- name: scVI
  role: Baseline deep-learning integration method providing a joint embedding for
    metric computation
- name: scANVI
  role: Baseline deep-learning integration method providing a joint embedding for
    metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:8d0b7ab27f260f810330bb5da243c6aad45a8b86acc8e76167721f1fecc3e904
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# integration-method-ranking-aggregation

## Summary

Benchmark single-cell data integration methods (e.g. scMerge2, scMerge2-h vs Seurat, Scanorama, Harmony, BBKNN, scVI, scANVI, DESC) by computing batch-ARI, cell-type-ARI, batch-ASW and cell-type-ASW on each corrected embedding and aggregating them into an overall per-method ranking table.

## When to use

Use when comparing multiple single-cell integration methods on the same dataset and you need a reproducible multi-metric ranking (batch/cell-type ARI and ASW) summarised into a single benchmark table.

## When NOT to use

- You only need a single integration result for downstream analysis rather than a comparative benchmark across methods — running scMerge2 alone suffices
- Your data are already integrated or you lack per-cell batch and cell-type labels, since ARI/ASW computation requires both batch labels and cell-type annotations as references

## Inputs

- Integrated/corrected embeddings from each method (scMerge2, scMerge2-h, Seurat, SeuratRPCA, Scanorama, Harmony, BBKNN, scVI, scANVI, DESC) computed on the same cell subset
- Per-cell batch labels for the 6 batches of the 200k-cell Liu+Stephenson COVID-19 CITE-seq subset (17,446 genes, 87 proteins)
- Per-cell cell-type annotations used as ground truth for biological preservation metrics

## Outputs

- benchmark_metrics.csv containing batch-ARI, cell-type-ARI, batch-ASW and cell-type-ASW plus the aggregated overall ranking for every method compared in Fig. 2a-b

## How to apply

Run every integration method — scMerge2 (with pseudo-bulk building and pseudo-replication within conditions), scMerge2-h (hierarchical integration with a data-guided order, e.g. integrating batches within one study first), and the baselines — on the identical input subset so embeddings are directly comparable. For each integrated representation, cluster cells and compute batch-ARI (agreement with batch labels) and cell-type-ARI (agreement with cell-type annotations). Then compute batch-ASW and cell-type-ASW as the average silhouette width of cells grouped by batch and by cell type on each embedding. Aggregate the metrics into the overall ranking per method and assemble them into a single benchmark table (benchmark_metrics.csv) reproducing the article's Fig. 2a-b comparison, where scMerge2/scMerge2-h better balance batch effect removal and biological signal preservation on the 200k-cell Liu+Stephenson COVID-19 CITE-seq subset (17,446 genes, 87 proteins, 6 batches).

## Related tools

- **scMerge2** (Method under test; produces corrected representation via pseudo-bulk building and pseudo-replication within conditions) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Baseline integration method providing a corrected embedding for metric computation)
- **Scanorama** (Baseline integration method providing a corrected embedding for metric computation)
- **Harmony** (Baseline integration method providing a joint embedding for metric computation)
- **BBKNN** (Baseline rapid integration method providing a joint embedding for metric computation)
- **scVI** (Baseline deep-learning integration method providing a joint embedding for metric computation)
- **scANVI** (Baseline deep-learning integration method providing a joint embedding for metric computation)

## Examples

```
BiocManager::install("scMerge")  # install scMerge/scMerge2 from Bioconductor, then run each integration method on the 200k-cell subset and aggregate batch-ARI, cell-type-ARI, batch-ASW and cell-type-ASW into benchmark_metrics.csv
```

## Evaluation signals

- benchmark_metrics.csv contains all five metrics (batch-ARI, cell-type-ARI, batch-ASW, cell-type-ASW, overall ranking) for every method compared in Fig. 2a-b
- Per-method metric values and their ordering are consistent with the reported Fig. 2a-b results
- All methods were run on the identical 200k-cell subset (17,446 genes, 87 proteins, 6 batches), so metrics are directly comparable across methods
- ARI values agree with the respective label sets: batch-ARI with batch labels and cell-type-ARI with cell-type annotations
- ASW values are computed as average silhouette widths grouped by batch and by cell type on each integrated embedding

## Limitations

- Embedding-only outputs (as with several rapid methods) restrict downstream use — the article notes that 'the lack of adjusted expression matrices restricts the utilisation of such integrative results'
- Ranking results are dataset-specific to the 200k-cell Liu+Stephenson COVID-19 benchmark subset and may not generalise to other datasets or modalities
- Multi-modal integration (RNA plus the 87 CITE-seq proteins) was only performed within each modality in the paper and is deferred to future work

## Evidence

- [methods] workflow: cluster cells and compute ARIs: [evidence span withheld — anchor status: not_found]
- [methods] workflow: silhouette widths: [evidence span withheld — anchor status: not_found]
- [results] workflow: assemble benchmark table: [evidence span withheld — anchor status: not_found]
- [intro] article: embedding-only methods: "several of these rapid procedures (deepMNN16, BBKNN17, Harmony18, scVI19, scANVI20 and DESC21) focus on extracting the joint embedding"
- [discussion] article: three innovations: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [readme] readme: scMerge package: "scMerge is a R package for merging and normalising single-cell RNA-Seq datasets."
