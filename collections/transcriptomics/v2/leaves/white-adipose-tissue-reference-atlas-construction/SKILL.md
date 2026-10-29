---
name: white-adipose-tissue-reference-atlas-construction
description: Use when multiple independent white adipose tissue (or comparable multi-donor
  tissue) scRNA-seq datasets must be uniformly annotated and integrated into a single
  reference atlas with cell type/state labels.
when_to_use_negative:
- The task is single-dataset clustering or batch correction within one sample set
  rather than cross-study atlas construction
- Input data are not single-cell transcriptomes of adipose tissue (or a tissue where
  re-using WATLAS labels is not biologically defensible)
- A quick supervised label transfer suffices and there is no need to re-annotate and
  re-integrate the six source studies
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: Per-dataset interpretable joint clustering and cell type/state annotation
    via kernel-based joint NMF; run with jointly(data, batch.var = 'batch_label')
    after preprocess()
  repo: https://github.com/madsen-lab/rJOINTLY
- name: scVI/scANVI
  role: Deep-learning-based integration of the six JOINTLY-labelled datasets across
    studies to harmonise cell type/state annotations into WATLAS
- name: Seurat
  role: Framework used to hold expression objects, run FindNeighbors/FindClusters
    on the JOINTLY reduction, and embed via RunUMAP
  repo: https://github.com/satijalab/seurat
- name: aricode
  role: Computes ARI and NMI to validate that cluster labels agree with cell type/state
    annotations
- name: UCell
  role: Scores JOINTLY's learned modules (AddModuleScore_UCell) for interpreting cell
    states during annotation
  repo: https://github.com/carmonalab/UCell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:b74fd7599e9236ef271cfc9ae4d9b065bcc2df31211ce77fb57d3562051d089f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - aricode
  - scVI
  - scVI/scANVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# white-adipose-tissue-reference-atlas-construction

## Summary

Constructs WATLAS, a white adipose tissue reference atlas, by annotating cells from six independent scRNA-seq datasets (Tabula Sapiens, Jaitin GSE128518, Vijay GSE129363, Hildreth GSE155960, Emont SCP1376, Barboza) with JOINTLY and integrating them with scANVI into a consensus annotation of ~300,000 VAT/SAT cells at 17 major-type and 43 subtype resolution.

## When to use

Use when multiple independent white adipose tissue (or comparable multi-donor tissue) scRNA-seq datasets must be uniformly annotated and integrated into a single reference atlas with cell type/state labels.

## When NOT to use

- The task is single-dataset clustering or batch correction within one sample set rather than cross-study atlas construction
- Input data are not single-cell transcriptomes of adipose tissue (or a tissue where re-using WATLAS labels is not biologically defensible)
- A quick supervised label transfer suffices and there is no need to re-annotate and re-integrate the six source studies

## Inputs

- Six public white adipose tissue scRNA-seq datasets: Tabula Sapiens, Jaitin GSE128518, Vijay GSE129363, Hildreth GSE155960, Emont SCP1376, Barboza
- Seurat, SingleCellExperiment or matrix-like expression objects with a batch variable (batch_label) per dataset
- WATLAS reference annotations at Broad SCP2289 for validation
- WATLAS model weights from Zenodo deposit 8086433 (optional)

## Outputs

- watlas_cell_annotations.csv: cell-by-cell-type/state consensus table with ~300,000 cells, 17 major cell types and 43 subtypes from VAT and SAT
- Per-dataset JOINTLY clusters, interpretable factors/modules and cell type/state labels
- scANVI-integrated, cross-dataset harmonised annotation object

## How to apply

Retrieve the six public WAT scRNA-seq datasets and preprocess each with JOINTLY's preprocess() step using the batch variable (batch_label). Run JOINTLY per dataset with jointly(data, batch.var = 'batch_label') to obtain joint clusters, interpretable NMF factors and cell type/state labels, re-running up to 5 times (with different seeds) and keeping the best solution, since results vary between runs. Integrate the labelled datasets across studies with scANVI, using the parameters stated in the article, to harmonise cell type/state annotations across donors, conditions and depots. Compile the consensus annotations into a single cell-by-cell-type/state table (~300,000 cells, 17 major types, 43 subtypes) and save it as watlas_cell_annotations.csv. WATLAS model weights are available at Zenodo (deposit 8086433) and the deposited atlas at Broad SCP2289 for comparison.

## Related tools

- **JOINTLY** (Per-dataset interpretable joint clustering and cell type/state annotation via kernel-based joint NMF; run with jointly(data, batch.var = 'batch_label') after preprocess()) — https://github.com/madsen-lab/rJOINTLY
- **scVI/scANVI** (Deep-learning-based integration of the six JOINTLY-labelled datasets across studies to harmonise cell type/state annotations into WATLAS)
- **Seurat** (Framework used to hold expression objects, run FindNeighbors/FindClusters on the JOINTLY reduction, and embed via RunUMAP) — https://github.com/satijalab/seurat
- **aricode** (Computes ARI and NMI to validate that cluster labels agree with cell type/state annotations)
- **UCell** (Scores JOINTLY's learned modules (AddModuleScore_UCell) for interpreting cell states during annotation) — https://github.com/carmonalab/UCell

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- watlas_cell_annotations.csv exists and contains ~300,000 rows
- The annotation table contains exactly 43 distinct subtype labels (and 17 major cell types)
- Cell type/state assignments agree with the deposited WATLAS reference at Broad SCP2289, quantified by ARI/NMI using aricode
- Per-dataset ARI/NMI between transferred labels and JOINTLY clusters is high (liver test example: ARI ≈ 0.74, NMI ≈ 0.63)
- JOINTLY runs repeated up to 5 times and the best solution retained, confirming solution stability was checked

## Limitations

- JOINTLY's stochasticity means setting a seed and re-running produces a different result; it may be necessary to run up to 5 times and choose the best solution
- Consensus PCA (cpca) within JOINTLY is not advised for complex multi-condition, multi-sample datasets like WATLAS; regular PCA is recommended instead
- The R version of JOINTLY was used for all manuscript analyses; the Python (pyJOINTLY) version does not produce identical results
- No method is always best across all integration tasks, so performance should be evaluated on each new task with ARI/NMI rather than assumed

## Evidence

- [discussion] Process and label six independent datasets with JOINTLY, then integrate across datasets using scANVI to build the WATLAS: "by processing and labelling six independent datasets with JOINTLY and subsequently, integrating across datasets using scANVI"
- [methods] Jointly call with batch variable: "liver <- jointly(liver, batch.var = "batch_label")"
- [readme] Re-run advice due to stochasticity: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [readme] WATLAS hosting and model weights: "For the white adipose tissue atlas (WATLAS), the model weights are availiable [here](https://zenodo.org/deposit/8086433)"
- [discussion] cpca caveat for multi-sample datasets: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
