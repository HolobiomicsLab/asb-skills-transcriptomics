---
name: cite-seq-multimodal-integration-benchmarking
description: Use when comparing integration algorithms (e.g. scMerge2, scMerge2-h,
  Seurat, Harmony, scVI, Scanorama) on a multi-batch CITE-seq or scRNA-seq dataset
  where both batch labels and cell-type annotations are available for evaluation.
when_to_use_negative:
- Joint integration across modalities (e.g. combining gene expression and protein
  information in one model) — the paper's integration was done within each modality
  and multi-modal joint use is deferred to future work
- Small single-study datasets where atlas-scale hierarchical integration is unnecessary
  — scMerge2's innovations target studies with thousands of samples and millions of
  cells
- When a knowledge-guided integration order is required — scMerge2 currently only
  implements a data-guided order (batches within one study or studies of similar size
  first)
tools:
- name: scMerge2
  role: Primary integration method using pseudo-bulk building and pseudo-replication
    within conditions
  repo: https://github.com/SydneyBioX/scMerge
- name: scMerge2-h
  role: Hierarchical integration variant of scMerge2 with a data-guided integration
    order
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Baseline integration method producing corrected embeddings for the same subset
- name: Scanorama
  role: Baseline integration method producing corrected embeddings for the same subset
- name: Harmony
  role: Baseline integration method producing corrected embeddings for the same subset
- name: BBKNN
  role: Baseline integration method producing corrected embeddings for the same subset
- name: scVI
  role: Baseline integration method producing corrected embeddings for the same subset
- name: scANVI
  role: Baseline integration method producing corrected embeddings for the same subset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:bca8737d0d608ab05d2740677c48f026109cfb0cc0919a324a0fc9be244171cb
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
  - scMerge2-h
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cite-seq-multimodal-integration-benchmarking

## Summary

Benchmark single-cell integration methods on a CITE-seq subset by clustering cells in each corrected embedding and scoring batch/cell-type ARI and silhouette widths, as done for scMerge2 and scMerge2-h against Seurat, fastMNN, Harmony, Liger, scVI and Scanorama on the 200k-cell Liu and Stephenson COVID-19 data.

## When to use

Use when comparing integration algorithms (e.g. scMerge2, scMerge2-h, Seurat, Harmony, scVI, Scanorama) on a multi-batch CITE-seq or scRNA-seq dataset where both batch labels and cell-type annotations are available for evaluation.

## When NOT to use

- Joint integration across modalities (e.g. combining gene expression and protein information in one model) — the paper's integration was done within each modality and multi-modal joint use is deferred to future work
- Small single-study datasets where atlas-scale hierarchical integration is unnecessary — scMerge2's innovations target studies with thousands of samples and millions of cells
- When a knowledge-guided integration order is required — scMerge2 currently only implements a data-guided order (batches within one study or studies of similar size first)

## Inputs

- 200k-cell Liu and Stephenson COVID-19 CITE-seq subset (17,446 genes, 87 proteins, 6 batches) from deposited data
- Batch labels for each cell
- Cell-type annotations for each cell
- Stably expressed genes (SEG) list, e.g. segList$human$human_scSEG from the scMerge package

## Outputs

- Corrected embeddings/adjustments from each of the nine integration methods (scMerge2, scMerge2-h and seven baselines)
- benchmark_metrics.csv with batch-ARI, cell-type-ARI, batch-ASW, cell-type-ASW and the overall ranking per method, reproducing the Fig. 2a-b comparison

## How to apply

Load the deposited 200k-cell Liu+Stephenson CITE-seq benchmarking subset (17,446 genes, 87 proteins, 6 batches) and run scMerge2 with pseudo-bulk building and pseudo-replication within conditions; run scMerge2-h with hierarchical integration using a data-guided order (batches within one study, or studies of similar size, first). Run the baseline methods (Seurat, Scanorama, Harmony, BBKNN, scVI, scANVI) on the same subset to obtain corrected embeddings, cluster cells in each representation, and compute batch-ARI (agreement with batch labels) and cell-type-ARI (agreement with cell-type annotations) plus batch-ASW and cell-type-ASW silhouette widths. Aggregate the metrics into a per-method overall ranking and assemble them into benchmark_metrics.csv reproducing the Fig. 2a-b comparison.

## Related tools

- **scMerge2** (Primary integration method using pseudo-bulk building and pseudo-replication within conditions) — https://github.com/SydneyBioX/scMerge
- **scMerge2-h** (Hierarchical integration variant of scMerge2 with a data-guided integration order) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Baseline integration method producing corrected embeddings for the same subset)
- **Scanorama** (Baseline integration method producing corrected embeddings for the same subset)
- **Harmony** (Baseline integration method producing corrected embeddings for the same subset)
- **BBKNN** (Baseline integration method producing corrected embeddings for the same subset)
- **scVI** (Baseline integration method producing corrected embeddings for the same subset)
- **scANVI** (Baseline integration method producing corrected embeddings for the same subset)

## Examples

```
BiocManager::install("scMerge"); library(scMerge); data(segList); segList$human$human_scSEG
```

## Evaluation signals

- benchmark_metrics.csv contains all five evaluation metrics for every method compared in Fig. 2a-b
- Per-method metric values and their ordering are consistent with the reported Fig. 2a-b results
- Both scMerge2 settings outperform the seven baselines overall, balancing batch effect removal and biological signal preservation
- batch-ARI and cell-type-ARI are computed by clustering cells in each integrated representation against batch and cell-type labels
- batch-ASW and cell-type-ASW are average silhouette widths computed on each integrated embedding grouped by batch and by cell type

## Limitations

- Integration was only performed within each modality; joint use of multi-modal information is deferred to future work
- The integration order is data-guided only; a knowledge-guided order is described as an alternative that is not used
- Embedding-only outputs increase interpretability challenges for downstream features such as cell–cell interactions or pathway information
- The concatenating integration approach is more computationally intensive than the balanced tree approach

## Evidence

- [abstract] scMerge2 removes dataset variability in CITE-seq experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments, demonstrating its applicability"
- [discussion] Three innovations behind scMerge2: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [discussion] Data-guided integration order: "Our current method is based on a data-guided order, in which we integrate batches within one study or studies with similar size first."
- [intro] Rationale for adjusted expression output: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [readme] scMerge R package scope: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
