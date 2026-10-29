---
name: single-cell-count-normalization
description: Use when a raw single-cell count matrix (e.g. a Seurat object or SingleCellExperiment
  such as the untreated PDAC snRNA-Seq dataset SCP1089) must be log-normalized with
  size factors and variable features scaled before clustering or LEADER subsampling.
when_to_use_negative:
- Input is already log-normalized with variable features identified and scaled (e.g.
  a processed Seurat object that LEADER can consume directly) — re-normalizing would
  double-transform the data
- The data are bulk RNA-seq or non-transcriptomic counts (e.g. the CyTOF bone marrow
  dataset), where scRNA-seq logcount/size-factor normalization assumptions do not
  directly transfer
- Only downstream cell IDs already exist and no clustering or marker scoring will
  be performed — normalization adds no value without a downstream use
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scuttle
  role: Calculates logcounts and normalizes with size factors for the SingleCellExperiment-based
    analysis route in the article
- name: Seurat
  role: NormalizeData, FindVariableFeatures (vst, 2000 features) and ScaleData produce
    the log-normalized, scaled Seurat object that the leader package requires as input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:d892118c0aa9193d5d11167d72d4a9bd33528c527fe2692e56aa0ae151f0cf57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GENIE3
  - PIDC
  - R
  - Seurat
  - leader
  - scLink
  - scuttle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-count-normalization

## Summary

Normalize raw single-cell expression counts into logcounts — via size factors with the scuttle R package or Seurat's NormalizeData — and identify/scale variable features so the data are suitable for clustering and LEADER adaptive-reweighting cell selection.

## When to use

Use when a raw single-cell count matrix (e.g. a Seurat object or SingleCellExperiment such as the untreated PDAC snRNA-Seq dataset SCP1089) must be log-normalized with size factors and variable features scaled before clustering or LEADER subsampling.

## When NOT to use

- Input is already log-normalized with variable features identified and scaled (e.g. a processed Seurat object that LEADER can consume directly) — re-normalizing would double-transform the data
- The data are bulk RNA-seq or non-transcriptomic counts (e.g. the CyTOF bone marrow dataset), where scRNA-seq logcount/size-factor normalization assumptions do not directly transfer
- Only downstream cell IDs already exist and no clustering or marker scoring will be performed — normalization adds no value without a downstream use

## Inputs

- Raw single-cell gene count matrix (e.g. Seurat object or SingleCellExperiment, such as the untreated PDAC snRNA-Seq dataset SCP1089)
- Cell metadata with unique cell IDs for the meta.data rownames (Seurat route)
- Optional marker YAML file with positive/negative marker genes per expected cell type (consumed after normalization by marker-informed LEADER)

## Outputs

- Log-normalized expression matrix (logcounts) with size factors set
- Seurat object with variable features identified (vst, 2000 features) and all genes scaled
- Normalized object ready for PCA (first 30 PCs) and LEADER adaptive_reweighting

## How to apply

Load the raw count matrix (e.g. SCP1089 snRNA-Seq subsampled to 6,000 cells) into a SingleCellExperiment or Seurat object. In the SingleCellExperiment/scuttle route, compute size factors and calculate logcounts with the scuttle R package; in the Seurat route required by the leader package, run NormalizeData, then FindVariableFeatures with selection.method='vst' and nfeatures=2000, then ScaleData over all genes. The Seurat object must carry unique cell IDs as the rownames of the meta.data dataframe, since these IDs become the subsampled cell identifiers. Rationale: LEADER's adaptive_reweighting explicitly requires a log-normalized Seurat object with variable features identified and scaled, because clustering on the first 30 principal components is performed downstream to drive cluster-based or marker-informed balanced sampling.

## Related tools

- **scuttle** (Calculates logcounts and normalizes with size factors for the SingleCellExperiment-based analysis route in the article)
- **Seurat** (NormalizeData, FindVariableFeatures (vst, 2000 features) and ScaleData produce the log-normalized, scaled Seurat object that the leader package requires as input)

## Examples

```
seu <- NormalizeData(seu); seu <- FindVariableFeatures(seu, selection.method = 'vst', nfeatures = 2000); all.genes <- rownames(seu); seu <- ScaleData(seu, features = all.genes)
```

## Evaluation signals

- The Seurat object's assays contain log-normalized expression, not raw counts, and variable features are identified (vst selection, nfeatures=2000) before any clustering call
- All genes are scaled via ScaleData(features = all.genes); no genes are left unscaled prior to PCA
- Rownames of the meta.data dataframe are unique cell IDs, since LEADER outputs these IDs as the subsample (e.g. in adaptive_reweighting_subsample.csv)
- In the scuttle route, size factors are set and logcounts are computed before downstream annotation methods such as SingleR are run
- Downstream LEADER clustering on the first 30 PCs yields clusters from which equal numbers of cells can be sampled, confirming normalization supported the intended analysis

## Limitations

- Dropout at the single-cell level could complicate cell type annotation when specific cell type markers are not expressed; this phenomenon was not addressed in the benchmarking
- Scaling all genes on large datasets can be memory-intensive, since the README instructs scaling over rownames(seu) (all genes) rather than only variable features
- Ground-truth labels for evaluating downstream performance came from previous studies rather than fresh manual annotation, so normalization quality cannot be judged against true manual labels in the source benchmark

## Evidence

- [discussion] we calculated logcounts and normalized with the size factor using the scuttle R package52: "we calculated logcounts and normalized with the size factor using the scuttle R package52"
- [readme] The input seurat oject must be log normalized and the most variable features must be identified and scaled prior to cell selection with LEADER: "The input seurat oject must be log normalized and the most variable features must be identified and scaled prior to cell selection with LEADER as follows:"
- [readme] seu <- FindVariableFeatures(seu, selection.method = 'vst', nfeatures = 2000): "seu <- FindVariableFeatures(seu, selection.method = 'vst', nfeatures = 2000)"
- [readme] Note that the seurat object must also have unique cell ID's as the rownames of the dataframe in the `seu@meta.data` slot.: "Note that the seurat object must also have unique cell ID's as the rownames of the dataframe in the `seu@meta.data` slot."
