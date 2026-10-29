---
name: multi-condition-dataset-merging
description: Use when integrating multi-sample, multi-condition scRNA-seq mixtures
  (e.g., different tissues from multiple donors) where batch effects within conditions
  must be removed without erasing across-condition biological differences.
when_to_use_negative:
- Simple single-tissue, multi-sample integration where all samples share the same
  axes of variation — consensus PCA in JOINTLY may be acceptable there, and the over-correction
  test is not the goal
- Datasets lacking a batch/condition metadata variable (batch_label), since jointly()
  requires batch.var to define batches
- Reproducing analyses with pyJOINTLY expecting identical results — the README states
  the R and Python versions do not produce identical results and all manuscript analyses
  used the R version
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: Primary integration method; run in regular PCA mode via jointly(merged, batch.var
    = 'batch_label') on the merged mixture
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Object handling, FindNeighbors/FindClusters (dims 1:15, res 0.05), RunUMAP
    and visualization of the merged multi-condition object
  repo: https://github.com/satijalab/seurat
- name: UCell
  role: AddModuleScore_UCell to score the interferon-signalling module on endothelial
    cells per method embedding
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc Wilcoxon rank-sum test to find the 3529 DE genes between pancreas-
    and lung-derived endothelial cells
  repo: https://github.com/immunogenomics/presto
- name: fastMNN
  role: Comparison batch-integration method run on the same merged mixture
- name: Harmony
  role: Comparison batch-integration method run on the same merged mixture
- name: scVI
  role: Comparison batch-integration method run on the same merged mixture
- name: LIGER
  role: Comparison batch-integration method run on the same merged mixture
- name: Scanorama
  role: Comparison batch-integration method run on the same merged mixture
- name: aricode
  role: ARI/NMI calculation between cell type labels and cluster labels for evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:337cecf1d939e47d162205a1ecdfaa635129c0b5d6f72161617afbcbd183dcc3
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
  - aricode
  - fastMNN
  - presto
  - scVI
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-condition-dataset-merging

## Summary

Merges multi-sample, multi-condition scRNA-seq datasets (e.g., Lung and Pancreas benchmarks from Zenodo record 8298157) into a single mixture object with a batch_label variable, then integrates them with JOINTLY in regular PCA mode to remove within-tissue batch effects while retaining across-tissue biological variability (over-correction robustness test).

## When to use

Use when integrating multi-sample, multi-condition scRNA-seq mixtures (e.g., different tissues from multiple donors) where batch effects within conditions must be removed without erasing across-condition biological differences.

## When NOT to use

- Simple single-tissue, multi-sample integration where all samples share the same axes of variation — consensus PCA in JOINTLY may be acceptable there, and the over-correction test is not the goal
- Datasets lacking a batch/condition metadata variable (batch_label), since jointly() requires batch.var to define batches
- Reproducing analyses with pyJOINTLY expecting identical results — the README states the R and Python versions do not produce identical results and all manuscript analyses used the R version

## Inputs

- Lung and Pancreas benchmark scRNA-seq datasets and cell type labels (RDS/TSV from Zenodo record 8298157)
- Merged multi-condition Seurat or SingleCellExperiment object with a batch_label metadata column

## Outputs

- JOINTLY embedding (H matrix stored as a 'JOINTLY' DimReduc) plus UMAP embedding for the merged mixture
- Cluster labels (seurat_clusters) and module scores (e.g., interferon-signalling UCell scores) per cell
- Comparison embeddings/clusters from fastMNN, Harmony, scVI, LIGER and Scanorama with iLISI/cLISI, over-correction, ARI and NMI scores
- Across-tissue DE gene set (3529 genes between pancreas- and lung-derived endothelial cells) via wilcoxauc

## How to apply

