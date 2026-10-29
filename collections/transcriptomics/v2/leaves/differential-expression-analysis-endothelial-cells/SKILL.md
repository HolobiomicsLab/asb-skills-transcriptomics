---
name: differential-expression-analysis-endothelial-cells
description: Use when benchmarking single-cell batch-integration methods on multi-condition
  mixtures and you need a quantitative readout of whether across-condition biological
  differences (e.g. tissue-specific endothelial DE genes) survive integration.
when_to_use_negative:
- When the goal is to detect within-tissue batch effects rather than across-tissue
  biological signal — use iLISI/cLISI integration scores instead
- When the input is not endothelial cells from distinct biological conditions (e.g.
  a single-tissue, single-condition dataset has no across-tissue DE signal to protect)
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: presto
  role: Provides wilcoxauc for the Wilcoxon rank-sum differential expression test
    between pancreas- and lung-derived endothelial cells
  repo: https://github.com/immunogenomics/presto
- name: UCell
  role: Scores the interferon-signalling module on endothelial cells to visualize
    retained across-tissue signal per integration method
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Holds the merged multi-condition mixture object and provides the RNA assay
    and DimPlot/FeaturePlot visualization for per-method embeddings
  repo: https://github.com/satijalab/seurat
- name: JOINTLY
  role: Integration method under evaluation; its reduction is compared against fastMNN,
    Harmony, scVI, LIGER and Scanorama for retention of endothelial DE signal
  repo: https://github.com/madsen-lab/rJOINTLY
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:7db29baa6ac4f1c4e46a8cd90825c69ebf0ee4317ea57fc601f582bdb00d98c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/presto
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
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# differential-expression-analysis-endothelial-cells

## Summary

Identifies differentially expressed genes between pancreas- and lung-derived endothelial cells (3529 genes) using a Wilcoxon rank-sum test (wilcoxauc) to assess how well each batch-integration method retains across-tissue biological signal and resists over-correction.

## When to use

Use when benchmarking single-cell batch-integration methods on multi-condition mixtures and you need a quantitative readout of whether across-condition biological differences (e.g. tissue-specific endothelial DE genes) survive integration.

## When NOT to use

- When the goal is to detect within-tissue batch effects rather than across-tissue biological signal — use iLISI/cLISI integration scores instead
- When the input is not endothelial cells from distinct biological conditions (e.g. a single-tissue, single-condition dataset has no across-tissue DE signal to protect)

## Inputs

- Merged pancreas-lung multi-condition Seurat object with batch_label variable (from Zenodo record 8298157)
- Cell type labels identifying endothelial cells per tissue
- Integration embeddings/clusters from JOINTLY, fastMNN, Harmony, scVI, LIGER, Scanorama

## Outputs

- Table of 3529 differentially expressed genes between pancreas- and lung-derived endothelial cells (wilcoxauc Wilcoxon rank-sum results)
- Per-method interferon-signalling module scores on endothelial cells (AddModuleScore_UCell)
- Per-method over-correction assessment showing whether across-tissue endothelial signal is retained (for Fig. 3A-F visualization)

## How to apply

Starting from endothelial cells in the merged pancreas-lung Seurat mixture (batch_label variable, data from Zenodo record 8298157), run differential expression between pancreas-derived and lung-derived endothelial cells on the raw, unintegrated expression values using the Wilcoxon rank-sum test implemented in presto's wilcoxauc (seurat_assay = 'RNA'), yielding 3529 DE genes that define the across-tissue biological signal. Then, for each integration method's output (JOINTLY, fastMNN, Harmony, scVI, LIGER, Scanorama), score how well this signal is retained, e.g. by projecting the interferon-signalling module onto endothelial cells with AddModuleScore_UCell and checking that pancreas and lung endothelial cells remain separable in each embedding. A method that collapses the two tissue-specific endothelial populations exhibits over-correction, while JOINTLY should retain the DE genes and module score separation while still removing within-tissue batch effects (improved iLISI/cLISI). Accept the workflow when the 3529 DE genes and the interferon-signalling module scores remain separable across methods' embeddings, reproducing Fig. 3A-F.

## Related tools

- **presto** (Provides wilcoxauc for the Wilcoxon rank-sum differential expression test between pancreas- and lung-derived endothelial cells) — https://github.com/immunogenomics/presto
- **UCell** (Scores the interferon-signalling module on endothelial cells to visualize retained across-tissue signal per integration method) — https://github.com/carmonalab/UCell
- **Seurat** (Holds the merged multi-condition mixture object and provides the RNA assay and DimPlot/FeaturePlot visualization for per-method embeddings) — https://github.com/satijalab/seurat
- **JOINTLY** (Integration method under evaluation; its reduction is compared against fastMNN, Harmony, scVI, LIGER and Scanorama for retention of endothelial DE signal) — https://github.com/madsen-lab/rJOINTLY

## Examples

```
res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA"); liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)
```

## Evaluation signals

- wilcoxauc on raw RNA values recovers 3529 DE genes between pancreas- and lung-derived endothelial cells
- AddModuleScore_UCell interferon-signalling scores show clear separation between pancreas and lung endothelial cells on the JOINTLY embedding
- Per-method UMAPs colored by cluster/cell type and batch show preserved across-tissue endothelial separation while within-tissue iLISI/cLISI improves
- Results reproduce Fig. 3A-F layout: per-method embeddings, cluster/cell-type and batch coloring, and module score feature plots

## Limitations

- Setting a seed and re-running JOINTLY produces a different result; the README notes it can be worth running JOINTLY up to 5 times and choosing the best solution
- Consensus (c)PCA in JOINTLY is not advised for complex multi-condition, multi-sample datasets; regular PCA should be used instead
- The DE analysis defines signal only for the pancreas-lung endothelial comparison; the 3529-gene result is task-specific and must be recomputed for other tissues/conditions

## Evidence

- [methods] DE on endothelial cells: "res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA")"
- [methods] module scoring: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [abstract] over-correction robustness: "JOINTLY is robust against over-correction while retaining subtle cell state differences between biological conditions"
- [intro] DE basis for over-correction test: "We evaluate the robustness against over-correction by integrating multi-donor datasets from different tissues and find that JOINTLY removes within-tissue batch effects but retains across-tissue"
- [discussion] multi-condition PCA advice: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
- [readme] JOINTLY stochasticity: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
