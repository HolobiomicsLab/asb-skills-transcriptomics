---
name: per-cell-type-accuracy-heatmap-visualization
description: Use when you have cell-type label predictions for query datasets mapped
  onto a pre-trained reference (e.g., scCobra KNN label transfer on GSE114297/GSE85247/GSE83312
  pancreas queries) and need a per-cell-type accuracy breakdown, such as reproducing
  the Fig. 6b-e ACC heatmap.
when_to_use_negative:
- Reference and query datasets do not originate from the same tissue type or do not
  contain similar cell types, making per-cell-type accuracy comparison unreliable
- Predictions come from a method that retrained on the query data rather than online
  label transfer with a frozen encoder, so the comparison to Fig. 6b-e is not meaningful
- Input is raw counts without the paper's preprocessing (cells < 200 genes filtered,
  genes < 3 cells filtered, mito fraction > 5% removed, log1p normalization), since
  accuracy values will not match the published heatmap
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCobra
  role: Provides the frozen encoder and pre-trained KNN classifier whose query predictions
    feed the per-cell-type accuracy computation
  repo: https://github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Preprocesses the pancreas reference and query datasets (cell/gene filtering,
    log1p normalization) and stores cell-type labels used for accuracy grouping
- name: scIB
  role: Downstream evaluation pipeline used alongside the label-transfer comparison
    in the scCobra workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:27d67efdac8f11d6858d378f89eeb8022747b870e8fb90ede0bbc234646ef1c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - PyTorch
  - Scanpy
  - scCobra
  - scIB
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# per-cell-type-accuracy-heatmap-visualization

## Summary

Renders a per-cell-type accuracy (ACC) heatmap that breaks down label-transfer performance of annotated query cells (e.g., scCobra online label transfer onto a pancreas reference atlas) by reference cell type, exposing which cell types the pre-trained KNN classifier annotates well versus poorly.

## When to use

Use when you have cell-type label predictions for query datasets mapped onto a pre-trained reference (e.g., scCobra KNN label transfer on GSE114297/GSE85247/GSE83312 pancreas queries) and need a per-cell-type accuracy breakdown, such as reproducing the Fig. 6b-e ACC heatmap.

## When NOT to use

- Reference and query datasets do not originate from the same tissue type or do not contain similar cell types, making per-cell-type accuracy comparison unreliable
- Predictions come from a method that retrained on the query data rather than online label transfer with a frozen encoder, so the comparison to Fig. 6b-e is not meaningful
- Input is raw counts without the paper's preprocessing (cells < 200 genes filtered, genes < 3 cells filtered, mito fraction > 5% removed, log1p normalization), since accuracy values will not match the published heatmap

## Inputs

- Query cell ground-truth cell-type labels (e.g., pancreas GSE114297, GSE85247, GSE83312 query annotations)
- Predicted cell-type labels from the pre-trained KNN classifier applied to scCobra query embeddings
- Reference atlas cell-type label list (all cell types to display as heatmap rows/columns)

## Outputs

- Per-cell-type accuracy (ACC) heatmap covering all reference cell types (as in Fig. 6b-e)
- Per-cell-type accuracy values table underlying the heatmap
- Aggregated weighted F1 score consistent with the heatmap (scCobra ~0.95)

## How to apply

After annotating query cells by passing the frozen scCobra encoder's query embeddings through the pre-trained KNN classifier fit on reference embeddings, pair each query cell's predicted label with its ground-truth label. Compute per-cell-type accuracy (fraction of correct predictions within each reference cell type) and render these values as a heatmap with all reference cell types annotated on the axes. Use Scanpy-preprocessed query data (cell/gene filtering, log1p normalization) and confirm the aggregated weighted F1 matches the paper's reported values (scCobra 0.95, SCALEX 0.95, TOSICA 0.94, scVI 0.92) so the heatmap is consistent with the Fig. 6b-e comparison. Use scIB/Scanpy utilities for clustering-independent accuracy computation and a standard heatmap renderer for the final figure.

## Related tools

- **scCobra** (Provides the frozen encoder and pre-trained KNN classifier whose query predictions feed the per-cell-type accuracy computation) — https://github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocesses the pancreas reference and query datasets (cell/gene filtering, log1p normalization) and stores cell-type labels used for accuracy grouping)
- **scIB** (Downstream evaluation pipeline used alongside the label-transfer comparison in the scCobra workflow)

## Examples

```
acc = query.obs.groupby('celltype').apply(lambda g: (g['pred']==g.name if False else (g['pred']==g['celltype']).mean())); sns.heatmap(acc.to_frame('ACC'), annot=True)
```

## Evaluation signals

- Reproduced weighted F1 for scCobra matches the ~0.95 value reported in Fig. 6b-e
- The ACC heatmap is generated with all reference cell types annotated on the axes
- Per-cell-type accuracies aggregate consistently to the overall weighted F1 (scCobra 0.95, SCALEX 0.95, TOSICA 0.94, scVI 0.92)
- Query datasets are the packaged GSE114297, GSE85247, and GSE83312 pancreas queries from the Integration_pancreas_label_transfer.ipynb tutorial
- Heatmap is computed from KNN-classifier predictions on frozen-encoder embeddings, not from a retrained model

## Limitations

- The label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types to ensure optimal performance
- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance in the comparison the heatmap accompanies

## Evidence

- [discussion] knn annotation: [evidence span withheld — anchor status: not_found]
- [discussion] tissue requirement: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types"
- [discussion] three-phase training: "optimization performed in three distinct phases"
- [readme] readme method: "By leveraging a deep generative model that combines a contrastive neural network with domain adaptation, scCobra mitigates batch effects and minimizes over-correction"
- [readme] readme run entry point: [evidence span withheld — anchor status: external_doc]
