---
name: over-correction-quantification-across-tissues
description: Use when integrating multi-sample, multi-condition (multi-tissue) scRNA-seq
  datasets and you need to check the chosen method (JOINTLY, Harmony, scVI, etc.)
  for over-correction of true biological condition differences.
when_to_use_negative:
- The integration task involves a single tissue/condition where only batch effects
  (not biological variability) need removing — over-correction is not the relevant
  failure mode.
- You only need overall clustering performance (ARI/NMI vs. labels) rather than testing
  retention of across-condition biological signal.
- Input is raw expression data that has not yet been merged into a batch-labelled
  multi-sample object; preprocessing and merging must precede this quantification.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: Primary integration method under test; run via jointly(merged, batch.var =
    "batch_label") in regular PCA mode on the merged pancreas-lung object
  repo: https://github.com/madsen-lab/rJOINTLY
- name: fastMNN
  role: Comparison batch-integration method producing embeddings/clusters for the
    same over-correction benchmark
- name: Harmony
  role: Comparison batch-integration method for the multi-condition mixture
- name: scVI
  role: Comparison deep-learning-based integration method; reported (with Scanorama)
    to retain biological variation across conditions
- name: LIGER
  role: Comparison NMF-based interpretable integration method for the benchmark
- name: Scanorama
  role: Comparison integration method in the over-correction evaluation
- name: Seurat
  role: Handles the merged Seurat object, FindNeighbors/FindClusters (dims 1:15, res
    = 0.05) and RunUMAP on each method's reduction
  repo: https://github.com/satijalab/seurat
- name: UCell
  role: AddModuleScore_UCell scores the interferon-signalling module on endothelial
    cells to quantify retained across-tissue signal
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc Wilcoxon rank-sum test identifies the 3529 DE genes between pancreas-
    and lung-derived endothelial cells
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:d8dcfe91d277bc50ad501269e8ff9e42707f8fdece4fa6fe9b91f65e09182570
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - Harmony
  - JOINTLY
  - LIGER
  - Scanorama
  - Seurat
  - UCell
  - fastMNN
  - presto
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# over-correction-quantification-across-tissues

## Summary

Quantifies whether a single-cell batch-integration method removes within-tissue batch effects without erasing genuine across-tissue biological variability, using the pancreas-lung multi-donor benchmark from the JOINTLY paper (e.g., interferon-signalling module scores and 3529 pancreas-vs-lung endothelial DE genes retained).

## When to use

Use when integrating multi-sample, multi-condition (multi-tissue) scRNA-seq datasets and you need to check the chosen method (JOINTLY, Harmony, scVI, etc.) for over-correction of true biological condition differences.

## When NOT to use

- The integration task involves a single tissue/condition where only batch effects (not biological variability) need removing — over-correction is not the relevant failure mode.
- You only need overall clustering performance (ARI/NMI vs. labels) rather than testing retention of across-condition biological signal.
- Input is raw expression data that has not yet been merged into a batch-labelled multi-sample object; preprocessing and merging must precede this quantification.

## Inputs

- Lung and Pancreas benchmark scRNA-seq datasets and cell type labels from Zenodo record 8298157 (e.g., Human_Liver.rds-style .rds Seurat objects and *_labels.tsv files)
- Merged multi-condition Seurat/SingleCellExperiment object with a batch_label metadata variable

## Outputs

- Per-method embeddings and clusterings for JOINTLY, fastMNN, Harmony, scVI, LIGER and Scanorama on the merged pancreas-lung mixture
- Within-tissue integration scores (iLISI/cLISI) and over-correction scores per method
- Pancreas-vs-lung endothelial DE gene list (~3529 genes, wilcoxauc) and per-cell interferon-signalling UCell module scores
- Per-method UMAP plots colored by cluster, cell type and batch (Fig. 3A-F analogue)

## How to apply