Download the Lung and Pancreas benchmark datasets and cell type labels from Zenodo record 8298157 and merge them into a single Seurat object annotated with a batch_label variable. Run JOINTLY on the merged object using the wrapper jointly(merged, batch.var = "batch_label") in regular PCA mode — the authors explicitly advise against consensus PCA (cpca) for multi-condition, multi-sample datasets because samples may not share the same axes of variation. Follow with FindNeighbors and FindClusters on the 'JOINTLY' reduction (dims 1:15, res = 0.05) and RunUMAP on the same dims. Run comparison methods fastMNN, Harmony, scVI, LIGER and Scanorama on the same merged mixture, then compute within-tissue integration scores (iLISI/cLISI), between-tissue over-correction scores, and ARI/NMI against cell type labels via aricode. To test retention of across-tissue signal, identify DE genes between pancreas- and lung-derived endothelial cells (3529 genes) with wilcoxauc (Wilcoxon rank-sum) and score the interferon-signalling module with AddModuleScore_UCell.

## Related tools

- **JOINTLY** (Primary integration method; run in regular PCA mode via jointly(merged, batch.var = 'batch_label') on the merged mixture) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Object handling, FindNeighbors/FindClusters (dims 1:15, res 0.05), RunUMAP and visualization of the merged multi-condition object) — https://github.com/satijalab/seurat
- **UCell** (AddModuleScore_UCell to score the interferon-signalling module on endothelial cells per method embedding) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc Wilcoxon rank-sum test to find the 3529 DE genes between pancreas- and lung-derived endothelial cells) — https://github.com/immunogenomics/presto
- **fastMNN** (Comparison batch-integration method run on the same merged mixture)
- **Harmony** (Comparison batch-integration method run on the same merged mixture)
- **scVI** (Comparison batch-integration method run on the same merged mixture)
- **LIGER** (Comparison batch-integration method run on the same merged mixture)
- **Scanorama** (Comparison batch-integration method run on the same merged mixture)
- **aricode** (ARI/NMI calculation between cell type labels and cluster labels for evaluation)

## Examples

```
merged <- jointly(merged, batch.var = "batch_label"); merged$object <- FindNeighbors(merged$object, reduction = "JOINTLY", dims = 1:15); merged$object <- FindClusters(merged$object, res = 0.05)
```

## Evaluation signals

- Within-tissue batch effects are removed: iLISI/cLISI improves for batches within each tissue relative to raw/comparison methods
- Across-tissue signal is retained: the interferon-signalling module scores and the 3529 pancreas-vs-lung endothelial DE genes remain separable between tissues in the JOINTLY embedding
- ARI and NMI between celltype labels and seurat_clusters are computed with aricode and comparable to benchmark values (e.g., liver test set: ARI 0.7416, NMI 0.6325)
- Consensus PCA was NOT used: cpca is replaced by regular PCA mode, as advised for multi-condition, multi-sample datasets
- UMAP plots colored by cluster, celltype and batch_label show batch mixing within tissues but separation between pancreas and lung (Fig. 3A-F)

## Limitations

- Consensus PCA in JOINTLY is not advised for complex multi-condition, multi-sample datasets; regular PCA should be used instead
- No method is always best across all tasks; integration performance must be evaluated per task, and metrics alone do not reveal which genes drive integration
- Results are stochastic: setting a seed and re-running produces different results, and it can be worth running JOINTLY up to 5 times and choosing the best solution
- Software version numbers are not stated in the methods, which may complicate exact reproduction

## Evidence

- [discussion] For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA"
- [intro] Over-correction evaluation scope: "We evaluate the robustness against over-correction by integrating multi-donor datasets from different tissues and find that JOINTLY removes within-tissue batch effects but retains across-tissue"
- [methods] Wrapper invocation and batch variable: "liver <- jointly(liver, batch.var = "batch_label")"
- [methods] Clustering parameters on the JOINTLY reduction: "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)
liver$object <- FindClusters(liver$object, res = 0.05)"
- [methods] Zenodo data source for benchmark datasets and labels: "Datasets, labels, embeddings and results used for testing JOINTLY can be downloaded here: https://zenodo.org/record/8298157"
- [readme] Stochasticity caveat and rerunning recommendation: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