Download the Lung and Pancreas benchmark datasets and cell type labels from Zenodo record 8298157 and merge them into a single multi-condition mixture Seurat object with a batch_label variable. Run JOINTLY in regular PCA mode — the authors advise against consensus PCA for multi-condition, multi-sample datasets — e.g. jointly(merged, batch.var = "batch_label"), then FindNeighbors/FindClusters on the JOINTLY reduction (dims 1:15, res = 0.05) and RunUMAP. Run comparison methods (fastMNN, Harmony, scVI, LIGER, Scanorama) on the same merged object and compute within-tissue integration scores (iLISI/cLISI) plus over-correction metrics under the same ARI/NMI-style framework for all methods. To quantify retained across-tissue signal, identify DE genes between pancreas- and lung-derived endothelial cells with a Wilcoxon rank-sum test (wilcoxauc, ~3529 genes) and score the interferon-signalling module with AddModuleScore_UCell, visualizing per-method UMAPs colored by cluster, cell type and batch to reproduce Fig. 3A-F.

## Related tools

- **JOINTLY** (Primary integration method under test; run via jointly(merged, batch.var = "batch_label") in regular PCA mode on the merged pancreas-lung object) — https://github.com/madsen-lab/rJOINTLY
- **fastMNN** (Comparison batch-integration method producing embeddings/clusters for the same over-correction benchmark)
- **Harmony** (Comparison batch-integration method for the multi-condition mixture)
- **scVI** (Comparison deep-learning-based integration method; reported (with Scanorama) to retain biological variation across conditions)
- **LIGER** (Comparison NMF-based interpretable integration method for the benchmark)
- **Scanorama** (Comparison integration method in the over-correction evaluation)
- **Seurat** (Handles the merged Seurat object, FindNeighbors/FindClusters (dims 1:15, res = 0.05) and RunUMAP on each method's reduction) — https://github.com/satijalab/seurat
- **UCell** (AddModuleScore_UCell scores the interferon-signalling module on endothelial cells to quantify retained across-tissue signal) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc Wilcoxon rank-sum test identifies the 3529 DE genes between pancreas- and lung-derived endothelial cells) — https://github.com/immunogenomics/presto

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); liver$object <- FindClusters(liver$object, res = 0.05)
```

## Evaluation signals

- JOINTLY improves within-tissue iLISI/cLISI (batch removal) while the interferon-signalling module scores stay separable between pancreas- and lung-derived endothelial cells.
- The ~3529 pancreas-vs-lung endothelial DE genes (wilcoxauc) remain detectable in the integrated embedding rather than being erased by over-correction.
- Across-method comparison uses the same ARI/NMI-style quantitative framework on the identical merged mixture, so scores are directly comparable.
- UMAP visualizations colored by cluster, cell type, and batch show batch mixing within each tissue but retained tissue separation, matching Fig. 3A-F.
- README sanity check: repeated JOINTLY runs vary (set seed, up to 5 runs); a successful liver-reference run yields ARI ≈ 0.7416 and NMI ≈ 0.6325.

## Limitations

- Consensus PCA (cPCA) in JOINTLY is not advised for complex multi-condition, multi-sample datasets; regular PCA is recommended instead.
- No single integration method is best across all tasks — performance must be re-evaluated on each new integration task.
- Integration metrics do not reveal which genes or features drive integration and clustering, so module/DE-gene checks are needed.
- JOINTLY is stochastic: setting a seed and re-running gives different results, and it can be worth running up to 5 times and choosing the best solution.
- The R and Python (pyJOINTLY) versions do not produce identical results; all manuscript analyses used the R version.

## Evidence

- [intro] We evaluate the robustness against over-correction by integrating multi-donor datasets from different tissues and find that JOINTLY removes within-tissue batch effects but retains across-tissue: "We evaluate the robustness against over-correction by integrating multi-donor datasets from different tissues and find that JOINTLY removes within-tissue batch effects but retains across-tissue"
- [discussion] we found that JOINTLY, scVI, and Scanorama retain biological variation across conditions, while removing batch effects: [evidence span withheld — anchor status: not_found]
- [discussion] For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
- [readme] Datasets, labels, embeddings and results used for testing JOINTLY can be downloaded here: https://zenodo.org/record/8298157: "Datasets, labels, embeddings and results used for testing JOINTLY can be downloaded here: https://zenodo.org/record/8298157"
- [methods] liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15): "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)
liver$object <- FindClusters(liver$object, res = 0.05)"
- [readme] Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution.: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
